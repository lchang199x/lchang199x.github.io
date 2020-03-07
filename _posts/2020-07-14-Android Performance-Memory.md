---
layout:     post
title:      "Android Performance: Memory"
subtitle:   Android系统性能 (4)
date:       2020-07-14
author:     Cliu
header-img: img/post-daily-bg.jpg
catalog: true
tags:
    - Android
---

> Based on resources of the udacity course [Android Performance](https://cn.udacity.com/course/android-performance--ud825) developed by Google. Code samples can be downloaded from [Github](https://github.com/udacity?q=ud825&type=&language=).

### Dialogue: Intro
*&gt;&gt; [SOUND] Oh, hey Colt, what's up?  
&gt;&gt; [INAUDIBLE].  
&gt;&gt; Yeah can I get a latte?  
&gt;&gt; [INAUDIBLE]  
&gt;&gt; Oh, you're right, yeah. Dude, by the way, I have to say, Java's manage memory is so cool, I saved so much time not having to care about when my objects are released.  
&gt;&gt; [INAUDIBLE]  
&gt;&gt; Yeah, it's like the perfect setup for a programmer. Allocate objects, this doesn't freeze them. I can focus on doing cooler stuff.  
&gt;&gt; [INAUDIBLE].  
&gt;&gt; Yeah and the best part, it's for free, the system just does it.  
&gt;&gt; [INAUDIBLE].  
&gt;&gt; No, seriously, the manage memory is free, you don't have to pay a cent.  
&gt;&gt; [INAUDIBLE].  
&gt;&gt; Oh, dude, just embrace it, I can write code however I want, Garbage Collector does the rest.  
&gt;&gt; [INAUDIBLE].  
&gt;&gt; Oh.  
&gt;&gt; [INAUDIBLE].  
&gt;&gt; Yeah.  
&gt;&gt; [INAUDIBLE].  
&gt;&gt; You're right.  
&gt;&gt; [INAUDIBLE].  
&gt;&gt; Oh, I see, so you're saying there's a tax for garbage collection events.  
&gt;&gt; [INAUDIBLE].  
&gt;&gt; Yeah when they happen in a short period of time.  
&gt;&gt; [INAUDIBLE].  
&gt;&gt; Okay, yeah I'll be more careful when I write my loops.  
&gt;&gt; [INAUDIBLE].  
&gt;&gt; Yeah, I totally hated to have draining into my apps framerate.  
&gt;&gt; [INAUDIBLE].  
&gt;&gt; Oh so how do I spot these problems then?  
&gt;&gt; [INAUDIBLE].  
&gt;&gt; Run the tools, got it.  
&gt;&gt; [INAUDIBLE]  
&gt;&gt; Okay, sounds good, I'll see you. Oh, hey wait, make it a double, I'm going to need it.*

### Memory, GC, and Performance
Now that all of our code is running fast and awesome, let's talk a bit more about memory and how it affects the performance in our system.

Many programming languages that are known for being close to the hardware, or rather, high performance, like C, C++, and Fortran, usually programmers to manage memory themselves. Effectively programmers are responsible for allocating a block of memory and then sometime in the future de-allocating it when they're actually done using it. Since you define when and how much memory to allocate in free, the entire quality of managing memory depends on your skills and effectiveness. That's a lot of responsibility.

And the reality programmers aren't always the best at keeping track of all those bits and pieces of memory. I mean think about it, product development is a muddy and crazy process and often memory ends up not getting freed properly. These un-liberated blocks of memory, are called `memory leaks` and they just sit around hogging resources, that you could use better or somewhere else. To reduce this chaos, stress, and sometimes big money losses, caused by memory leaks, `managed memory languages` were created.

The run times of these languages `track` memory allocations and release memory back to the system when it's no longer being needed by the application itself, all without any intervention from the programmer. This art, or rather science, of reclaiming memory in a managed memory environment is known as `garbage collection`, this concept was created by John McCarthy in 1959 to solve problems in the lisp programming language.

The basic principles of garbage collection are as follows,

* Number one, find data objects in a program that `cannot be accessed` in the future for example, any memory that is no longer referenced by the code.

* And number two, reclaim the resources used by those objects.

Simple concept in theory, but it gets pretty complex once you've got 2 million lines of code and four gigs worth of allocations. Now think about it, garbage collection can be really gnarly, I mean, if you've got some 20,000 allocations in your program right now. `Which` ones aren't being needed anymore? Or better yet, `when` should you execute a garbage collection event to free up memory that isn't used? These are actually very difficult questions, and thankfully we've had about 50 years worth of innovation to improve them, which is why the garbage collector in Android's Runtime, is quite a bit more sophisticated than McCarthy's original proposal. It's been built to be as fast and non-intrusive as possible.

Effectively the `memory heaps` in android's runtimes are segmented into spaces, based on the type of allocation and how best the system can organize allocations for future GC events. As a new object is allocated, these characteristics are taken into account to best fit what spaces should be placed into depending on what version of the android runtime you're using. And here's the important part. Each space has a set size, as objects are allocated, we keep track of the combined sizes, and, as a space starts to grow, the system will need to execute a garbage collection event in an attempt to free up memory for future allocations.

Now it's worth putting out that GC events will behave differently depending on what Android runtime you're using. For example, in Dalvik many GC events are `stop the world events`, meaning that any managed code that is running will stop until the operation completes. Which can get very problematic, when these GCs take longer than normal or there's a ton of them happening at once, since `it's going to significantly eat into your frame time`.

And to be clear, the Android engineers have spent a lot of time making sure that these events are as fast as possible to reduce interruptions, that being said, they can still cause some application performance problems in your code.

* Firstly, understand that the more time your app is spending doing GCs in a given frame, the less time it's got for the rest of the logic needed to keep you under the 16 millisecond rendering barrier. So if you got a lot of GCs or some long ones that are occurring right after each other, it might put your frame processing time over the 16 millisecond rendering barrier, which will cause visible hitching or jank for your users.

* Secondly, understand that your code flow may be doing the kinds of work that force GCs to occur more often, or making them last longer than normal duration. For example, if you're allocating a hoard of objects in the inner most part of a loop that runs for a long time, then you're going to be polluting your memory heap with a lot of objects and you'll end up kicking off a lot of GCs quickly, due to this additional memory pressure.

<<<<<<< HEAD
And even though we're in a managed memory environment, memory leaks can still happen. Although they're not as easy to create as the other languages. These leaks can pollute your heat with objects that won't get freed during a GC event, effectively `reducing the amount of usable space you have` and `forcing more GC events to be kicked off`, in a regular fashion as a result. So that's the deal, I mean, if you want to reduce the amount of GC events that happen in a given frame, then you need to focus on optimizing your app's memory usage.
=======
And even though we're in a managed memory environment, memory leaks can still happen. Although they're not as easy to create as the other languages. These leaks can pollute your heap with objects that won't get freed during a GC event, effectively `reducing the amount of usable space you have` and `forcing more GC events to be kicked off`, in a regular fashion as a result. So that's the deal, I mean, if you want to reduce the amount of GC events that happen in a given frame, then you need to focus on optimizing your app's memory usage.
>>>>>>> Merge branch 'master' of https://github.com/lchang199x/lchang199x.github.io

From a code perspective, it might be difficult to track down where problems like these are coming from, but thankfully, the Android SDK has a set of powerful tools at your disposal. Let's take a look. 

### Memory Monitor Walkthrough
Let's go ahead and look at a tool called `Memory Monitor`. Memory Monitor is a tool that can show you how your application is using memory over time. Let's go ahead and start it.

As with other demos, if you want to follow along, make sure you start Android studio. Connect the physical device, enable debugging, and then make sure your Sunshine application is in the forefront. So if we want to start Memory Monitor, we can do so through the menu, like so. We're going to go to tools, Android, Memory Monitor. Or if you have an icon here at the right, you can go ahead and click that. So let's do that. Okay, so the tool opens as a tab at the bottom right of your Android studio window. And if it finds your application, it's going to immediately start recording memory usage right from your app.

Just like you should be seeing right about here. At the top left of your Memory Monitor window, you will be able to toggle your currently connected devices. And on the right, you can select the application you want to monitor. The stacked graph that takes up most of the window shows the total memory available to your application. The amount of memory that your application is currently using is shown in dark blue. And the free memory, or unallocated memory, that's available to your application, is shown in light blue or light gray. The graph updates continuously to show you changes in memory usage. And it shows how much memory the app has available to it over time.

Now, in a world where your app isn't doing much of anything, you should see a flat graph like this one. From a performance perspective, this is actually an ideal scenario. But as your application allocates and frees memory, you'll see that the allocated amount changes in your graph. And if your app suddenly needs a lot more memory, the overall memory allocated for your app will also increase, represented by this box here. Because if it didn't, your app would run out of memory and essentially crash.

Now anytime you see the allocated memory drop by a significant amount like here, that's a pretty good signal that the garbage collection event has occurred. Now in this example, the garbage collection looks pretty healthy. On the other hand in this graph it looks like there might be some problems. You see, the app allocates a lot of memory within a short time, then frees that memory almost immediately, creating these narrow, skinny spikes. And it does it over and over. This means that your app is spending a lot of time garbage collecting. `And the more time you spend doing GCs, the less time you have to do other stuff like rendering or streaming audio`.

So let's see what this looks like live. Okay, so here we are with memory monitor running on Sunshine and once we start tapping on a day and then looking at the detail view and then say, tap the back button or say we do this a few times. We're going to see that our memory starts to fill up. Just like around here. And say we were to go ahead and change the zip code a few times just to get some new data. And go back and check out some more detailed weather information. Cool, looks like it's going to be clear on Wednesday. Ooh, a little rain on Friday. So gradually we're seeing how memory is filling up.

And eventually, it will actually grow until we actually have no more free memory left. If we keep going, at this point, a garbage collection event will be triggered to free up a chunk of memory. You can see this drop happen right here. Now remember, this does not free up all the memory because Android's memory management system is generational in nature.

In our nifty tool, we can actually force individual garbage collection events. There's this garbage truck tool here on the top-left of the Memory Monitor. If you press it, it's going to go ahead and trigger an individual garbage collection. Notice what's happening here on the right of the graph. Now, we can go ahead and actually press it a few times. And if you do that a few more times, you can free all the freeable memory like so. And this seems to be back to our original state. Okay, so next, we're going to take a look at memory leaks and the heap viewer tool. And finally, we'll use all of these together to find and fix memory leaks in some real code. 

### Memory Leaks
One of the best things about Android's Java language is that it's a managed memory environment that is, you don't have to be super careful about handling when objects are created or destroyed. While this is generally great, there's some hidden performance problems lurking under the surface here.

Now remember, the memory heaps in Android's runtimes are segmented into spaces, based on the type of allocation and how best the system can organize allocations for future GC events. And each space has its own reserved memory size. When the combined size of an object in a space begins to approach its upper limit, a garbage collection event is kicked off to free up space and remove unneeded objects.

These GC events aren't generally a noticeable problem to your performance. However, a lot of them recurring over and over and over and over again can quickly `eat up your frame time`. The more time you're spending doing GCs, the less time you have to do other stuff like rendering or streaming audio. One common situation that developers can fall into that cause a lot of these GCs to occur is known as memory leaks.

Memory leaks are objects which the application is no longer using, but the garbage collector fails to recognize them as unused. The result is that they stay resident in your `heap`, taking up valuable space that's never freed up for other objects. As you continue to leak memory, the available space in your heap's generation continues to get smaller and smaller and smaller, which means that more GCs will be executed more often to try to free up space for normal program execution.

Finding and fixing leaks is tricky business. Some leaks are really easy to create, like making circular references to objects which the program isn't using. While other are not so simple, like holding on handles to class loader objects as they're being loaded. In either case, a smooth running, fast application needs to be aware and sensitive to memory leaks that may exist. I mean, your code's going to be running on a federation of devices and different types, and not all of them are going to have the same memory footprints and sizes. Thankfully, there's a simple tool that's available to help us see where these leaks might exist inside the Android SDK. Let's take a look. 

### Heap Viewer Walkthrough
Okay. To get more information on the state of our memory and the objects that are taking up space, we can use a handy tool called `Heap Viewer`. Now with Heap Viewer, you can see how much memory a process is using at certain points in time. Now as before, if you want to follow along, go ahead and start Android Studio and bring sunshine to the foreground on your connected device. 

In order to start Heap Viewer, you'll want to start Android Device Monitor first and there are a few options to do that. One way is through the tools menu where you can click on tools, android, and android device monitor. Or, you can click on this nifty android icon here in your tool bar at the top. So I'm going to do that and android device monitor is starting, then we're going to want to go ahead and click the DDMS tab. The heap viewer is one of the DDMS tools and we're going to go over here to the left. And we're going to select the app we want to profile, so we're going to select Sunshine now I'm going to pull up this panel down here.

So once you have Sunshine selected you're going to want to select this heap tab to get more information. Now initially you might not see much, but you notice this little. Hint here at the top that reads `Heap updates will happen after every GC for this client`. Why don't we go ahead and click on this and cause a GC to update your data. Whoa, look. We have all this new information now. Now the table updated and shows you what data is currently available and alive on the Heap.

If you want to get some more details go ahead and select a single data type. I'm going to click this class object. Now you'll see a lot of data will update in the panel down below. You can now see a histogram for the number of allocations and also the specific memory size for that data type. In this case we're talking about the class object. Now the heap viewer is really helpful to see what types of objects your application has allocated, as well as how many and what sizes they are on the heap.

Again here, see we see the total sizes of particular types on our heap. Like for example there's over 1400 two biter arrays on our heap, that's taking about 120 kilobytes. Whereas there are only it's roughly only taking up about two megabytes. Now the heap viewer is really helpful to see what types of objects. You application is allocated. As well as how many, and their respective sizes on the heap. For example, if we look here we have 27 one byte arrays that are taking up roughly two megabytes of data. And then we have about 2000 four byte arrays that are taking up currently 228 kilobytes of data. This information is super helpful when you're trying to track down memory leaks. 

### Spotting Leaks In Memory Monitor
All right, so let's talk about memory leaks. Memory leaks are sneaky. They can be slow and insidious, sometimes taking days or weeks before you even realize that you have one. In fact, you might only realize memory's an issue when your users start complaining about mysterious slowdowns that happen after using your app. Don't let this happen to you. Fortunately, with some patience, a perf mindset, and the right tools, you'll have the opportunity to abolish these leaks from your app, period.

We'll use Memory Monitor to watch the behavior of a leak in action, and then in the next video, we'll `use heap viewer to gain initial confirmation`. Now let's look at a micro example of what a leak can look like, and see how the SDK tools can help us identify such a leak. In this example, we're going to go ahead and rotate the device for a few minutes and profile it with Memory Monitor. This is by design to showcase a common leak situation that can arise during the creation and destruction of an activity.

We can intentionally trigger this cycle by changing the device's orientation. And yes, I know, it may seem that this is a totally weird thing to do, but we're going to do this to demonstrate how a leak may happen and to show how they can be slow and insidious. Now, in the first pass, the leak slowly consumes the free memory available to your app, until eventually it causes a garbage collection or GC event. More important, the key thing to notice is, the garbage collector isn't able to reclaim that much energy due to the leak in the app.

And then, eventually, a second GC event occurs much sooner, about 30 seconds later. Now, notice when the leak consumes all of the free memory, Android actually adjusts and grants the app a higher memory ceiling. While this is a nice adjustment by the system, if the leak isn't fixed, it will keep consuming memory until the system can no longer allocate any more. This will slow the performance of the device and eventually lead to your app crashing. You can wait a little bit longer, and the third GC will occur. And then a fourth somewhat similar to the first two. Now, as you can see, the pattern continues, and more memory is allocated by the system. You can also similar behavior using heap viewer.

### Leaks Continued With Heap Viewer
Using heap viewer, we can see that after the first GC, only 1.39 megs is free. This may indicate that the garbage collector wasn't able to reclaim much memory due to a leak. After a second GC event, heap viewer indicates that the system has decided to accommodate a larger memory footprint for this app by allocating more memory. Increasing the heap size to 32 megabytes, which is up from 20 megabytes in the first GC. This time, we have 12.9 megabytes free in our heap. At this point, the system is dynamically accommodating for the larger memory footprint of this app.If the expansion repeats, this may lead to an app crash if the system can no longer allocate more memory for the app. So remember, memory leaks are slow and they're insidious and require time and the proper test environment to confirm.

Also, keep in mind that sometimes a pattern like this might represent a legitimate use of memory. For example, imagine an application that was designed to manipulate large graphics or photos. The takeaway here is be on the lookout for slow leaking memory, but always weigh the data you collect, against the memory implications of your app's core functionality. Now at this point, you should understand how memory leaks manifest in SDK provided tools such as Memory Monitor and Heap Viewer. But you might not know where they originate. Here are some best practices you can take to avoid a leak. `Track the life of your objects throughout your code` and `clean up references when you no longer need them`. Okay, so in the next slide, we'll identify what might be causing this leak.

### Tracking Down the Leak in Code
Looking at the custom view's init method:

```java
private void init() {
    ListenerCollector collector = new ListenerCollector();
    collector.setListener(this, mListener);
}
```
It’s a seemingly harmless idea to store away all view listeners for a particularly activity, but if you forget to clean them up, you may inadvertently create a slow leak. This is the case with the line:

> collector.setListener(this, mListener);

This problem is compounded when the activity is destroyed and a new one is created. In this example, when a new activity is created due to the device orientation changing, an associated Listener is created by the view, but when the activity is destroyed, that listener is never released. This means that no listeners can ever be reclaimed by Java's garbage collector, which creates a memory leak.

When the device is rotated and the current activity’s onStop() method is invoked, make sure to clean up any unnecessary references to view listeners.

### Memory Leak In Allocation Tracker
Now what's also interesting, is that you can use the `allocation tracker` tool to identify the extraneous memory bloat, that arises from stale views residing in memory. Now as you can see here, I've selected a common set of objects or classes that are still residing on the heap. Now these objects are put onto the heap, when we call onCreate on this particular activity. Now each time the device is rotated, a new activity is created and thus a similar set of objects are basically inflated and put on the heap. So if a leak exists, and we rotate the device, the garbage collector won't be able to clean up these items and will essentially replicate a large set of these on the heap. An allocation tracker will help you see this. 

### Understanding Memory Churn
Now that we've found a way to clean up all those nasty leaks, we run into an even larger problem called, memory churn. Remember that `each heap space has a finite size that limits the number of objects that it can accommodate`, and as the size of the heap gets too big, a GC event is kicked off to eliminate unneeded objects to free up memory. Now, `memory churn is a term used to describe the process of allocating lots of objects in a very short amount of time`.

For example, if you're allocating a load of temporary objects in the middle of a for loop, or you're allocating a bunch of objects in your on draw function. This is effectively the same problem as an inner loop, any time the screen needs to be redrawn or an animation is occurring, you'll end up with calls to these functions every frame, which can quickly add up, adding pressure to your heaps. In both cases, we've created a scenario where a high volume of objects can be created in a very short period of time.

And depending on how many of these objects are created, or their size per object, you can end up quickly consuming all of the available memory in the young generation, forcing a GC to kick off. And as more of these are kicked off, the more of your precious frame time is going to be eaten up by them. As such, it makes sense that for a high performance application, you would want to identify and move allocations out of inner loops or any code that's going to be executed repeatedly. To make finding these allocations easier, Android Studio has a handy tool built just for this problem. 

### Allocation Tracker
All right. Here we are again with another handy tool. Now looking at a heap view of your apps' memory allocations is a great way to understand where most of the data is going and what type of data is being allocated. This helps you find data that is allocated and maybe shouldn't be. But sadly, heap viewer doesn't tell us `exactly where in your code that data is being allocated`. So for that, we need a tool called Allocation Tracker.

As usual, go ahead and start Android Studio, connect your device, and load Sunshine into the foreground. Okay. So if we want to get to the tool in Android Studio, we can click on the Android tab, which is at the bottom left of a standard Android ID window. This is going to open the Android DDMS tab. I'm going to go ahead and enlarge this so we can see this a bit easier. So, the DDMS tab shows the devices and logs related to your device. Now depending on what you've been working on in Android studio, this tab will show different views. Make sure the Devices tab is the one that you have selected.

And we'll go ahead and select the Sunshine app. Now next in the left margin go ahead and click the bottom most icon. This one right here. The icon has a `start allocation tracking tool` tip. Okay. So, the DDMS tab shows the devices and log. I'm going to go ahead and click it. And see, we're enabling allocation tracker here.

All right, so go ahead and bring up your Sunshine application and go ahead an interact with your app for a little bit. So let's go ahead and take a look at what the weather is like in Cleveland. Unfortunately looks like there's some rain tomorrow, but a high of 74, humidity is 93. Ooh, sticky. And we see rain Wednesday, still pretty warm. And then rain on Saturday, but still pretty warm. Fortunately, less humidity then. Looks like you got warm and rainy weather, Cleveland. All right so go ahead and you can finish interacting with your app a few times. And when you're done, click on the same button as before. And the tool tip should say `Stop Allocation Tracking`.

Now depending on how long you've been doing stuff. Processing the data might take a little while. And the button will stay depressed. Don't click it multiple times. Just be cool and hang out for a bit. All right. It looks like our's finished. Now if you look closely up here, a new tab was created in our IDE. It's labeled something like DDMS followed by a large number that we have it right here.

Now, this view lists all the allocations that occurred during that sampling period when you were interacting with your application. And each row in this view represents distinct allocation. Now the order column is going to tell you. When the allocations happened. The allocated class column is going to show you what type of data was being allocated, the size of it, and you'll also have some information telling you what thread made that specific allocation. Lastly, the allocation site column tells you what function in your code actually allocated that memory.

For example here, we select integer. The value of method is what made this allocation of this integer. Now if you click on an allocation, you can see its full call stack. There's a lot of information in this table. If you don't see everything, you may need to resize the panels. Now, you can also move the columns around to your fitting to get the ones that you care about within view. Or you can sort columns by clicking on the header of a column. Now obviously this tool is super handy for tracking down memory churn. Let's go ahead and take a look at this in practice. 

### Identify Memory Churn using Traceview
For this exercise let's launch the memory churn activity. Oh hello there pirate. Now press the button, do interesting things with arrays. I'm sure you noticed that the dancing pirate hangs or pauses, but then eventually resume dancing. As you know, this is Jank and it's bad for many reasons. So let's go ahead and fix it. Now this should sound familiar, so go ahead and profile this activity using trace view and when you're done, you should see a tool output similar to this one. Notice the frequent GC events that are happening within a short period of time. Now given what you know about garbage collection, what do you see in this timeline view that might be hurting app performance? Write your observations in this box. 

Now if you guessed too many and too frequent garbage collection events, then you'd be spot on. Also note that we could capture this memory monitor as well. Now this screen shot illustrates how memory churn would manifest using the memory monitor tool. 

### What’s Causing the Churn
Now that we've used the SDK tools to capture enough data to know when we have a memory churn situation, let's zero in on the code that's causing it. Well it turns out TraceView offered us a clue. Let's take a closer look at the profile view, when selecting a method on the main thread. Now if you were to scrub the main thread's methods, you'd find multiple recurrences of Java character array copy operations, like this one. Now walking up the call stack, we get more confirmation that the array copies were used to enlarge a string buffer.

```java
// Now go through and dump the sorted version of each row to output!
for(int i = 0; i < lotsOfInts.length; i++) {
    String rowAsStr = "";
    for (int j = 0; j < lotsOfInts[i].length; j++) {
        rowAsStr += getSorted(lotsOfInts[i])[j];
        if(j < (lotsOfInts[i].length - 1)){
            rowAsStr += ", ";
        }
    }
    Log.i("CachingActivityExercise", "Row " + i + ": " + rowAsStr);
}
```

So now let's look at the source code for the MemoryChurnAcitivity. As you can see here in the OnClickListener, we go ahead and call this function imPrettySureSortingIsFree, so let's take a look at that code. Now, in this method aptly named imPrettySureSortingIsFree, the code is constructing a new string, one cell value at a time, using string concatenation. See the instruction notes for the code I'm referring to. But, here's specifically, where the concatenation occurs. Now, it might look harmless at first, but, why would this code lead to memory churn? 

The frequent garbage collection events are the result of two things. Number one, each cell value concatenation requires the creation of a new character array and this is compounded by it occurring in rapid succession within the loop. So, it's also this one. Now, you can also confirm this character array bloat via allocation tracker. We're going to go ahead and improve the code and let's see what happens in the next video. 

```java
// Now go through and dump the sorted version of each row to output!
// This time, use a StringBuilder object so that we can construct 
// one String per row, instead of wasting String objects with
// ridiculous concatenation that never ends.
StringBuilder sb = new StringBuilder();
String rowAsStr = "";
for(int i = 0; i < lotsOfInts.length; i++) {
    sb.delete(0, rowAsStr.length()); // clear the previous row
    for (int j = 0; j < lotsOfInts[i].length; j++) {
        sb.append(getSorted(lotsOfInts[i])[j]);
        if(j < (lotsOfInts[i].length - 1)){
            sb.append(", ");
        }
    }
    rowAsStr = sb.toString();
    Log.i("CachingActivityExercise", "Row " + i + ": " + rowAsStr);
}
```

### Improve Your Code To Reduce Churn
We can make a small adjustment in our code to prevent excessive churn. Let's take a look at a comparison view. Rather than concatenate one cell value at a time to build each row, let's use a StringBuilder instance, and construct each row using a single string. Note that the StringBuilder's `instantiated outside the loop`, and thus its memory is allocated once.

And then we simply use it as a buffer for each iteration of the loop where we first clear it, and then we append a single string of ints to represent the row for that loop iteration. Now see the instructor notes for more details into this code segment.

All right, now it's time to verify. You want to go ahead and load the improved branch of code, which is called memory_churn_optimized, into both trace view and memory monitor to confirm we've reduced the amount of GC's occurring in the short time window. You may also use allocation tracker to verify. If you use allocation tracker, or if you got something unexpected in trace view, or memory monitor. Share a screenshot of your output in the discussion forums. We're interested in seeing what you've got.

Now for us, even with these changes, the Perf pirate still pauses. But this time for less time. Now this point, this also might mean that this function is probably a good candidate to be backgrounded. 

### Recap
To recap, you learned how to use multiple tools to diagnose memory problems. Now we started with our familiar friend `Trace View`, but you also learned about some memory-specific tools including `Memory Monitor`, `heap viewer` and `Allocation Tracker`. But most importantly, you want to evaluate your problematic code against several tools and seek multiple perspectives as to what is actually happening underneath the hood.

Finally, it's important to understand each tool has a strength.  
* Memory Monitor's a great way to get a dynamic view of your memory over time, and

* Heap Viewer's great for showing what exactly is on your heap.

* And lastly, Allocation Tracker shows you where a particular location came from your code.

### Dialogue: Outro 
*&gt;&gt; So let's change this.  
&gt;&gt; This line is-  
&gt;&gt; Man this managed memory thing is so cool, so much better than C++ and a ton faster. I think I never have to worry about performance again.  
&gt;&gt; [SOUND]*