---
layout: post
title: Controlling your AI agents from a game handheld
category: llms
---

## Introduction

Everybody talks about how they control their AI agents from their mobile phone. This is great.
But did you know that I use my gaming handheld for my personal agents during business trips?

At the heart of the setup is an android gaming device that is normally aimed at gaming emulators.
It is the Retroid Pocket 5 that combines a small 1920x1080 OLED screen with built-in gamepad.

![Retroid Pocket 5 handheld](../../assets/ai-agents-handheld/retroid-pocket5.jpg)

It is essentially an android phone which means that you can install standard android applications.

The setup is the following

* Retroid Pocket 5 (about $250)
* Arzopa A3C OLED portable screen 13inch ($130)
* Powerbank or charger
* Portable foldable keyboard ($20)

![Herdr app for controlling AI agents](../../assets/ai-agents-handheld/herdr.jpeg)

The idea here is that I just use stuff that I already brought anyway on a business trip. The portable
screen is the second screen for my business laptop. The gaming handheld is for long flights. I only bought
the keyboard after I realised that I could reuse the same things for a different purpose

For software I use

* Termius (you can also use TermRover or moshi.app)
* Tailscale for accessing my mini pc that actually runs the agents
* pi/opencode 
* herdr for multiplexing

The mini pc has also several required utilities for text editing, running docker containers, processing images etc.

So far the setup works great. I have only used it in places such as hotels and airports. Most of the Android terminal apps also support Mosh so I assume that places such as trains could also work fine.




## Main Text


![Gaming on the Retroid Pocket](../../assets/ai-agents-handheld/retroid-gaming.jpeg)

![VS Code server running on the handheld](../../assets/ai-agents-handheld/codeserver.jpeg)

