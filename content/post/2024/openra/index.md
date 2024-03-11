---
title: "OpenRA Rebuilt for the Modern Era with Multi Arch Docker Images"
date: 2024-03-11T22:10:00+00:00
draft: false
author: Dennis Kruyt
categories:
- automation
- games
cover:
  image: /images/2024/openra-banner.png
slug: openra-docker
tags:
- automation
- games
---

## Personal Touch

I still remember the day I got Command and Conquer Red Alert as a birthday gift. I've spent too many hours to count playing this game over the years. These days, I still enjoy it from time to time, teaming up with my son or work buddies for a match. Today, thanks to OpenRA, an open-source version of Red Alert is available, not to mention Dune and Tiberian Sun. We usually play these games online, which is why I run them on my ARM-based server. But there's always been one issue: there are no decent Docker images available for OpenRA. Some are outdated; others aren't compatible with ARM architecture. So, I decided it was time to sort this out. I created a Docker build file and a Docker compose file that syncs with GitHub Actions. This setup helps me keep an eye on new OpenRA releases and auto-builds them. Whenever a new update is released, I can just pull a new Docker image onto my server. It's as simple as that!

{{< bookmark url="https://github.com/dkruyt/openra" thumbnail="https://opengraph.githubassets.com/22da4133c352819969e8aa7102acf052870d592371d068817a1fdb23dee5bb67/dkruyt/openra" caption="" >}}

## Technical Details 

I began by setting up a Dockerfile. This file is like a recipe that tells Docker how to build my image, including all the instructions and requirements. Once I had my Dockerfile ready, I set up GitHub Actions to run a check every day for new OpenRA releases. If a new release is detected, GitHub Actions automatically builds and pushes a new Docker image. It's like having an assistant who does all the work. And it even lets me trigger the workflow manually if I want to.

Starting the OpenRA server with the Docker image is pretty simple. You can use either Docker or Docker Compose. And if you want to run a different version of the game, just tweak a small part of the code.

What I like about Docker Compose is that it allows you to change game settings with ease. You can take care of things like modifying the server name, mod, ports, and so on.

Here's a diagram that shows the steps in of the GitHub Actions workflow that wil make sure always in 24 hourse of a new release there is a new docker contianer:

![Workflow Diagram](/images/2024/openra-flow.svg)

If there's no new OpenRA release, the workflow stops there. But if a new release is detected, it logs into the GitHub Container Registry, sets up Docker Buildx, and then builds and pushes the Docker image. Once all that is done, it wraps things up by updating and committing the new version. And there you have it, a fresh and updated OpenRA game ready to go!

Thanks to these workflows, playing OpenRA is always just a new Docker image pull away. It doesn't get simpler than that!

[Github Container Registry](https://github.com/dkruyt/openra/pkgs/container/openra)
