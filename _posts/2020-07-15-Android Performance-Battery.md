---
layout:     post
title:      "Android Performance: Battery"
subtitle:   Android系统性能 (5)
date:       2020-07-15
author:     Cliu
header-img: img/post-daily-bg.jpg
catalog: true
tags:
    - Android
---

> Based on resources of the udacity course [Android Performance](https://cn.udacity.com/course/android-performance--ud825) developed by Google. Code samples can be downloaded from [Github](https://github.com/udacity?q=ud825&type=&language=).

### Dialogue: Intro
*&gt;&gt; Wow, what a day.  
&gt;&gt; Hey Chris?  
&gt;&gt; What's going on? Oh, hey, what's up?  
&gt;&gt; I got tell you, man, I'm really impressed with this application you wrote, that actually shows where all your coworkers are in real time. This is pretty cool, like it even shows me as I'm moving it around.  
&gt;&gt; Well, cool, thanks man, yeah, it was fun to make, the key is to keep an open connection when the phone screen is off, so that we can update your location in real time.  
&gt;&gt; Whoa, no, tell me you're not doing that.  
&gt;&gt; Well, you know, there were some problems stopping the GPS when folks entered the bathroom, but I think that's a small.  
&gt;&gt; Dude, Chris.  
&gt;&gt; To fix.  
&gt;&gt; My battery just died.  
&gt;&gt; Oh, dang that sucks maybe it's a dud.  
&gt;&gt; Dud, Chris this is a brand new phone, your application ate through my entire battery in eight minutes, are you even taking into account the battery churn? Those open GPS and networking connections are taking up?  
&gt;&gt; I thought so, I ran it through a bunch of tools.  
&gt;&gt; Oh wait, timeout. What, what tools are you using to measure battery, Chris?  
&gt;&gt; You know. The usual ones like Logcat.  
&gt;&gt; [LAUGH] Okay. Stop what you're doing, we're going to go fix this now, because this is an abomination, all right, come on, let's get going.  
&gt;&gt; 'Kay.  
&gt;&gt; Oh, bring your backpack, you're going to need it.*

### Understanding Battery Drain
As your mobile device is busy executing tasks, calculating how to split a bar tab and uploading photos of your cat, the underlying hardware is effectively pulling energy from your battery to accomplish this work. And, as we've all seen, the more work your device does, the more battery it pulls and sooner rather than later, your users are left holding onto an uncharged phone that doubles as an expensive doorstop.

The key to writing applications that are light on battery drain has everything to do with understanding how the process works under the hood. In the electrical engineering world, this action of a hardware pulling energy from the battery to execute tasks is called `current draw over time` and anyone who has an undergraduate in electrical engineering will tell you not every action on your device draws the same amount of battery in the same way, over the same period of time.

To prove this, let's take a handy Nexus let it sit on our desk. If we leave the phone like this, doing nothing, we'll roughly get a month of life before the battery is completely exhausted. Feel free to try this one at home, kids. We can consider this the `baseline in terms of battery life` but that's not really how these devices are typically used. As soon as you're active, you're going to be eating up more battery. Now active in this context includes things like the CPU doing work, cellular radio transmitting data, and the screen itself being awake.

So the question is this, what tasks are my application doing that's eating up the most battery? Sadly though, that's not easy to answer. Monitoring battery drain at the hardware level is a catch-22 because of course the monitoring itself needs to drain battery to execute the actions to record how much battery is being drained and as such, most mobile devices don't do this. The only real way to gather these types of `statistics on battery draw` is to attach a third party piece of hardware to your Android device, which can record the operations without using up the phone's power to do so.

Now when we do this, we actually get some really interesting information. For example, when a Nexus 5 is in standby mode, there's actually not much power drain going on, right? But when we wake the device up or turn on the screen, we can see a huge power spike in the battery monitor, which I mean, you kind of expect, right? Turning on LEDs, painting the screen and then all that CPU, GPU work required to draw to the screen isn't free from a battery perspective, which by the way is completely different from when the application wakes up the device, say if it's using `wake clock`, `alarm manager` or the `job scheduler` API.

When the device is asleep and it's woken up through one of these APIs, you'll see an initial battery spike as the processor first wakes up, followed by a bit of work as it's executed. Now, it's important to note that once the work is done, the device will go back to sleep on its own, which, is really important. Keeping the device awake for long periods when doing little or no work will easily chew up your battery life.

Now your cellular radio, on the other hand, is a completely different beast in this regard. When your device tries to send data over the mobile network, you can see that there is a quick wake up cost associated with getting your mobile hardware ready, followed by a large spike for sending out a data packet and then another large spike for receiving one back. And because getting the radio started is so expensive, after it's done executing work, it will stay on in a wait state for a short period of time in case there are more packets ready to come in right away.

So all this is great data but most developers don't have access to this kind of equipment, right? I mean but with the `L release of Android`, you've got a whole new set of tools to help you optimize the battery draw of your application. Let's take a deeper look at `Battery Historian`. 

### Battery Historian
Hey, glad you're back. Let's talk about another useful tool that'll help you gather data and get better insight into how your applications using energy. It's called Battery Historian. Basically, you're going to use ADB to dump data from your phone. And then use the Battery Historian tool to convert that data into a nice HTML table that you can view in your web browser.

Now Battery Historian is a separate open source python script that you're going to need to download from [GitHub](https://github.com/google/battery-historian). So let's go ahead and do that. Okay. So here we are at the GitHub page for Google's version of Battery Historian. Now in order to download this file, we want to come down here to lower right and click this download zip button. For this demo we're going to work from the desktop. But you can use any directory of your choosing.

Now to get started let's go ahead and double-click this zip file which is going to extract the folder with the python script that we care about. Okay. Let's go ahead and open the folder. As you can see there's a license file, as well as the historian python script. Let's go ahead and drag this script to the desktop. Okay, that's it for setup. We should be all set to use Battery Historian.

Now the next thing you're going to want to do, is make sure you connect your phone to your computer. Also make sure that USB debugging is turned on. I'm going to go ahead and run ADB devices just to confirm. All right, the next steps will work from any terminal window. But if you have Android Studio opened like I do, you can actually open a terminal right within the IDE, like so.

Go ahead and cd to your desktop. Now as a precaution, we're going to go ahead and shut down our ADB server. The way you do this is by typing `adb kill-server`. Now this is an important step because while we're developing, lots of stuff can be turned on that might cause conflicts, while trying to do battery recordings. So to be safe here, we want to start ADB clean.

All right. So I'm going to go ahead and type `adb devices`. As we can see, ADB is restarting, and our phone is attached. If you don't see any devices, make sure your phone is connected and your USB debugging is turned on. Then go ahead and kill and restart ADB once more. Now the next thing we want to do is reset the battery data gathering so you can start from a blank state, we do it like this.

> adb shell dumpsys batterystats --reset

So I just type this command adb shell dumpsys batterystats, With the reset option. This is going to go ahead and reset all the battery gathering data. Now if you don't do this, you're going to get a lot more data than is actually useful or relevant. In other words, you're going to see a lot more noise than you want when you're reading your battery stats.

All right. So now you can go ahead and disconnect your phone from your computer. When you go ahead and pull this cable out right here, so you don't draw any current from your computer instead of the battery. 

### Battery Historian Part 2
All Right so we're back in Android studio. Let's make sure that our phone is reconnected. So we're going to go ahead and type ADB devices once more. Just to confirm that it's back here in our list. All right so now it's time for the real fun. Let's go ahead and look at some battery statistics. And the way we do that is we run this command.

> adb shell dumpsys batterystats > batterystats.txt

You type in ADB shell. Dump Sis, which is stands for dump service battery stats. Let's go ahead and pipe that to a file. Now the next thing we're going to do is run the same command again, however we're going to restrict the data that it outputs. By passing in the package name of the application we're profiling. Let's do it for sunshine.

> adb shell dumpsys batterystats > com.example.android.sunshine.app > sunshine.txt

All right, so with these commands we've partially generated some data that we're. going to use. Unfortunately the data in these files aren't very readable. So we're going to use the historian Python script to extract the battery of information and turn that into some HTML that we can then view in the browser.

> python historian.py sunshine.txt > sunshine_battery_stats.html

So this is how we do it. All right, so let's go ahead and invoke that Python script called historian.py that we downloaded earlier. That's the battery historian script. So I'm going to type in python historian.py. Let's look at the sunshine file. And let's generate a sunshine battery stats HTML file. Okay let's go ahead and open this file in a browser.

All right, here we are looking at the battery story and output now as an HTML file in our web browser. Now there's a good amount of information in this file. Let's go ahead and see what some of it is. And what it might mean. And at the top left, you can see the file name and some overall information such as the `battery level`. In this case, we're at 67%. This can give you a quantitative idea of battery drain over time. As you see, this point here is actually when our battery went down. By 1% to 66.

All right, so if we scroll down a little bit, towards the bottom here, there's a `timeline` and a scale. And then, the time in this table runs from left to right, and everything that is vertically aligned happened at that same time. Now you can see the scale here but you can also adjust. That zoom level right here. Now, when you see these little bars here that represent activity, you can actually mouse over and you can actually see when this particular item started and how long it's been active.

Now, what battery storage does not tell you is how much battery was used for each individual action, only how often and how long it used battery. But you can look at the battery level over time to get a general sense of the absolute battery drain. Now the rows are pretty obvious from their names so we're not going to explain them right now, you'll get that more deeper during the tools exercise. Now you see these bars, and you can actually mouse over each one of them. To find out when it started and how long it took.

Check out this item here, this one called `screen`. Now, this represents how often the screen is on. In this case, when I was interacting with the phone. You could see that screen was on the entire time. We also have this section for `top` which actually lists which application was in the forefront at the time. In this case, we've interacted with Sunshine for a little bit and then if we scroll to the right here, this is when we hopped into Maps. And this is actually when we switched to YouTube.

Now what's new about this file is we see a lot of things additionally that might impact the battery. For example, we have stats of when `WiFi` was one, in this case all the time. `Video` playbacks seems to be happening. You'll. Notice the `GPS` started here. And if we scroll up, this is the time that we actually hopped into the maps application. So that would make sense that when our maps application was started, our GPS was turned on. And this time the GPS was using a bit of battery here. We also see bursts of spikes here in our WiFi strength. And that's changing over time as well, dynamically.

Now here's a more in depth example output from a battery store and capture. You can see that there's a very large amount of `mobile radio` and `wake lock` work that's occurring on this device. It looks like a wake lock is turning on running some networking work and then going back to sleep over and over again. This level of behavior definitely deserves a closer look to see if it's using too much battery. In the next section, we'll use battery storing to find out battery problems in our code. Let's go ahead and do it.

### Track Battery Status Using Battery Manager
Here is a simple example of some logic using `BatteryManager`:

```java
/**
 * This method checks for power by comparing the current battery state against all possible
 * plugged in states. In this case, a device may be considered plugged in either by USB, AC, or
 * wireless charge. (Wireless charge was introduced in API Level 17.)
 */
private boolean checkForPower() {
    // It is very easy to subscribe to changes to the battery state, but you can get the current
    // state by simply passing null in as your receiver.  Nifty, isn't that?
    IntentFilter filter = new IntentFilter(Intent.ACTION_BATTERY_CHANGED);
    Intent batteryStatus = this.registerReceiver(null, filter);

    // There are currently three ways a device can be plugged in. We should check them all.
    int chargePlug = batteryStatus.getIntExtra(BatteryManager.EXTRA_PLUGGED, -1);
    boolean usbCharge = (chargePlug == BatteryManager.BATTERY_PLUGGED_USB);
    boolean acCharge = (chargePlug == BatteryManager.BATTERY_PLUGGED_AC);
    boolean wirelessCharge = false;
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN_MR1) {
        wirelessCharge = (chargePlug == BatteryManager.BATTERY_PLUGGED_WIRELESS);
    }
    return (usbCharge || acCharge || wirelessCharge);
}
```

In this example, we start by setting up an intent filter to catch the battery status. Then we register this filter, while passing null which gives us the current state. From this state, we can grab an extra from the intent that stores the current charging state. We save this in chargePlug.

BatteryManager has a handful of constants representing the state of the battery. In this case, we’re checking whether chargePlug is equal to the constant for being docking with the `USB, AC or Wireless charge`. We store this in a boolean, then we can use it for app logic.

### Wait for Docking
For battery intensive tasks, you should determine whether they can wait until you’re connected to power. For example, taking a picture isn't something that you can delay, especially if you're trying to capture that special moment, while filtering a picture can be deferred.

Now let's hop back into Android Studio and take a look at the solution for how to defer a task. All right, first, let's look at our applyFilter method. See the instructor notes for the code snippet, specifically. Now here, we call `checkForPower` and if it returns that the phone is not charging, we then change the text to inform the user to plug in their phone and return before completing the method.

Simple enough. All right, let's take a look at the logic within checkForPower. Again, see the instructor notes for the code snippet. Okay, so here we are within checkForPower. Now the first thing we do is we set an intent filter to describe for changes to the battery state. And then we actually get an integer, right here, representing the battery plugged in status.

We then compare this integer to various constants within the BatteryManager class. For instance, one for whether it's charging via AC or alternating current. This means it's plugged into the wall. Another for USB. And finally, whether it's charging via wireless. We wrap the wireless check in an if statement to make sure that the SDK build version is high enough. And then finally, if it was charging via USB, AC, or wireless, we return true. Otherwise, false. 

### Wakelock and Battery Drain
It's no secret that in order for your app to be successful, a user has to run it. And for it to continue to provide value, sometimes it needs to keep running. However, this can cause problems when the user or the system puts the phone to sleep. While it might seem like a right idea to wake up the phone to do your work, there's a massive battery problem waiting to happen here.

To illustrate this point, let's talk about my good friend Raido whom you all may remember from the Android fundamentals course. He's an avid social user and a game player, and his running applications are constantly checking scoreboards, uploading photos, and pulling down the latest hashtag crazes. But they do this even when Raido puts his phone to sleep, which means by lunch, his phone is already out of battery.

Raido has to constantly recharge his phone because it suffers from `sleep deprivation`. That is because these applications are constantly waking up the phone, they're eating up battery life instead of letting the phone go to sleep. Remember that Android will turn off various parts of your hardware in order to prolong battery life. First, the screen will dim followed by it eventually turning off. And finally, the CPU will go to sleep. All to save precious, precious battery life. 
But even in this inactive state, most apps will try to do work, and they will have to wake up the phone to do so.

Now the easiest way for your app to wake up your phone to do work is with the `powermanager.wakelock` API. Which is used to keep the CPU running and prevent the screen from dimming or turning off. This allows your app to do things like wake up the phone at some future date and do some work, and then let the phone go back to sleep. But see it's that last part that's actually the tricky one.

`Acquiring wake locks is really easy but knowing when and how to release them is more problematic.` There's lots of ways that this can go horribly wrong. Like er what if your application takes seconds to analyze an image content before uploading it to the web. Or what if the server crashes and you never get a response from your social ping and end up waiting forever. Well the result is the phone would stay on, waiting for the content and thus draining the battery in no time. Which is exactly why you should be `using the version of wakelock.acquire that takes a time out parameter`. This will force the wake lock to be released in these cases of crazy so that your app doesn't keep the phone awake until the battery is dead.

But even this doesn't solve the problem completely, what should you set the timeout value to be in these cases and how long should you wait after a failure before using a wake lock to try again? The right solution to this might be to use `inexact timers` which allow you to schedule work for some future date. But if the system detects that it could occur sooner or later in order to conserve battery life then it will wait for that time to do that work. For example, if another process has woken up the phone, your app might wait to be woken up with that group of processes rather than happening ten seconds sooner like it was originally scheduled.

And in reality, there's a whole group of situations that might be better to do battery intensive work. For example, `when the phone is charging, or connected to wifi or already woken up by another process`. If any of your work can be deferred to a future date when these conditions might be ideal, then you can help improve battery life significantly. And this is exactly what the `job scheduler API` is for.

See this API is perfect for scheduling some future work to occur in a battery efficient manner. For example, any non-user facing work waiting until the unit is plugged in waiting until you're on wifi, or `batching` a bunch of tasks to occur together at the same time. Basically, with one simple API, you can get all that scheduling for free. So, let's take a look at this API in action. 

### Job Scheduler
Hello again. So Colt just talked about Raido's phone suffering from sleep deprivation. Now this refers to the sad situation where Raido's favorite apps would continually drain his device's battery by preventing it from going to sleep.

Now here's an example of what this kind of hyper-active app behavior looks like in battery storing and output. This is a screenshot from a Google IO that shows the activity of 50 simulated apps. Now I want you to notice a few things.

* Notice the frequent wake lock activity. Lots of gaps usually mean lots of wake up cycles for the device.

* Also notice the bursty network activity. As we know, all these things contribute to significant battery drain.

By the way, I strongly recommend that you watch The Talk. It's linked in the instructor notes and it's a good 30 minutes well-spent.

All right. Let's look at some code that might contribute to a graph looking like this. If you'd like to follow along, go ahead and check out the 3.21 wake locks branch in the sample app. Traditionally, if you want to keep the device awake we would implement a Wake Lock. Now just an FYI, when you build the code and run it, you'll notice a new button in your sample app called Release the Wake Lock. Now this launches a new activity with a button to trigger some code. This code models what it might be like for an app to poll a server for an update of some kind. Perhaps retrieving some new news data on some periodic basis.

So let's look deeper into the code. All right so we're going to start looking at the free the wake clock activity source. Now as we can see, we have a set onClickListener that basically is going to call this function called pollServer when we press that button on our app. Let's look into that code. Now this method writes some code to mimic what it's like to poll a server for some type of uptake.

Now this method is going to use a wake lock. Now, in code the first thing that's done is you require a wake lock to keep your device awake. So that you can then proceed to do the network update. We then check to see if the network is connected. Now in real life if it's not connected the app might just retry until it is. If it is connected, we use an async task to retrieve new data from the server away from the main UI thread.

Now, for demonstration purposes. Within our simple download task, we basically open up an HTTP connection to google.com and get a simple get request. And, when our background thread finishes executing, we then go ahead and we release the wake lock. The inherent problem with wake locks is that they're manually held and released. So if you hold onto the wake locks for too long, you could be draining the user's battery needlessly. `Maybe it's okay if one app is doing this, but if multiple apps are using wake locks inefficiently, that's a recipe for sleep deprivation`.

So the moral of the story is, make sure to release your wake lock as soon as you are finished doing your work. In the next video, we're going to see how we can still achieve periodic background updates. But implement them in a way that will help conserve more of your devices battery power. 

### Network and Battery Drain
Let's take a moment to make something insanely clear. `As far as battery is concerned, networking is the biggest, baddest, dirtiest offender` that there is. Remember that inside of your phone is a small piece of hardware that's effectively a HAM radio, its whole purpose is to communicate with local cell phone towers and transmit data to them at high volumes. But the trick is that this chip, isn't always active, you see, once you send a packet of data the `radio chip` will stay on for a certain amount of time, just in case there's a response from the server that it's expecting. But if there's no activity, the hardware will shut down to save battery life.

And as we've seen before, there's a large battery spike when this chip first turns on, and as long as it's keeping itself awake to wait for responses, it's going to keep on draining the battery at the same time. Now it's worth pointing out that there's two primary ways in which most apps interact with the radio.

* Firstly, there are events that need to occur right now. These events are the result of some user action, or they arise from the immediate need to update the UI of your app. For example, imagine when the user asks to load a new batch of tweets for a trending hashtag, since this is a user initiated action, your app should respond ASAP.

* On the other side of this coin are all the networking jobs that don't need to happen in a time critical manner for example, uploading user data, synchronizing background statistics, or re-sizing all of your social photos.

So while the first set of tasks happen, has to happen immediately, in order to provide feedback to the user, the second set of tasks can easily be put off until later, when they can be performed in a battery efficient manner. And there's a high probability, that the majority of your network requests in your application fall into this second category. Converting networking jobs over to being more efficient is a two step process.

* Firstly, take a hard look at the mobile radio row in your battery historian tool for your application. Each of those red bars that you see here represent an active mobile radio, any gaps between those bars represent when the radio is asleep. If you see lots of narrow bars and gaps in your graph, this can point to your performance problems, since it means that you're churning through lots of wake up and sleep cycles. 

* What you want instead, is to see large gaps next to large blocks of activity. This way you've reduced overhead by minimizing the number of network requests, and even better, don't use the radio at all. I mean you can wait until the phone is connected to WiFi and then let the `WiFi hardware` do all this work with much less battery train.

Now, the problem is that writing the code to `batch, cache, and defer` all these networking requests is really difficult to get right, which is why we've done the work for you. The `JobScheduler API that rolled out with the L release of Android` provides a full suite of API's that do all of this network request management work and more, on your behalf. But rather than telling you about this wonderful API, why don't you take a swing at it in practice? 

### Using Job Scheduler
As Colt just talked about, there are certain tasks you can delegate to the system because the system will have a better idea of when to schedule those tasks in order to save battery. Now as a developer, the key is to identify which tasks in your app should be delegated to Android's job scheduler using the job schedule API. For example, this is `non user facing work tasks that can wait for the device to be plugged in or connected to WiFi or just general situations where a bunch of tasks can be batched to run at the same time`. All of these are good candidates for the job scheduler.

So let's see how we can modify our sample app code to use it. To follow along, check out the branch 3.22_wakelocks_optimized. Now when you check out this optimized branch, you'll notice that your activity's been updated. You have a new button within your freeway clock activity called pull smarter, be smarter bees and it's going to go ahead and execute our improved code.

So, let's go ahead and see what this code actually does. All right, so here we are back in Android studio and I want to call out a few new additions to our project here. Namely, a new file called my job service. We'll quickly learn what we're going to do with this file. Let's go ahead and focus back on the main source for this wake lock activity.

Now in order to use the job scheduler, you're going to need to create a service endpoint. This is what the system will call when it's appropriate to run your job. And this is exactly where our job service source comes in. All right, so here we're extending the framework provided `JobService` class and then all we need to do is implement the `onStartJob` and `onStopJob` callbacks. So let's look at them.

Now, in our onStartJob method, we can implement the logic to retrieve our new updates from the network in the background. As you can see, this is similar lines to what you've seen before. We're calling our simple download task and executing it in the background. Now, you should already be familiar with this code from before. You check to see if the device is connected to a network. And then if so, we go ahead and execute our simple download task in the background.

And we don't do anything major in the on stop job callback in this mock situation. But this method is invoked if your job must be stopped prematurely, i.e., before your code calls job finished. Now, this might happen if the requirements for your job aren't met. For example, if your device loses connection to WiFi. Now, if you are shipping in an app, make sure to add code here that would be able to prevent your app from misbehaving due to your job being halted.

And when you're finished with your work,. It's important that you call the job finished method. This lets the system know when you're done with your work so it can release the wake lock that it's been holding on your behalf. Now since the majority of our work occurs in the simple download task, it would make logical sense to come down here and call job finished during our onPostExecute method.

So now that we know what will happen when our job is actually called upon by the system to run. But how do I actually do submission to the job scheduler? Let's take a look. So I want to go over here and launch our FreeTheWakelockActivity again. So let's go ahead and modify what this pollServer function does. Again this method pollServer is actually called when we press this button Poll Smarter, Be Smarter. As you can see, it's scheduling jobs.

Okay, so here we are within the pollServer method. Now the first thing we do is we get a reference to the job scheduler itself using a call to get system service. The next thing we do is the important part. We're going to go ahead and create what the framework calls a job info instance. And this is a basically an encapsulation for the requirements for a particular task you'd like to defer to the system to do on your behalf.

```java
public void pollServer() {
    JobScheduler scheduler = (JobScheduler) 
		getSystemService(Context.JOB_SCHEDULER_SERVICE);
    for (int i=0; i<10; i++) {
        JobInfo jobInfo = new JobInfo.Builder(i, mServiceComponent)
                .setMinimumLatency(5000) // 5 seconds
                .setOverrideDeadline(60000) // 60 seconds
		// WiFi or data connections
                .setRequiredNetworkType(JobInfo.NETWORK_TYPE_ANY)
		// For WiFi only
		//.setRequiredNetworkType(JobInfo.NETWORK_TYPE_UNMETERED)
                .build();

        mWakeLockMsg.append("Scheduling job " + i + "!\n");
        scheduler.schedule(jobInfo);
    }
}
```

Now the way we generate a job info object is we use this builder class with the job info. And it's got a set of handy helper methods within it to basically help specify the constraints for our particular job. In this case, we call `setMinimumLatency`, which means that the system should wait at least five seconds before running this job. We then set an override `deadline` of 60 seconds. That means if this job hasn't been executed within a minute, go ahead and execute it. The next thing we do is we set a required `network type` to be any. So, we allow this job to run on WiFi and any data connections. Finally, we finish it by calling .build which generates our job info.

Now, once we've encapsulated our entire job constraints within the job info object, all we need to do is call scheduler.schedule and pass in the job info instance that we just created. Pretty cool, huh? 

### Job Scheduler Quiz
Now it's worth noting by using the JobScheduler API, you're being a good Android citizen. And the more good Android citizens there are, the greater the collective impact you'll have on battery life. Now what we've shown you is just a small sample of what job scheduler supports. There are way more options and configurations that you an tailor for your own jobs.

Now for more details on how to use this API, see the [Android documentation](https://developer.android.com/reference/android/app/job/JobScheduler) in the instructor notes. Also be sure to check out the [Google Devbyte](https://youtu.be/QdINLG5QrJc) on how to use the job scheduler. You'll find the link in the instruction notes as well. All right, now it's time for a little quiz. What [Jobinfo.Builder](https://developer.android.com/reference/android/app/job/JobInfo.Builder) method would you append to your job to make sure it runs when the device is plugged in? Go ahead and enter your answer right here.


Okay, so the answer we're looking for here is `setRequiresCharging` with true passed in as a parameter. So hopefully you were able to find your way to the JobInfo.Builder documentation. And then fortunately when you reviewed the public methods, you'd be pleased to find this function called setRequiresCharging. And it specifically addresses what we want it to do. Which is to specify that to run this job, the device needs to be plugged in. 

### Reduce Network Access to Save Battery
Now, let's apply the JobScheduler API to minimize the unnecessary wakeup cost of the cellular radio because we know that frequent reoccurrences of this comes at a significant cost to the battery life. Instead, we're going to use the JobScheduler to avoid waking up the cellular radio and schedule our network activity only when the user's device is connected to wi-fi.

All right, so let's go ahead and look at some code. Now, in order to follow along, go ahead and check out the 3.31_wait_for_wifi branch in the repo for your sample app. Now just as a reminder when you build this branch in code, you'll notice another button just like similar exercises added to your sample app's UI. This one's called no wi-fi, no deal. When you click on it, it'll launch a new activity called find the wi-fi activity.

In this case when you press the button download all the data signal or no, we're going to be triggering some code that we're going to be looking at. It looks like it's trying to make some connections, and it looks like it's fetching some HTML. All right, here we are back in Android Studio, and I've gone ahead and opened up the source code for FindTheWifiActivity. Now similar to our previous exercises, we have a button's OnClickListener basically invoking a function that's doing a particular task. In this case we call the function downloadSomething.

Now, this code should look familiar. It's not too different than the previous exercise when we're trying to basically make a network connection and do a simple download from a server. Again, here in our downloadSomething method, we're basically mocking the behavior that an app would take to essentially check to see if the network is alive, and then execute a task in the background to download some data.

Now if you remember from Colt, we'll know that `cellular network access is one of the biggest offenders for reduced battery performance`. So in order to conserve battery life, we're going to try to reduce the amount of times we start up the cellular radio, thus incurring less of a tax when having to start up the cellular radio incurring this long network tail of energy.

All right, so let's go ahead and hop back into our sample app code and let's see if we can repurpose it and focus on executing our network tasks only when wi-fi is available. Now what this really means is we're going to want to repurpose I would downloadSomething method. This time around, we're going to want to go ahead and schedule a job for the job scheduler. So let's take a look. Now instead of trying to download something in a manual fashion, let's go ahead and see how we can craft a job for the job scheduler to check for a wi-fi only situation.

So what I have up here is a comparison view of basically the wait_for_wifi branch, and then our You can go ahead and check this out in your sample app code repo if you want to see the difference, but I'm going to go ahead and describe the change. Now in our optimized code, we're going to look at a method called downloadSmarter. Now this is hooked up to our on click listener for our button in our UI. But mainly this downloadSmarter method is going to go ahead and use the job scheduler.

Now similar to previous examples, the first thing we do is we get a reference of the scheduler via the getSystemService method. Now in the area where we normally submit jobs, we're going to go ahead and bundle up a job info. Again, that's the encapsulation of what we want to do for our particular job. Essentially it's the constraints for running it. In this case, we've changed one particular line from our previous example. In this case we used internal method .setRequiredNetworkType to specify a network that's unmetered, essentially, wi-fi only.

And then we go ahead and we call .build and that's going to go ahead and generate a JobInfo instance for us. And then last but not least, we go ahead and we pass that to the schedule function of the Scheduler. Again the last app that we actually have to do is called the scheduler, with the schedule method passing in our new job info instance. And when it returns, our job service will be called and the task will be run. And when the system deems that it's time to run our job it's going to go ahead and call back into our MyJobService class, and it's going to call our onStartJob method that we implemented before.

In this case we're going to go ahead and check for a network connection, in this case it should be wi-fi, and then execute our SimpleDownloadTask. Then lastly when we're finished doing our network activity, which again is implemented in our SimpleDownload class, particularly in the doInBackground method, we go ahead and we open up an http connection and do a get request. Then, lastly, when onPostExecute is called on our Async task, we go ahead and call `jobFinished`, which is critical in telling the system that we are done with our work, and it can go ahead and release the wake lot that is held on our behalf.

All right, so this was an example on basically how to repurpose some of our sample app code to use job scheduler, this time to `minimize cellular radio wakeup`. Now as mentioned before, the framework offers a lot more possible combinations for specifying a particular job and its associated requirements. Now I suggest that you go ahead and review the developer documentation for all the detail information and see the link in the instructor notes. 

### Final Quiz
Now, throughout this class, we've been giving you sample code to work on. Now I've got a little interesting opportunity for you. Now it's your turn to apply what you've learned to your own apps. So your final mission is to profile your app using `battery historian`.

* In number one, go ahead and identify opportunities to use the JobScheduler. For example, look for inefficient wake lock and network access that you can reduce.

* Number two, go ahead and generate a report and battery string and take a screenshot. Make sure to keep your raw HTML file as well.

* Third, go ahead and make improvements to areas in your code that you've identified as potentially inefficient or problematic.

* Number four, go ahead and re-profile your code, then go ahead and generate a new report in battery storing and take a new screenshot.

And then when you're done, go ahead and share the before and after screenshots on the forum, and then call out any specific improvements that you see in your graph. Or if you don't see any real clear difference in the before and after screenshots, post us to why this might be the case. We'll be checking the forums periodically and we want to see what you guys come up with. All right, cool, so this wraps up our course and I want to thank you for all your hard work, it's certainly been a pleasure to come along and assist in your journey. I hope you learned to always apply performance mind set, when writing your code and maybe hope you had a little fun too. Take care and we'll see you soon.

### Dialogue: Outro
*&gt;&gt; All right, I'll take that as an action item and let's wrap this up.  
&gt;&gt; It's really cool that your fixed the battery issues on this phone. I finally can find all my friends.  
&gt;&gt; Oh yeah, I mean you know with the right tools and the right process it turns out turning your code from battery issues is really easy.  
&gt;&gt; There, there's still one bug left. You know this right?  
&gt;&gt; Oh really?  
&gt;&gt; It says that code has been stuck in there for 16 hours.  
&gt;&gt; Oh you caught that. Yeah. The truth is that's not really a bug.  
&gt;&gt; No?  
&gt;&gt; Yesterday we overheard these guys talking about managed memory environments. Like, they come for free in terms of performance. He completely flipped out so we had to lock him in the elevator. [SOUND].  
&gt;&gt; Now this place ain't so bad. Put some curtains in. Maybe a window over there. That would. Be bad. Some air. [NOISE] I just want to make apps faster. Is that so bad? [NOISE] We can do this. We can do this. I'll get you, Chris. Mark my words.*

