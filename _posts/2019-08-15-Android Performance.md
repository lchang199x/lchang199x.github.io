---
layout:     post
title:      "Android Performance: Introduction"
subtitle:   Android系统性能：引入
date:       2019-08-15
author:     Cliu
header-img: img/post-daily-bg.jpg
catalog: true
tags:
    - Android
    - Performance
---

> 本文为Google官方系列Android性能优化课程整理，[课程地址](https://cn.udacity.com/course/android-performance--ud825)

## Lesson 1: Introduction

### Why: Developers' Last Thing vs Users First Concern
For the majority of Android developers out there, the concept of performance is the last thing on their minds.

Most app development is a mad sprint towards getting features in, making the UI look perfect, and figuring out a viable monetization strategy, or a lack thereof if you're in Silicon Valley.

But application performance is a lot like the plumbing in your house. When it's working great, no one really notices or thinks about it. But when something goes wrong, suddenly, it's everyone's problem.

You see, users notice bad performance before any other features in your app. Before your social widgets, awesome image filters, and how one of your supported languages is Klingon. And guess what users unhappy with performance give bad reviews at a higher percentage than any other problems in your app.

This is why we like to say that perf matters.

It's easy little side of performance as you're developing your app. But frankly perf is involved in everything you do. Users feel bad performance. They complain about bad performance. They uninstall your app, and then vengefully give you a bad review all because of performance. When you think of it this way, performance sounds more like a feature that you should focus on, rather then a burden that you have to put up with.

But in honesty, improving performance is a really tough thing to do.

### How: Know Why It's Slow & Know the Tools
Fixing these problems means knowing why it's slow, which means knowing how to use the right tools in the right ways to get insight into that data. But don't worry. Everything in this Udacity course has been specifically built around helping you know the tools, see how problems manifest in them, and understanding what they mean from a theory prospective. Basically, it's everything that you need to become a performance ninja. I'm Colt, this is Chris, let's get started.

### Process: Three Steps
The process of improving application performance can be a daunting task, but it's actually much simpler than that. Once you shift gears into focusing on performance problems, you've now entered what we call the performance improvement life-cycle. It's a very small set of tasks you must perform, to find and fix a problem.

It namely consists of three steps.

* Number one, `gather information`. When someone says that your app is slow, you have to go and figure out why. This means you need to run the profiling and feedback tools to collect information about your app. What you can measure, you can optimize, right? Which means that in the beginning of optimizing any of your applications, the process hinges on being able to measure its state and then evaluate it after you make changes.

* Step number two, is to` gain insight`. The data that you gather is not always obvious. Now, most of the time, the content is a big collection of floating point numbers, which then, if you're lucky, gets turned into some snazzy visualization by a tool. But even then, chances are that you're still at a loss for what all those colors, lines, and charts actually mean. This is where developers become performance gurus. Now you see interpreting a sheet of floating point numbers and then realizing that you're spending too much time serializing XML, is the modern equivalent of a shaman reading chicken bones to predict the future. 

And honestly these first two stages of gathering data and then gaining insight, happen in a really tight loop. You may use one tool, gain some info from it, and then realize that the problem is in another part of your pipeline. Which then you'll need a separate tool to diagnose. 

* But of course that leads us to the third step, `take action`. This is often the most difficult part of the loop this is where you take all the numbers and all the insight, and know where the problem is, and then you have to go convince the other programmers the right way to fix it. Of the three stages, this has the most human component to it. Because solving the problem isn't enough, you need to solve it in a way that meets your coding standards for your complexity or your company or takes in account libraries or a particular module or a platform that you're running on, or other crazy restrictions that your code base might have. Before your solution is accepted, you usually have to take all of these things into account.

### Tools not Rules
But before you run off into the weeds and become a performance engineer I have one small piece of warning about performance. Throughout this course we'll be teaching you what's going on under the hood of Android and teaching you how to use the tooling to gain insight into some of those actions. Some of that content is going to be prescriptive like if you see this, it means this, so do that. But in honest, that might not always be the case for every application sure something may be inefficient but if it's not impacting your performance, it's not worth stressing over. This is why we like to say, `tools not rules`, while it's important to understand the rules and the flow of things, you need to validate the true real issues by trusting what your tools say first. It may not make sense for you to solve a problem that you actually don't have. Always go back to the data and validate the true nature of a problem, decide if it's a problem, before moving on to the optimization process. You've been warned, 