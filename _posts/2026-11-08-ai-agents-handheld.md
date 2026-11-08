---
layout: post
title: Controlling your AI agents from a game handheld
category: llms
---

## Introduction

Everybody talks about how they control their AI agents from their mobile phone. This is great.
But did you know that I use my gaming handheld for my personal agents during business trips?

At the heart of the setup is an Android gaming device that is normally aimed at running gaming emulators.
It is the [Retroid Pocket 5](https://www.goretroid.com/products/retroid-pocket-5-handheld), which combines a small 1920x1080 OLED screen with a built-in gamepad.

![Retroid Pocket 5 handheld](../../assets/ai-agents-handheld/retroid-pocket5.jpg)

It is essentially an Android phone, which means that you can install standard Android applications.

The setup is as follows:

* Retroid Pocket 5 (about $250)
* [Arzopa A3C](https://www.arzopa.com/products/arzopa-a3c-pro-13-3-oled-portable-monitor-400-nit-bright-ultra-thin) OLED portable screen, 13-inch ($130)
* Power bank or charger
* Portable foldable keyboard ($20)

![Herdr app for controlling AI agents](../../assets/ai-agents-handheld/herdr.jpeg)

The idea here is that I just use stuff that I would bring anyway on a business trip. The portable
screen is the second screen for my business laptop. The gaming handheld is for long flights. I only bought
the keyboard after I realised that I could reuse the same things for a different purpose.

For software, I use:

* [Termius](https://termius.com/) (you can also use [TermRover](https://termrover.sh/) or [moshi.app](https://getmoshi.app/))
* [Tailscale](https://tailscale.com/) for accessing my mini PC that actually runs the agents
* [pi](https://pi.dev/)/[opencode](https://opencode.ai/)
* [herdr](https://herdr.dev/) for multiplexing

The mini PC also has several required utilities for text editing, running Docker containers, processing images, etc.


When I want to edit files by hand or view the diffs in detail, I launch a code server (VS Code on the web) and just use it as a normal IDE:

![VS Code server running on the handheld](../../assets/ai-agents-handheld/codeserver.jpeg)

Alternatively, I just open vi directly from inside herdr.

It also goes without saying that the OLED screen is amazing when actually gaming (which is the original purpose of the Retroid).

![Gaming on the Retroid Pocket](../../assets/ai-agents-handheld/retroid-gaming.jpeg)


So far the setup works great. I have only used it in places such as hotels and airports. Most of the Android terminal apps also support [Mosh](https://mosh.org/) so I assume that places such as trains could also work fine.

## F.A.Q.

Q. Dude, just get a laptop
A. I already have a business laptop for my trips. Getting a second laptop just for running my personal agents would increase the weight a lot. Not to mention the cost.

Q. Just get a tablet then
A. An Android tablet with a 14-inch OLED screen costs almost $1400. The Retroid is something I already had. Also, the latest Samsung flagship tablets do not use Snapdragon, so I would still have to carry the Retroid for emulators.

Q. What about a mobile phone (like everybody else)?
A. Only flagship (i.e. very expensive) mobile phones support video output over USB-C. The price of the Retroid is now less than $200. And it always doubles as a game console. Also, when I am on a trip I prefer to only use my mobile phone for essential tasks (maps, calls, tickets, etc.).

Q. What mini PC do you use?
A. Nothing special. Just a GMKtec G10. It only runs the agents. I have another gaming PC that I am going to set up for running LLMs as well, but this is for another article.
