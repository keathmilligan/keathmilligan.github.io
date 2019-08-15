---
layout: post
title: Implement a RESTful interface with Android Things and Raspberry Pi
date: 2017-01-13 22:16:14 -0500
categories: [dev]
tags: [android, android-things, java, rest, raspberry-pi]
featured-img: rpibcm21ledon.jpg
---

<img src="/assets/images/bugdroid.png" align="right">[Android Things](https://developer.android.com/things/index.html) (formerly “Brillo”) is a streamlined version of Android designed for small form-factor and IoT devices. Here is a simple example of a Things app that provides a RESTful web interface to control the state of an LED on a Raspberry Pi GPIO port.
<!--more-->

What you will need:

* A Raspberry Pi 3 Model B
* A breadboard, ribbon cable, 330 ohm resistors, LEDs, jumpers
* An 8GB or larger Micro SD card
* Ethernet cable
* HDMI cable
* Android Studio with API 25 or higher tools installed
* cURL

There are many Raspberry Pi 3 kits available that come with everything you need to get started. Check the [Android Things Raspberry Pi hardware page](https://developer.android.com/things/hardware/raspberrypi.html) for more information on getting the Things preview image loaded onto an SD card and booting up.

You will also need access to “adb” from the command line.

# Setup the Hardware

Unplug your Raspberry Pi and attach the ribbon cable and breadboard. Connect a 330 ohm resistor between BCM 21(see [pinout](https://developer.android.com/things/hardware/raspberrypi-io.html)) and the long leg (anode/+) of an LED. Use a jumper to connect the short leg (cathode/-) of the LED to ground. For example:

![Raspberry Pi LED](/assets/images/rpibcm21led.jpg)

Double-check your connections and power up the device. Connect the Android debugger with:

`adb connect <ipaddress>`

The IP address should be displayed at the bottom of the Android Things boot screen.

# Create the App

The complete source code for this example is available on Github.

Open Android Studio and create a new project.

* You can leave “Phone and Tablet” checked since Android Studio doesn’t yet have an option for an Android Things device.
* Select “API: 24 Android 7.0 (Nougat)” as the minimum SDK.
* Choose an empty activity to start with.
* Uncheck “Backwards Compatibility” on the “Customize Activity” page since we don’t need to worry about running on on older devices.

## Add the Required Libraries

Download and unpack the Android version of the Restlet Framework. Copy the following jars from the lib directory into your application’s app/libs folder:

* org.restlet.jar
* org.restlet.ext.nio.jar
* org.restlet.ext.json.jar

Now, in Android Studio, edit the “Module: app” build.gradle file and add the necessary libraries for Restlet and Android Things to the “dependencies” section as follows:

```
compile files('libs/org.restlet.ext.nio.jar')
compile files('libs/org.restlet.ext.json.jar')
compile files('libs/org.restlet.jar')
```

and:

```
provided 'com.google.android.things:androidthings:0.1-devpreview'
```

## Update AndroidManifest.xml

Open the app/manifests/AndroidManifest.xml file and add the following to the “application” section:

```xml
<uses-library android:name="com.google.android.things" />
```

Modify the main activity as follows:

```xml
<activity android:name=".MainActivity">
  <intent-filter>
    <action android:name="android.intent.action.MAIN" />
    <category android:name="android.intent.category.LAUNCHER" />
    <category android:name="android.intent.category.IOT_LAUNCHER" />
    <category android:name="android.intent.category.DEFAULT" />
  </intent-filter>
</activity>
```

This sets up your application as the default activity for the device.

### Checkpoint: Run the App

At this point, you should be able to launch the app on the Raspberry Pi and see a “Hello, World!” message. Make sure adb is connected (see above) and click the Run icon in Android Studio.

# Add the LED Control Model

Create a new class called LEDModel. This class will provide access to the GPIO pin that controls the state of the LED:

```java
package com.example.restfulthings;
 
import com.google.android.things.pio.Gpio;
import com.google.android.things.pio.PeripheralManagerService;
 
import java.io.IOException;
 
public class LEDModel {
    private static LEDModel instance = null;
    private PeripheralManagerService mPMSvc;
    private Gpio mBCM21;
 
    public static LEDModel getInstance() {
        if (instance == null) {
            instance = new LEDModel();
        }
        return instance;
    }
 
    private LEDModel() {
        mPMSvc = new PeripheralManagerService();
        try {
            mBCM21 = mPMSvc.openGpio("BCM21");
            mBCM21.setDirection(Gpio.DIRECTION_OUT_INITIALLY_LOW);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
 
    public static void setState(boolean state) {
        try {
            getInstance().mBCM21.setValue(state);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
 
    public static boolean getState() {
        boolean value = false;
        try {
            value = getInstance().mBCM21.getValue();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return value;
    }
}
```

For more info on controlling the GPIO outputs of your device, refer to the Android Things SDK documentation.

## Add the Restlet Resource

Create a new class called LEDResource that extends org.restlet.resource.ServerResource. This class provides a RESTful representation of the LED state:

```java
package com.example.restfulthings;
 
import android.util.Log;
import org.json.JSONObject;
import org.restlet.data.MediaType;
import org.restlet.ext.json.JsonRepresentation;
import org.restlet.representation.Representation;
import org.restlet.representation.StringRepresentation;
import org.restlet.resource.Get;
import org.restlet.resource.Post;
import org.restlet.resource.ServerResource;
 
public class LEDResource extends ServerResource {
 
    @Get("json")
    public Representation getState() {
        JSONObject result = new JSONObject();
        try {
            result.put("state", LEDModel.getState());
        } catch (Exception e) {
            e.printStackTrace();
        }
        return new StringRepresentation(result.toString(), MediaType.APPLICATION_ALL_JSON);
    }
 
    @Post("json")
    public Representation postState(Representation entity) {
        JSONObject result = new JSONObject();
        try {
            JsonRepresentation json = new JsonRepresentation(entity);
            result = json.getJsonObject();
            boolean state = (boolean)result.get("state");
            Log.d(this.getClass().getSimpleName(), "new LED state: "+state);
            LEDModel.setState(state);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return new StringRepresentation(result.toString(), MediaType.APPLICATION_ALL_JSON);
    }
}
```

## Add the Service

The HTTP server will be run as a background service on a separate thread from the user interface activity. Add a new class called RESTfulService that extends IntentService:

```java
package com.example.restfulthings;
 
import android.app.IntentService;
import android.content.Intent;
import android.content.Context;
import android.util.Log;
 
import org.restlet.Component;
import org.restlet.data.Protocol;
import org.restlet.engine.Engine;
import org.restlet.ext.nio.HttpServerHelper;
import org.restlet.routing.Router;
 
public class RESTfulService extends IntentService {
    // IntentService can perform, e.g. ACTION_FETCH_NEW_ITEMS
    private static final String ACTION_START = "com.example.restfulthings.action.START";
    private static final String ACTION_STOP = "com.example.restfulthings.action.STOP";
 
    private Component mComponent;
 
    public RESTfulService() {
        super("RESTfulService");
        Engine.getInstance().getRegisteredServers().clear();
        Engine.getInstance().getRegisteredServers().add(new HttpServerHelper(null));
        mComponent = new Component();
        mComponent.getServers().add(Protocol.HTTP, 8080);
        Router router = new Router(mComponent.getContext().createChildContext());
        router.attach("/led", LEDResource.class);
        mComponent.getDefaultHost().attach("/rest", router);
    }
 
    public static void startServer(Context context) {
        Intent intent = new Intent(context, RESTfulService.class);
        intent.setAction(ACTION_START);
        context.startService(intent);
    }
 
    public static void stopServer(Context context) {
        Intent intent = new Intent(context, RESTfulService.class);
        intent.setAction(ACTION_STOP);
        context.startService(intent);
    }
 
 
    @Override
    protected void onHandleIntent(Intent intent) {
        if (intent != null) {
            final String action = intent.getAction();
            if (ACTION_START.equals(action)) {
                handleStart();
            } else if (ACTION_STOP.equals(action)) {
                handleStop();
            }
        }
    }
 
    private void handleStart() {
        try {
            mComponent.start();
        } catch (Exception e) {
            Log.e(getClass().getSimpleName(), e.toString());
        }
    }
 
    private void handleStop() {
        try {
            mComponent.stop();
        } catch (Exception e) {
            Log.e(getClass().getSimpleName(), e.toString());
        }
    }
}
```

This service provides intent actions that the main activity can use to start and stop the server.

## Integrate the Service

Finally, edit the MainActivity class to start and stop the service:

```java
package com.example.restfulthings;
 
import android.app.Activity;
import android.os.Bundle;
 
public class MainActivity extends Activity {
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
 
        // start the server
        RESTfulService.startServer(this);
    }
 
    @Override
    protected void onDestroy() {
        super.onDestroy();
 
        // stop the server
        RESTfulService.stopServer(this);
    }
}
```

# Test

Launch the app. You should see a message in the logcat output that says Starting the internal [HTTP/1.1] server on port 8080.

Open a command-line window and use curl to request the state of the LED:

`curl http://<ipaddress>:8080/rest/led`

Where `<ipaddress>` is the IP address of your device. This should return the current state (false) in JSON format:

`{"state":false}`

To turn the LED on:

`curl -H "Content-type: application/json" -X POST -d '{"state": true}' http://<ipaddress>:8080/rest/led`

![RPI Led On](/assets/images/rpibcm21ledon.jpg)

The LED should now be lit. To turn it off again:

`curl -H "Content-type: application/json" -X POST -d '{"state": false}' http://<ipaddress>:8080/rest/led`
 
### Souce Code

The complete source code for this example is [available on Github](https://github.com/keathmilligan/RESTfulThings).
