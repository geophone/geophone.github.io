---
layout: post
title: "Vast.ai hosting"
date: 2026-04-15
---
vast.ai hosting

Looked into vast.ai hosting recently and learned a bunch!

For starters your best bet is to start with a Ubuntu 24.04 LTS base image.
Using more recent versions of Ubuntu is problematic due to a number of factors. One principle one is the pipes module which is deprecated in modern ubuntu system python. Using a non-standard python (non-system) results in other complications such as reliance on deprecated apt-key.

I learned this through viewing install logs and examining the script.
There are some small bugs in the UI such as speedtest when manually run through the vast.ai not necessarily updating the uplink and downlink which can often be stale through my experimentation.

Also there is mention of rocM support (AMD GPUs) however this is actually out of date and currently only Nvidia is supported.

Most of the install guides can be skipped once you have the 24.04 base image you simply run the python3 wget and install line and it handles all the nonsense. I found that removing a machine and re-adding remedied the local network ip address change which is mentioned in the doc.
In short summary
```bash
Use ubuntu 24.04 LTS base image

Install directly via install line without messing with the rest of the configuration(although ssd partitioning may be a serious improvement) and if you need an update fresh install is easiest
```
Nvidia only is supported

GL hosting!
