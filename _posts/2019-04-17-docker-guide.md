---
layout: post
title: Docker intermediate guide
category: containers
---

Docker (and containers in general) are currently taking the world by storm. Containers are one of those technologies that touch several aspects of software development and act as a bridge between developers and operators/sys-admins.

If you have already read any basic Docker tutorial and wanted to learn more, then this article is for you.

We will present good techniques for creating dockerfiles, how to work with Docker registries and introduce Docker compose.


## Creating Docker images

First things first. Before you manage Docker containers you need to learn how to create Docker images. Any beginner tutorial can explain how you can create a Dockerfile for Node and Go applications. 

Creating a Dockerfile is not rocket science but you always need to pay attention to two factors:

* The size of your Docker images
* The speed of re-building an image when you make a change in your source code or dependencies.

### Docker layers and how they work

Docker images are created in layers. You can visualize them as a cake with different flavors. Each layer sits on top of the previous one.

In a Dockerfile, every time that you add a new directive (i.e. a `RUN`,`COPY`, `ADD` command) you create a new layer.

This means that it is very easy to create huge Docker images if you are not careful. See for example this Dockerfile


```dockerfile
# Start from a small image
FROM alpine:3.6

# set a workdir inside docker
WORKDIR /incoming

# Download something
RUN wget https://storage.googleapis.com/kubernetes-release/release/v1.14.0/bin/darwin/amd64/kubectl

# delete it
RUN rm kubectl

```

This dockerfile starts from the [alpine image](https://hub.docker.com/_/alpine?tab=tags) which is a minimal docker image around 4-5 MB of space. It then downloads a big file and then immediately deletes it.

So how big is the resulting image? If you build this file and check its size you might be amazed to see that it is close to 50 MB

```shell
$ docker build . -t my-minimal-docker-image
[...]
Successfully tagged my-minimal-docker-image:latest

$ docker images 
my-minimal-docker-image latest 7b6b4 2 minutes ago  54.2MB
```

How is that possible? We deleted the file and it is not in the container:

```shell
$ docker run -it my-minimal-docker-image sh
/incoming  ls -ls
total 0
```

So where are the 50 MB coming from? They are coming from the previous layer. Remember the cake analogy. This docker image has a layer with the 50 MB file and then a **second** layer on top which deletes the `kubectl` file. The second layer does not change the contents of the first.

This means that you need to create and clean-up in the same layer.

Here is the corrected Dockerfile.

```dockerfile
# Start from a small image
FROM alpine:3.6

# set a workdir inside docker
WORKDIR /incoming

# Download something
RUN wget https://storage.googleapis.com/kubernetes-release/release/v1.14.0/bin/darwin/amd64/kubectl && \
  rm kubectl
```
If you build this Dockerfile you will see that it is actually 5 MB.

The example might seem contrived but it actually makes sense if you see how updates are installed on real Dockerfiles.

Here is an example for PHP dockerfile:

```dockerfile
FROM ubuntu:18.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update \
 && apt-get install -y --no-install-recommends apache2 ca-certificates php7.2  php7.2-sqlite3 php7.2-tokenizer \
 && ln -s /usr/bin/php7.2 /usr/bin/php7 \
 && rm -rf /var/lib/apt/lists/*

```

You can see that all dependencies are fetched in a single RUN command. Cleanup of apt resources is also happening in the same line.

The second important topic is layer caching. Docker will cache all layers that have not changed. If you rebuild an image, docker will reuse from cache any layers that are assumed to be unchanged.

The important point here is that if you change any layer Docker will invalidate it along with **ALL** layers that follow it.

Let's see an example:

```dockerfile
FROM golang:1.7.1
COPY src /go/src

RUN go build -o bin/sample src/sample/my-example-app.go

RUN wget https://storage.googleapis.com/kubernetes-release/release/v1.14.0/bin/darwin/amd64/kubectl
```

Here I am compiling my Go source code and also downloading the `kubectl` command which is 50 MB.

If you build the docker image Docker will create caches for both layers that are formed by the `RUN` commands.

If you then change anything in your source code and try to build the image again, you will see that Docker will re-download `kubectl` because it assumes that both layers need rebuilding (remember the cake analogy. If you decide you want a different base for your cake you need to start the cake from scratch).

The correct approach is to start your Dockerfile with the layers that change rarely and add layers that change often (such as the source code) at the bottom of the dockerfile.

Here is the corrected Dockerfile.

```dockerfile
FROM golang:1.7.1

RUN wget https://storage.googleapis.com/kubernetes-release/release/v1.14.0/bin/darwin/amd64/kubectl

COPY src /go/src

RUN go build -o bin/sample src/sample/my-example-app.go
```

No matter how many times you change the source code, Docker will bring `kubectl` from cache and never re-download it after the first time.

```shell
$ docker build . -t my-go-util
Sending build context to Docker daemon  96.26kB
Step 1/4 : FROM golang:1.7.1
 ---> 47734a1408b7
Step 2/4 : RUN wget https://storage.googleapis.com/kubernetes-release/release/v1.14.0/bin/darwin/amd64/kubectl
 ---> Using cache
[...]
Successfully built 8ace884fd1f4
```


### Multi stage builds

If you are still not sure about how Docker layers work, you can take matters in your own hands by using [Multi-stage builds](https://docs.docker.com/develop/develop-images/multistage-build/)

Multi-stage builds break the cake analogy. Instead of having each docker layer build on top of the previous one you can decide exactly what gets copied from each layer to the next.

In fact the order of layers does not really matter if you use multi-stage build. Only what you copy matters. Multi-stage builds help you by allowing you to define exactly what goes in your final Docker image.

Here is a Dockerfile for a Go web application:

```dockerfile
FROM golang:1.12
COPY src /go/src
RUN go build -o bin/sample src/sample/trivial-web-server.go
EXPOSE 8080
CMD ["/go/bin/sample"]
```

At first glance this Dockerfile looks correct. However if you look at the size of the image you will see that it is about 700MB. 

The reason behind this size is that the [golang](https://hub.docker.com/_/golang) image is a full-blown debian image and it includes more than you actually need.

You could try to find a slimmer go image or create your own in order to cut the size down. This approach would work but it is very time-consuming.

Multi-stage builds allow you to think outside of the box. You can use specific docker layers for compiling/packaging your application that are not actually shipped to the final image.

Especially for GO applications the results are impressive, because GO executables don't actually need the full GO development environment to run. 

Here is the multi-stage build of the previous example:

```dockerfile
FROM golang:1.12 AS build-env
COPY src /go/src
RUN CGO_ENABLED=0 GOOS=linux go build -o bin/sample src/sample/trivial-web-server.go

FROM scratch
COPY --from=build-env /go/bin/sample /app/sample

EXPOSE 8080
CMD ["/app/sample"]
```
Here even though we start from the same go image, we use it only to compile a static executable.

Then we discard that layer and copy from it only the executable. The layer that is actually shipped comes from [scratch](https://hub.docker.com/_/scratch) which is a very minimal docker image. 

The resulting image is about 8 MB! Most of this comes from the executable.

This was just a very simple example. You can use multi-stage builds to run unit tests, perform static linting, run quality scans without affecting the final size of the image.

### Docker ignore

The last point to consider regarding Docker build performance is your build context. The build context is what gets sent to the Docker daemon in order to create the image.

By default the build context is **ALL** files in your current directory when you run the build commands. This means that if you don't pay any attention blindly running `docker build` in your project folder you are sending to the daemon:

* The contents of `.git`
* The contents of `node_modules` for node projects
* The content of `target` for Java projects
* The content of `/docs`, `/sql-scripts`, `/db-dumps` and every other folder currently in your project.

This can result in very slow build times. It can also present issues with the Docker build process as it can mix libraries that were created out of the container (i.e. the existing `node_modules`) with those that were created inside the container (i.e. if you run `npm install` in your Dockerfile).

You need to make sure that you have a `.dockerignore` file which at the very minimum contains `.git`.

Then depending on your programming language and build system you also need to define all folders that are unrelated to the build process.

Here is an example for Node.

```
.git
node_modules
npm-debug.log
```

You can define much more complex rules for which files should be excluded.


### Health checks

One of the most underused features of Dockerfiles are [healthchecks](https://docs.docker.com/engine/reference/builder/#healthcheck).

The `HEALTHCHECK` command allows you to check if your docker container is really running as you would expect it.

Here is a simple Health check:

```dockerfile
FROM golang:1.12
COPY src /go/src
RUN go build -o bin/sample src/sample/trivial-web-server.go
EXPOSE 8080
CMD ["/go/bin/sample"]

HEALTHCHECK --interval=10s --timeout=3s CMD wget --quiet --tries=1 --spider http://localhost:8080/ || exit 1
```

Here I am using `wget` to test the url of the application. You can also use `curl` or any other command/script to verify the correctness of your container. 

The healthcheck command does not need to deal with ports. It could also check for a file, an environment variable or anything else that you can think of that can be scripted and has a binary result (0 success, 1 failure).

Docker health checks are used as sanity checks. You can see the status of your container when

* you run `docker ps`
* you run `docker inspect`
* you use any Docker UI or tool that supports healthchecks

Here is an example where I launch the same container with and without a healthcheck.
```shell
$ docker ps
CONTAINER ID IMAGE      COMMAND        CREATED      STATUS 
1773ce8      my-app     "/sample"    4 seconds ago  Up 3 seconds
b0ea7c       my-app-hc  "/sample"    2 minutes ago  Up 2 minutes (healthy)   
```

The healthcheck directive also supports settings for:
 * start period (how much time to wait before first check)
 * interval (how often to check afterwards)
 * timeout (how long can each check take)
 * retries (number of times to retry)

Spend some time to fine-tune these settings. If your container takes a long time to start for example you should provide a big enough value for the start period.



## Docker registries

Now that we have talked about the creation of Docker images, the next step is their storage. Most beginner articles describe how you can push a public image to Dockerhub.

Dockerhub is just one example of a Docker registry. In an organization/company you should use multiple Docker registries (both public and private) in order to create a promotion process.

Before talking about promotion, let's look at base images.

### Base images

Most example dockerfiles that you can find on-line (including those already shown in this article) start from some well known image (with the `FROM` directive) such as [python](https://hub.docker.com/_/python/), [node](https://hub.docker.com/_/node/), [alpine](https://hub.docker.com/_/alpine/) etc.

These are top level images that are curated by the Docker team and are expected to be safe/stable. You can always see their Dockerfile and adopt it to your needs if you want to create a custom image.

Every other image contained in Dockerhub (in the form of username/image) is unofficial and has no guarantees regarding safety and stability. It is ok to use Dockerhub images for experiments or side projects, but not a very good practice for security sensitive companies

In a company environment you are not typically creating images starting from Dockerhub. You should instead create your own base images that describe exactly what your environment needs.

The creation of base docker image does not always happen by developers. Your company might have a dedicated team for this purpose.

Your company Docker images will typically reside in a private Docker registry which is completely internal. In most cases, a security scanning service will also be in place to scan the images uploaded for vulnerabilities.

It is hard to describe what should go in a base image because it is very specific to company processes. Usually it should have

* the main programming language utilities used within the company
* test frameworks, quality scanning, linting tools etc.
* special security updates, custom certificates or other secure libraries

It is also possible to have more than one base images and use them in different build phases (as seen in multi-stage builds).

The point is that if you are joining a company you should learn about the base images that are used already and use them as the starting point for your dockerfile.

Most of your Dockerfiles should look like this:

```dockerfile
FROM my-company-approved-image:stable
COPY src /src
[...rest of Dockerfile]
```

or if you use multi-stage builds:

```dockerfile
FROM my-company-approved-builder:stable as builder
COPY src /src
[...build something]

FROM my-company-approved-tester:stable 
COPY --from=builder ....
[...test something]

FROM my-company-approved-base:stable 
COPY --from=builder ....
[...package something]
```

And if you find that there are no base images, coordinate with your team to create some!

Base images will make your Docker builds much faster as well, because they will typically be using the cache effectively (see the first section of this article).



### Tags and releases

Tags in Docker images are sometimes confusing because they don't behave like typical version releases as found in packaging systems like npm, maven, pip, gems etc.

First of all let's clear some misconceptions regarding the `latest` tag. Here are the 3 rules for its usage:

1. Don't use the `latest` tag for any production project/workflow
1. Don't use the `latest` tag for any production project/workflow
1. Don't use the `latest` tag for any production project/workflow

The problem with the `latest` tag is that people assume that it is special while in reality it isn't. A docker image is tagged as latest explicitly or it gets this tag if it is not tagged at all. It is **NOT** the most recent tag of docker version.

Here is a simple example.
```dockerfile
FROM alpine:3.6
CMD echo version 1
```

I can create an image and run it

```shell
$ docker build . -t my-app
[...]
Successfully tagged my-app:latest

$ docker run my-app
version 1
```

So now I tell my teammates (or my CI system) to use my image and use the latest tag.

Now I want to update my image. So I change my dockerfile to:

```dockerfile
FROM alpine:3.6
CMD echo version 2
```

I now tag my image as version 2. What will happen if I run the "latest" version?

```shell
$ docker build . -t my-app:v2
[...]
Successfully tagged my-app:v2

$ docker run my-app
version 1
```

The old version is still shown as `latest` . The `latest` tag was not updated automatically. `Latest` means nothing to the Docker daemon. There is no extra processing or smartness behind `latest` tags.

In order to avoid pitfalls with `latest` it is much safer to tag your images explicitly:

1. Don't assume the presence of a `latest` tag
2. Don't build empty/latest tags
3. Don't use empty/latest tags in your dockerfiles or CI builds

Now you know about the `latest` tag. So how should you tag your Docker images.

There is no standard rule here. Docker allows you to use any kind of tags that matches your workflow. A common pattern would be application versions

* `my-app-image:1.2.5`
* `my-app-image:1.6.4`

For less critical projects (or helper images) you could use the branch name:

* `my-fancy-utility:master`
* `my-fancy-utility:develop`
* `my-fancy-utility:feature-X`

It is also possible to use stages of the software lifecycle

* `my-app:production`
* `my-app:security-checks-ok`
* `my-app:load-testing`

Remember that with Docker you can use multiple tags for the same Docker image. So you could combine multiple tags and have an image that is marked with:

* `my-app:production`
* `my-app:v2.4`
* `my-app:security-passed`
* `my-app:unit-tested`
* `my-app:git-hash-323a3aa1f8ac`

```shell
$ docker -t my-app:v1.2 -t my-app:production -tmy-app:git-commit-323a3aa1f8ac
[...]
Successfully built a74715953638
Successfully tagged my-app:v1.2
Successfully tagged my-app:production
Successfully tagged my-app:git-commit-323a3aa1f8ac
```

The point here is to agree with your team on a set of tag rules and naming conventions. All members of the team should follow the **same** rules.


### Pushing and promoting

Dockerhub is the default public repositories for Docker images. It is good for learning how Docker pushing works. But in production scenarios it should not be your only Docker registry.

In a team/company you should have at least 3 Docker registries in place

1. The development registry
2. The production registry
3. The public registry

The development registry is where **ALL** Docker images end up as created from developers. It should be considered ephemeral and non-production (periodically it should be cleaned from all images)

Images contained in the development registry should be shared within teams for experimentation and testing. The development registry should also be accessible by all developers (anybody should be able to push to it even from a workstation)

The production registry on the other hand has very strict security requirements. First of all nobody can push there by hand. Only a CI server should have access by promoting images from the development registry. 

Development images that pass the set of company requirements (code scanning, security analysis, integration/load testing) should be promoted to the production registry automatically.

The production registry is the only registry accessible to production servers (i.e a developer cannot directly deploy to production an image that is not found on the production registry).

The production registry should also have a well defined audit policy. The images contained in the registry will serve as a historical record on what was deployed in production.

Finally, a public registry can be used for software that the company wants to release to the internet. It should be separate from the other two registries.

There are many ways to install or host a docker registry so you can mix and match solutions for this 3-layer-registry approach.


## Docker compose

Docker compose is a way to launch and manage multiple containers with a single configuration file and a single set of commands.

You can use it to manage:

* A single application that has lots of micro-services in the form of different docker images
* An application that also depends on other services such as databases and message queues
* Any set of supporting workflows (i.e. integration tests against a docker image)

The [official tutorial](https://docs.docker.com/compose/gettingstarted/) is very good and explains how you can manage a python application with redis as a single entity.

A more interesting example is also the [voting-app](https://github.com/dockersamples/example-voting-app/blob/master/docker-compose.yml).

The first thing to know regarding Docker compose is the syntax. The "canonical" version of compose is version 2. [Version 3](https://docs.docker.com/compose/compose-file/compose-versioning/) might be newer but can sometimes be very confusing because it is blurring the capabilities of Docker compose with [Docker swarm](https://docs.docker.com/engine/swarm/) which is another product by Docker Inc.

If you are learning your way around Docker-compose stick with [version 2](https://docs.docker.com/compose/compose-file/compose-file-v2/). 

The second important point is that Docker compose can both build images on the fly as well as use pre-existing images. In most cases you use the on-the-fly build directive for the application that you are developing and the pre-existing images for other services such as your database or message queue.

### Where to Go from Here

I hope you found this intermediate guide useful. There are so many more topics to discover in this area.

* Service dependencies
* Volume mounting
* Management with Kubernetes
* Security scanning and other CI practices.

We will cover all these topics in future articles.




