---
title: Release of angular-ocpu v0.1.0
layout: post
tags: r angular opencpu development
---

I finally found some time to spent on something I wanted to do for a while now:
starting an AngularJS based library for [OpenCPU](https://www.opencpu.org/).
This is to supplement rather than to compete with Jeroen's excellent [jQuery based library](https://github.com/jeroenooms/opencpu.js).
At [work](http://www.list.lu/en/erin/) I'm developing an interactive application for my biology colleagues.
For those who have following me a bit, it should not come as a surprise that this application is based on OpenCPU and AngularJS.
To cut down on dependencies and coding styles/semantics, I figured it would be nice to have an $ocpu AngularJS service.

<br>
This evening I sat down and had a look at some work I started roughly three months ago.
My initial take was basically a copy paste of [opencpu.js](https://github.com/jeroenooms/opencpu.js/blob/master/opencpu-0.5.js).
However, I figured that the jQuery semantics are slightly different than the AngularJS semantics.
As such I started a rewrite which is more AngularJS like (hopefully).

<br>
Today I'm pleased to anounce the release of [angular-ocpu v0.1.0](https://github.com/bbroeksema/angular-ocpu/releases/tag/v0.1.0), which provides **very** basic functionality.
To be more precise $ocpu.call and $ocpu.rpc [seem](http://bbroeksema.github.io/angular-ocpu/hello.html) to work, but that's all I dare to say for now.
Nevertheless, it's a good start for getting opencpu in your angular application, the angular way.

<br>
Surely, I have not integrated this in my actual application yet.
That will happen as soon as I have implemented all the features I need.
In the meanwhile, if you feel like contributing, have a look at the [source](https://github.com/bbroeksema/angular-ocpu) and at [contributing.md](https://github.com/bbroeksema/angular-ocpu/blob/master/CONTRIBUTING.md).
Once done reading and programming, send me the pull requests!
