---
layout: post
title: "Android and BLE"
description: "Tips for working with BLE on Android"
category: 
tags: ["ble", "android", "bluetooth", "java", "app", "mobile", "gatt", "133 error"]
date: "July 9th, 2016"
---
{% include JB/setup %}

Bluetooth Low Energy (BLE) is part of Bluetooth 4.0 and is used in a bunch of random stuff like [socks](https://vimeo.com/70365693), [album covers](https://www.youtube.com/watch?v=YCAzEh6Wyj8), and [baby pacifiers](https://www.bluemaestro.com/smart-thermometer-pacifier/).  BLE in Android has come a long way since it was first supported in Android 4.3, however there are still a few quirks that can make it seem sort of like black magic.  Here’s a few tips that can hopefully make BLE on Android a little less... scary.

## Run BLE Functions on the UI Thread

When I first started working with BLE, one of the first things I noticed was that I was getting null pointer exceptions all over the place.  All the usual remedies did exactly nothing to help and I was teetering on the edge of insanity.  A helpful [post](http://stackoverflow.com/questions/20069507/gatt-callback-fails-to-register) saved my life and solved a lot of problems. The solution: **Run all BLE interactions on the main thread.**  This includes the functions connectGatt, connect, disconnect, close, and discoverServices.  I did this by using a handler:

```java
Handler handler = new Handler(Looper.getMainLooper());

handler.post(new Runnable() 
{
    @Override
    public void run() 
    {
        if (bluetoothDevice != null) 
        {
            bluetoothDevice.connectGatt(this, false, mGattCallback);
        }
    }
});
```

But you could also use a local service or runOnUiThread.
	
## Check the Permissions
		
Have you ever written an app that worked fantastic on Android 5.1 but then failed spectacularly on 6.0? Because I have, and it felt a little like this:

![alt text](http://i.giphy.com/l2QZPHSiJ4do7gg2k.gif "Sad Troy")

For some reason, the BLE scanning app that worked so well on Lollipop didn’t find any devices when run on Marshmallow.  Turns out it was because I didn’t have location services enabled on my device.  To make sure BLE scanning works on 6.0 and above, you’ll need to first, request location permissions:

```xml
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
```
And second make sure that the GPS on the device is actually turned on.

Makes perfect sense, right?  Yeah, I didn’t think so.  Some have speculated that it has something to do with iBeacons or GeoFencing or something.  I’m still scratching my head as to why this fixes the issue.  If any of you have insight, please share. 

## Manage your GATT Objects

One thing you’ll notice when working with BLE is that everything you do is through the GATT server.  GATT stands for General Attribute Profile and is the specification by which we send and receive little pieces of data over Bluetooth.  [Android BLE documentation](https://developer.android.com/guide/topics/connectivity/bluetooth-le.html) states that “Once your app has finished using a BLE device, it should call close() so the system can release resources appropriately.”  This may seem like a bit trivial, but this one sentence is the key to making a BLE app that runs smoothly and reliably.  Devices have a limited number of BLE connections that can be opened at a time, if these connections are not closed properly it will cause issues (trust me I learned this the hard way).  These stalled connections will prevent other connections from being made and in most cases the only way to clear them out is by power cycling the device (which does not make for a great user experience).  This is why you always, always, always, call gatt.close() and gatt.disconnect() (in that order) after you’ve finished your operations.  

One of the most important places to call these functions are when the activity or service that’s handling your bluetooth operations closes (for example onDestroy) so that if your app crashes while it’s connected to a device, it will still properly close out the BLE connection. I cannot stress this enough, calling these functions in all the right places will save you a lot of headaches.
		
## Handle the Dreaded 133 Error

Ok, so even though you’ve followed all of the above tips, there’s one error you’re still probably going to see from time to time.  It’s one of the darkest errors I’ve ever encountered... The 133 error.  This error is thrown in the `onConnectionStateChanged` function of your Bluetooth Gatt Callback (The status will equal 133).  And the spooky thing is that no one knows exactly what it means.  Luckily, there are ways we can handle it. 

The best method I’ve come up with for handling it is to stop BLE scanning as soon as it’s thrown.  Then turn off bluetooth... wait for a few seconds... and turn bluetooth back on and resume scanning.  All of this can be done programmatically but you’re definitely going to want to provide some sort of alert so that the user knows what’s going on as this process takes 5 - 10 seconds.  I’ve seen a lot of 133 errors in my day and this is the most reliable way to recover from it.  On occasion the error will be thrown 2 or 3 times in a row, but cycle the bluetooth stack enough times and you’ll be able to get out of it.

Of course, the best thing to do would be to avoid the 133 error entirely.  Implementing the first three techniques we discussed will dramatically reduce the number of times this error gets thrown.  

So, that’s BLE for you.  I’ve developed and am currently developing several android apps that use BLE, but I still feel like I’m discovering new secrets about it every day.  Do you know something that I don’t?  Please share it with me.  I’d really like that.  