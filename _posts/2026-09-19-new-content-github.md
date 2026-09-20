---
layout: post
title: "New repos on geophone"
date: 2026-09-19
---

I added a new russian to english translation utils vibecoded

[Russian Tools](https://github.com/geophone/russian_tools)

Some snippets to my gists for building llama.cpp for rocm and cuda(GB10)
After fixing the GPU firmware on my framework to use version 0x86 so that it actually can use the GPU properly for ROCm stuff noticed that it's only using like 60GB when Nvidia is using 80GB of GPU for same model so looking into that not sure if it's offloading to disk or what

[Ryzen Build](https://gist.github.com/geophone/18d67883a688f93bb98c66c39438d702)

[GB10 Build](https://gist.github.com/geophone/29a24f8e55c3f5207b3ac4d98112d971)

Also added some fun audio utils using magenta realtime2 which on mrt2_small for both the spark and the framework can run it in realtime so you can audio sample which is recommended about 60 seconds slice and then run it

[Audio Stuff](https://github.com/geophone/audiostuff)

And the website as here and updated my github pins and website

[Website Github source](https://github.com/geophone/geophone.github.io)

[Pins updated](https://github.com/geophone)
