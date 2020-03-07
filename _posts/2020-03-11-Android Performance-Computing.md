---
layout:     post
title:      "Android Performance: Computing"
subtitle:   Android系统性能 (3)
date:       2020-03-11
author:     Cliu
header-img: img/post-daily-bg.jpg
catalog: true
tags:
    - Android
---

> Based on course materials of [Android Performance](https://cn.udacity.com/course/android-performance--ud825) by Google



### Dialogue: Intro
*&gt;&gt; Hey cool.  
&gt;&gt; Hey what's going on man?  
&gt;&gt; So I found this stack overflow post, that says if I change the syntax of my for loop, and use a pre-increment over post-incrementor, I'll get like a hundred x performance on my for loops. Pretty awesome yeah?
&gt;&gt; Mm, that's not a thing, Chris.  
&gt;&gt; What do you mean? That seems really useful, all I need to do is plus plus i versus i plus plus, I mean, then I get a boost for every loop I write.  
&gt;&gt; Chris you should probably stop now, I mean we are recording, it's, it's not a thing.  
&gt;&gt; Jeez. That's kind of harsh, if you know better, why don't you show me then? Let me check this out.  
&gt;&gt; Okay, so here's the deal, you need to remember that Java running on Android is effectively executing in a virtual machine environment, which means there's lots and lots of layers of complexity going on here, from the precompiler to the compiler to the optimizer to the code itself, actually running on the device. What you're identifying here is something that we call compute performance.  
&gt;&gt; So, like, the performance of my computer?  
&gt;&gt; No, more like how the algorithms or the computing processes are performing. Which has everything to do with how the compiler is generating the code and how the virtual machine itself is actually executing it on the hardware. See, what this post has identified was a very, very specific instance where the compiler could make a pre-fetching optimization on gigantic four loops where a collection class has more than 20,000 elements in it. Effectively by changing the syntax there for your incrementer it was able to hint to the compiler that this particular type of optimization could actually be made.  
&gt;&gt; Totally. You know, I was hoping, I could use it in general, though.  
&gt;&gt; Well, see, it's not that easy when you actually want to get huge wins in computer performance scenarios, it actually means understanding what each piece of code is actually doing on the hardware. Which usually means, having to go and make small little changes throughout your entire code base, just to get the performance wins that you're actually looking for.  
&gt;&gt; Jeez, that sounds like a lot of work, I'm already exhausted.  
&gt;&gt; Chris, my friend, it's the whole reason I'm bald.*

### Slow Function Performance
So let's start in a place very familiar to all of us, slow function performance. This is your basic computer science 101 concept of performance. That is, you've written some code and it executes more slowly than you'd actually expect. Now, this often happens innocently enough. I mean, you're focused on creating some code change to solve a particular problem in a particular way. But soon you realize that it's taking much, much longer to execute that code than you'd like. 

Now the primary reason that code can be taking too long to execute has everything to do with how the language and of course the associated hardware is handling the execution of your code. For example on some older hardware which we will not actually name executing a branching statement on a floating point comparison took almost 4x as long the same thing on integers or booleans. The reason for this was the chip architecture. The part of the CPU dedicated to floating point calculations occurred after the Branching logic stages. Meaning that any floating point comparisons would have to wait until the end of the cycle pipeline, stalling the rest of the operations until it could finally execute its branching logic. But, listen don't freak out here. Modern hardware generally doesn't have these type of nuanced issues to deal with. But it does illustrate a very, very important point. How you write you code affects performance, depending on how the language executes on the hardware, all the way down to how your silicon chips are actually structured. Let me be clear on this. To optimize your code, you have to understand the system that runs it. 

Now, slow function performance generally comes in two flavors.

* Firstly, you have your single slow function form. Now, this is pretty straightforward. You've got some function that's taking like, 2x or you actually want it to. This is actually pretty easy to fix. I mean, you find the slow function, take a look at its code, figure out what the problem is, and then try a few fancy ways to fix it.

* Now much harder to figure out is our second type. Death of a thousand cuts effectively this is when you have a hundred functions, each taking one millisecond longer than it should resulting in a hundred millisecond extra in your overall program execution. This type of problem is the hard one to track down and even harder to fix because you usually end up slogging through every piece of your executed code to find small wins that in the end, can make the difference between shipping and your company going under. Now, fixing these tiny performance problems is all about `profiling`. That is, timing your code to figure out what portions of it are running slower or longer than the others and then making some small tweaks and then timing your code once again. And, once you find one of the offending functions, you'll then need to start timing individual lines of code in that function and in all the functions that it actually ends calling later. 
Now this can get really gnarly unless you're an expert in the field but fear not. The Android SDK has some excellent tools to help you track down these problematic portions of your code so you can fix them up right. Let's take a look. 

### Traceview Walkthrough
Let me show you how you track out some compute-based performance problems in your apps. For this demo, we're going to hook up the Sunshine application with a profiling tool called Traceview. Let's go ahead and load it up.

First, make sure, your device is connected. And then, start the application you want to profile. In this case, we'll bring up Sunshine. Okay. And now, let's hop back to Android Studio. And in order to access the `trace view tool`, we're going to want to start the` Android Device Monitor`. Now, you can either do that through the Tools menu by going to Tools > Android and Android Device Monitor. Or you can click this little green guy here up at the Toolbar in the Android Studio IDE. So I'm going to go ahead and click this button. Okay. So once Android Device Monitor is up and loaded, make sure you come over here and check your tabs and make sure that the DDMS tab is selected. Then you'll want to hop over here to your Devices pane and select the activity that you want to profile. We're going to go ahead and check Sunshine. Okay, so now I want to draw your attention to some icons here in the taskbar. Particularly this one right here, that looks like three arrows with a red dot on top of it. This is the button that we can press that has a little tooltip that says `start method profiling`. This is how we invoke Trace View.

Let's go ahead and click it. You'll get a pop-up with two ways of profiling your app. You can either record every method's entry and exit which is very resource intensive down here, or you can do some sample based profiling. What that means is by default a profile is going to ping your application `every one millisecond` to find out and record what function is actually being executed. So let's go ahead and use the default settings. I am going to go ahead and click OK. So now that the profile is recording, let's go ahead and interact with our application and see if we can profile some actions. So I want to hop over here and interact with Sunshine a little bit. Oh, nice. Some pretty good weather here in Mountainview. Unfortunately not so good coming up this weekend. Looks like we have some rain in store. But, why don't we go down the coast and see how our friends are doing in Southern California? Oh, weird. They've got a strange winter, very untypical of San Diego. How about we check on our friends in Texas. All right, well maybe a little bit more warmer weather, there. Not too bad tomorrow. All right. So let's go back to Android Device Monitor, where we want to ` stop our profiling`.

Now we can do so by clicking the same icon that we did to start it. It's got this black icon or black square on top of it now so let's go ahead and stop our profiling by clicking on it. Now, it might take a few moments to `load the trace`. Which is going to show up here in the tab at the top of your window. Just keep in mind it might take a little longer depending on how much recording you actually did. All right. Let's, so let's talk about trace view. Now trace view has two main components to it. It's got this top panel here called the `timeline panel` and then it's got this bottom panel with a lot of information. It's called the `profile panel`. 

Now the timeline is great in showing you how your code is executing over time. Each row that's shown here in the, in display, actually corresponds to a thread, and each color that's displayed corresponds to a particular method that's running. So for example here we have all the activity on our main thread. And we can, we can see spikes when methods were being started and stopped. What's even more useful is we can zoom down and we can try to figure out particular methods and how they're behaving. They kind of show up in these U shapes. With the left bar here that denotes when the method is actually starting and then another bar here on the right which actually denotes when the method is finishing. And the width of the bar actually represents how much time it took for that method to be executed.

Now let's go ahead and select the particular method. Let's hop down here to the bottom of the trace view window and here we're going to look at some profiling data that shows up. Particularly, we get some information about what methods actually called the one that we've selected, which are represented by their parents in this blue highlight here. And we actually get some additional information about which methods are actually called within this one. So, in other words we called dispatch input event method with a native pole once which is what is selected at the moment. And for each of these methods that we have selected we get a whole lot of additional statistical information.

For example, we have the `exclusive CPU time`. This is the time that's spent in our method itself. And we can use it to find specific issues for that particular method. Now the inclusive CPU time is for the particular method and all the methods it calls internally. This can help you find problems within the individual indication tree for the method that you selected. Oh, and another really useful statistic is how many times the method was called or called itself recursively. We can find this information if we scroll to the right and we've got this column here called `calls and recursion`. Again, that measures how many times the method was called or how many times it was called recursively. Now there's a whole lot of additional information here in this profile panel. And if you don't have a really big display it's kind of hard to see. So you have to do some scrolling back and forth to kind of find the metrics that you care about. And don't forget this little nifty search box that can help you kind of zoom in on the functions that you care about. Okay so that's a little trace for you let's go ahead and use it on some real code.

### Batching and Caching
I want to introduce you to my two most favorite performance techniques, batching and caching.

As we already talked about, some functions or operations have a specific amount of overhead involved with them that is different than the performance costs of the operation itself. For example, loading data into a new place in memory before executing on it, or sorting a set of values before doing a search through it. When executed multiple times, where multiples are really big number, this overhead can become a serious performance burden for your application. `Batching is the process of fixing this performance problem by trying to eliminate the per execution overhead of these operations`, kind of like sharing a car instead of everyone driving themselves, thus saving gas. This is most often seen in your code where you have to prepare your data before actually operating on it.

Now, for example, let's say the most efficient way to find a value existing in a set is to sort it, and then execute a binary search on it. Now, wait to, to be clear, this isn't actually the most efficient way, but just stay with me, I'm trying to make a point here. Anyhow, the simplest way to do this would be to write a function that, given a set and a value, would sort the set and then search it to see if the value exists in it. Now, this may be fine for some performance level but let's say you've got like 10,000 values you want to test, and the size of the set is in the millions. Suddenly you're incurring a ton of overhead per test in the form of the sort. The answer here is pretty clear. You'd want to create a sorted version of the set once, and then allow all 10,000 values to be tested for inclusion after that point. `This is batching in action. We factored out the operation that is repeated and do it over once`.

Now, similar to this is actually a concept known as caching and is by far the most important performance technique you can understand, mainly because it drives everything in modern computer technology. Take your computer, for example. The whole point of your RAM hardware is to provide a place to store information that's faster to access for the CPU than the hard drive is. Or take networking, and look at the modern internet itself, huge warehouses of servers called data centers exist all over the world. Their only purpose is to store or `cache frequently accessed content` so that your computer doesn't have to hit a server 12,000 miles away every time your friend in Egypt posts a picture. Well, unless of course you're in Egypt, but, you know, you get the point.

Now in your code, `the most common place that you can find optimizations for caching has to do with data that is calculated multiple times, but the result is always the same`. For example, if you're in the middle of a loop that you're calculating the derivative of a four by four matrix, and that result is always the same, then you're actually wasting performance, recomputing it for each iteration of that loop. Instead, compute and save the results of that derivation outside of the loop, and then let your inner portions of the loop reference that cached result.

The reason that I love batching and caching so much is that pretty much every performance improvement that you can think of, including the ones that we're talking about in this course, is effectively a variance of these two basic techniques. And if you're serious about becoming a performance ninja, then you better become a pro at what it means to leverage the awesome power of these techniques. So, let's get started. 

### Fixing a Problem with Batching and Caching
```java
public int computeFibonacci(int positionInFibSequence) {
        if (positionInFibSequence <= 2) {
            return 1;
        } else {
            return computeFibonacci(positionInFibSequence - 1)
                    + computeFibonacci(positionInFibSequence - 2);
        }
    }
```
So the answer we're looking for here is actually the computeFibonacci method. So let's review what I did to figure it out. All right, so running Traceview on the following activity, and basically profiling the function that happens when I press this compute Fibonacci function. This is what the Traceview output looks like. All right, so here's an output that I got from running Traceview. You should see something similar. Notice this large pink area. This is a bad thing, this basically indicates that something is taking up a lot of CPU time on our main thread. So if you `sort by exclusive CPU time`, or by hovering over this pink area. You'll notice that the computeFibonacci method, which is coming from our caching activity, is the one that's actually occupying the most CPU resources. So this is something we want to fix 

All right, so what are good ways to fix this problem? Go ahead, and select all that apply. 
* Move computeFibonacci's code off the main Thread
* Batch the method calls to computeFibonacci and wait to execute them
* Remove the computeFibonacci method
* Cache intermediate values inside of computeFibonacci

Now, we shouldn't be doing additional work on the main thread that isn't necessary, right? So, let's leave that thread only to handle user input and drawing to the screen. Now this is a good intuition in general, but for the sake of this example, let's see if we can optimize this function to run much faster and have less computer overhead using one of those handy techniques called shared. So let's go ahead and cache intermediate values. 

```java
public int computeFibonacci(int positionInFibSequence) {
        int prev = 0;
        int current = 1;
        int newValue;
        for (int i=1; i<positionInFibSequence; i++) {
            newValue = current + prev;
            prev = current;
            current = newValue;
        }
        return current;
    }
```

It is important to understand what your code is doing, no matter how simple the task. For example, most people know better than to compute Fibonacci numbers recursively, but it is not unusual to unintentionally redo work in your application. Check your app for places where you can cache current results for future re-use.
 
In this case, recursive Fibonacci calls fib8 which calls fib7 and fib6, but that fib7 call calls fib6 again and fib5, So now you've got two fib6's and one fib5 call, but each of those fib6 calls will have a fib5 and fib4, so now you have three calls to calculate fib5, blah, blah, blah.  Recursive fibonacci is terrible.  Iterating lets you calculate fibX once,use that result twice, and move on.

### Blocking the UI Thread
Now, it's important for performance that each individual function runs as efficiently as possible, but equally important for performance is when and where in your code that function executes.

See, whenever you first start an Android application, the main thread of execution is created, this main thread is very important because it's in charge of running your code, dispatching events at the appropriate views, and executing your drawing functions, which we've kind of already talked about. Basically, this main thread is where your application was. Now the main thread is also sometimes called the UI thread, because this is also the thread your users interact with. For instance, if you touch a button on the screen, then the UI thread dispatches a touch event to the view, the view sets the button state to pressed, and posts a validate request to the event queue, then, the UI thread processes the request and notifies the button to draw itself in a pressed state.

Whew! Now if you have any on touch event handlers, those will be executed in the middle of that huge flow, and the longer it takes your on touch functions to process, the longer it will take, before your draw function is ticked off, and the view is updated visually for your users to see. The takeaway here is that `your input handling code is sharing cycles with your rendering and update code on this same thread`. This means the UI can't draw while you're updating in the middle of a computation that may be dealing with touch events, network access, or database queries.

In simple situations, this can cause you to miss the 16 millisecond window dropping a rendering frame and letting your user experience annoying lag. However if you end up pausing the rendering of your UI thread for more than five seconds, the users presented with the infamous application not responding dialogue. Which basically asks the user if they'd like to close your application, which I'm not sure is ideal in terms of user attention. Right?

So how do you fix this? Well, you identify any pieces of work that do not need to be done on the main thread that is `the work doesn't need to be completed in order for the draw to occur`. And then you move that work onto a separate independent thread of execution, where it doesn't block the UI thread. For example, if pressing the submit button completes an order, then composing and sending the confirmation e-mail can be done on a separate thread. Now that may sound daunting but don't worry, Android has a cool set of APIs to make this easy for you to do. 

### Use Traceview to Identify Problems
In this exercise, I'm going to show you how to use traceview, to identify a function that's messing with your apps `frame rate`. Now if you want to follow along, you'll need to download the compute and memory sample app from the linked in the instructor notes below.

```java
public void sepiaAndDisplayImage() {
    Bitmap loadedBitmap = BitmapFactory.decodeResource(getResources(), R.drawable.punk_droid);
    
    int width = loadedBitmap.getWidth();
    int height = loadedBitmap.getHeight();
    Bitmap sepiaBitmap = Bitmap.createBitmap(width, height, loadedBitmap.getConfig());
    
    // Go through every single pixel in the bitmap, apply a sepia filter to it,
	// and apply it to the new "output" bitmap.
	...
}
```

After you install the sample app, go ahead and press the slow on click handler button, you'll notice the familiar dancing pirate. Then go ahead and press the button labeled, display an image. See as you can see, the pirate's dance pauses and then resumes again when you do this, and then the Android image is displayed underneath the button. Just like before, it seems like pressing the button, relates to the pause in the pirate's dance, so hopefully, you know what comes next, and more importantly what tool to use.

We're going to profile the app using traceview, so let's go ahead and do that. All right, so I want to come over here to my devices list and make sure that the compute activity's selected, before we start profiling, I'm going to click on the slow unclick handler to launch the activity, let's go ahead and start a trace, like so. Go ahead and hit OK, all right, so the trace is running now, let's go ahead and click this display an image button, and there we go. Let's stop our trace. So let's go ahead and look at our output. All right, so we have a lot of data here in our output, but what does all of it mean? Now here's a task, look through the traceview output and see if you can identify some methods coming from the sample app, that are using significant CPU resources. Jot down your top two answers here. 

All right, so let's take a look at some Traceview output. Notice this big section of activity here, but let's talk about a few observations. You might notice that the top function here is the most resource-intensive when sorted. Now, there are a few others, such as this nativePullOnce. But these are system methods that we don't own. So if we go down a little bit further, notice that we have this nativeSetPixel and this nativeGetPixel.

So let's see where they're coming from. Let's go ahead and expand a little bit. Ah-ha, here we go. So setPixel looks like it's being called from something within our busy UI thread activity, which is from the sample app code. And the same is the case for getPixel, which also seems to be coming from our busy UI thread activity. So now that we've identified that setPixel and getPixel come from our busy UI thread activity, let's go ahead and explore them further. 

So, getPixel and setPixel aren't methods that we wrote. So what parent method in the code is actually calling getPixel and setPixel? Write the method down in the box here, and make sure to use proper casing. 

All right, so the parent method is actually sepiaAndDisplayImage. So let's go back to trace view and see what we mean. All right, so here we are back in trace view. Now if we expand our collapsible menus here, we can actually walk up the parent calling stack and see that set pixel is actually called by sepia and display image. Now, this is within our busy UI thread activity. So, let's go ahead and see how we can optimize.

Decoding an image is a substantial task. It's also a task that we can't easily optimize away. A general rule of thumb for long running tasks, especially those involving network access, lengthy database calls and image manipulation is to move them off of the main thread.

You have a few options for this in Android. We're going to use an AsyncTask.

```java
private class SepiaFilterTask extends AsyncTask {
    protected Bitmap doInBackground(Bitmap... bitmaps) {
        ...    
        return sepiaBitmap;
    }
}
```

### Container Performance
So we've talked about how your code could be slow because the type of hardware that's executing underneath it, remember the whole floating point branching issue problem? Well, that's mostly a non-issue for today's hardware. There's one set of issues that you still need to worry about, that is, the performance of the primitives in the language that you are using.

Take a fundamental algorithm such as sorting. Now there are many ways in which to sort, and some are better than others, depending on the circumstances. For examples, quicksort is generally faster than bubble sort except when you have a fewer than a thousand elements or take searching for objects in a large sorted list. Generally, the best way to do this is with a binary search. But completely different is finding objects in an unsorted array. Now instead of in comparing each object for the value you're looking for, you can use a hash function to find it immediately.

Now this is all basic canon of modern computer science and data structures. And thankfully, modern languages like Java supply these containers and algorithms on your behalf, so that you don't have to rewrite the Murmur over and over again. But let me reveal something here. In all of my years of programming, the one problem that consistently bites performance of your project has to do with the performance of these language-provided container objects. I mean, it's awesome, right? Java's providing you with an implementation of a vector class that you can push, pop, add, and remove objects as you see fit, but in order to get that flexibility, it has to use a linked list structure under the hood, which has a unique set of performance characteristics. As long as you are only operating on the front of the list, it's super fast. But if you're trying to insert or remove in other places, it's going to default to the worst possible time. The point is that just because the underlying system provides these containers doesn't mean that they're performant with respect to how your program is going to actually be using them.

James Sutherland published a series of microbenchmarks on the performance of specific data structures provided by the Java framework and found that there's some differences in performance versus functionality that people need to be aware of. For example, he found that Hashtable performance was about 22% faster than HashMap performance, depending on how you're actually using the containers themselves. The point is this. Have you profiled your container classes that you're using in your code? Are you confident that they are using the absolute fastest container for what your code is actually doing? Mm, yeah, that's what I thought. But the good news is that you can gain visibility into the performance of these containers with some handy profiling MPIs in Android. So let's see if Chris's code stands up to our scrutiny. 

### Exercise: Data Structures
In this exercise, you'll see the performance implications of choosing suboptimal data structures in containers when building an app. To do this, we use tools in the Android SDK to identify performance issues related to an inefficient data structure choice. Now as developers, it's easy to overlook the performance impact that comes with writing code to store and manipulate your app's data, so let's take a look at a situation that explores this problem and apply a performance mindset to it.

First, make sure you download the compute and memory sample application linked in the notes below. For this example, we focus on the runtime of a method that generates a list of numbers ranked by their popularity. For demonstration purposes, this method is invoked when we press the following button, dump popular numbers to log. Now similar to previous examples, the code seems to impact the `frame rate` of our dancing pirate via slight stutter. And if you take a look at logcat, you'll see it running via the tag popularity dump, like so. Let's take a look at what is happening under the hood and learn to measure how fast this code is running.

We want granular measurements of exactly what's happening when we click that button, so we use `the trace classes, begin section, and end section methods` to specify exactly where we want the start of measurements and where we want to finish them. First, we zero in on the code that is computing the popularity ranks, which is this method right here. See the instructor notes below for the code stubs we'll use to instrument the code and measure its runtime. After the next two quizzes, we'll run Systrace and retrieve the running time in milliseconds.

```java
public void dumpPopularRandomNumbersByRank() {
    Trace.beginSection("Data Structures");
    // First we need a sorted list of the numbers to iterate through.
    Integer[] sortedNumbers = SampleData.coolestRandomNumbers.clone();
    Arrays.sort(sortedNumbers);
    
    // Great!  Now because we have no rank lookup in the population-sorted array,
    // take the random number in sorted order, and find its index in the array
    // that's sorted by popularity.  The index is the rank, so report that.  Easy and efficient!
    // Except that it's... you know... It's not.
    for (int i = 0; i < sortedNumbers.length; i++) {
        Integer currentNumber = sortedNumbers[i];
        for (int j = 0; j < SampleData.coolestRandomNumbers.length; j++) {
            if (currentNumber.compareTo(SampleData.coolestRandomNumbers[j]) == 0) {
                Log.i("Popularity Dump", currentNumber + ": #" + j);
            }
        }
    }
    Trace.endSection();
    }
```

In this example, by using `dumpPopularRandomNumbersByRank()`, we roughly incur the cost of the sorting operation, plus the O(n^2) cost of iterating through a double loop to generate the list by rank.

### Run systrace on Code
So go ahead and run systrace on this code. Now in this case, how long did the method dump country population ranks take to run in milliseconds? Write your answer in this box here, and remember to use numbers only. 

All right, so here's a screenshot of our output of systrace. Now for us, the unoptimized code took about 23 milliseconds to run. Now, I think we can do better than that. So let's go to the next section to find out how. 

So did you see an improvement? We certainly did. Our optimized code in systrace took about 8 milliseconds. That's almost one-third the original 23 milliseconds. Awesome.

We used a HashMap to optimize our code, See below for our updated code.

```java
public void dumpPopularRandomNumbersByRank() {
    Trace.beginSection("Data structures");
    // Make a copy so that we don't accidentally shatter our data structure.
    Map rankedNumbers = new HashMap<>();
    rankedNumbers.putAll(SampleData.coolestRandomNumbers);
    // Then, we need a sorted version of the numbers to iterate through.
    Integer[] sortedNumbers = {};
    sortedNumbers = rankedNumbers.keySet().toArray(sortedNumbers);
    Arrays.sort(sortedNumbers);
    
    Integer number;
    for (int i = 0; i < sortedNumbers.length; i++) {
        number = sortedNumbers[i];
        Log.i("Popularity Dump", number + ": #" + rankedNumbers.get(number));
    }
    Trace.endSection();
    }
```

Improvement notes:

* There’s always a one-time cost to sort items (you'd pick the ideal sorting item according to the size of your dataset).

* By using a HashMap, we save time by obtaining a linear look-up time (O(n)) vs. the quadratic time O(n^2) access of the array in the unoptimized case. We save this one order of access time because the data is already stored in key value pairs!

* This matters significantly when n (or the sample size) is particularly large, which may be the case if you were working with, say, a list of all professional football players in the world and wanted to display a ranking of them across some attribute.

Let's take a moment to confirm the improvement. Go ahead and add back in (if you removed it) the call to endSection and beginSection. Then, run systrace on the optimized code.

### Dialogue: Outro
*&gt;&gt; You know, I gotta say, Trace View is an awesome tool.  
&gt;&gt; [LAUGH]  
&gt;&gt; I was using it to profile my app last night and I got so much awesome data about my function performance. Who knew that the vector class is slower than the array list?  
&gt;&gt; You know, that's the trick with Trace View, man. I mean it's got a ton of data in there, right? And-  
&gt;&gt; Yeah.  
&gt;&gt; Makes it really hard to use because of it, but once you actually start getting rolling and actually start understanding what it's doing. It's kind of like free awesome in a box, or better yet, like a one stop performance shop. You know, I like that, like [SOUND] we should like, make that into a meme or something like that. Hey, you think if I put that on a t-shirt anybody will buy it?  
&gt;&gt; Yeah, maybe, I mean, yeah, perhaps.  
&gt;&gt; Oh.  
&gt;&gt; But. [LAUGH] I'm going to do this.  
&gt;&gt; Oh no, no, no, no, no Aw.  
&gt;&gt; Yes Come on, man. That's how it's done, man.  
&gt;&gt; Best three out of five.  
&gt;&gt; Heh. If you think it's going to matter.  
&gt;&gt; Eh. Come on.*