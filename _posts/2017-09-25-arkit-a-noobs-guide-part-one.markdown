---
title: "ARKit: A Noob's Guide - Part I"
layout: post
date: 2017-09-24
tags: Swift ARKit
---
## Part I: Introducing ARKit 🔮
ARKit is the new kid on the block. It’s the shiny new Apple’s framework on the iOS Platform to develop Augmented Reality apps and games. Its pretty cool actually. I will try to not bore you to death and tell you things you need to  know. 
To get started.
Like right away.

### What can it do?
ARKit is a library that’s going to change how people interacted with their phones, I say this because the iPhone is a popular device, and iOS is a popular platform. 
Come on, admit it.

With ARKit, you can:

- Detect plane surfaces
- Place 3D virtual objects around you
- Interact with it, _using your device_
- Track its position in real time

### What do I need?
To be able to use ARKit, or write code for it you need an  device with A9 processor or better. Like A10. Or A11.

See if you have one of these, if you do, you’re good to go:
- iPhone 6s and 6s Plus
- iPhone 7 and 7 Plus
- iPhone SE
- iPad Pro (_both 1st gen and 2nd gen_)
- iPad (2017)
- iPhone 8 and 8 Plus
- iPhone X

### ARKit: The Basics
Let’s get right into it. There are a few ARKit-ty stuff (_more like classes, but oh well_) you need to understand before you’re ready to take ARKit for a spin:

- ARWorldTrackingConfiguration
- ARSession
- ARSCNView
- ARAnchor
- ARPlaneAnchor

##### ARWorldTrackingConfiguration

The **ARConfiguration** class  deals with establishing a relationship between the real world where the device is, and the virtual co-ordinate space where content can be modelled. The **ARWorldTrackingConfiguration** class is a subclass of ARConfiguration, and this is responsible for tracking the device’s movement with six degrees of freedom (_6DOF_). 
The six degrees of freedom represent the freedom of movement of an object in three-dimensional space, like so:

**Rotation Axes**:
- Roll
- Pitch
- Yaw

**Translation Axes**:
- Movement in x
- Movement in y
- Movement in z

You can count, there are six.

Let’s now get to initialising an ARWorldTrackingConfiguration object, that we can create an ARSession with.

> Pro-tip: When initialising a **ARWorldTrackingConfiguration**, you can optionally set a boolean property called `planeDetection` to `true` to enable plane detection in your session.

**Code**
- Instantiating a session configuration

{% highlight swift %}
let configuration = ARWorldTrackingConfiguration()
{% endhighlight %}

##### ARSession
Every AR experience built with ARKit, relies on a ARSession object to manage the session. This object co-ordinates with your device’s camera, the motion sensors, and performs image analysis on the capture frames.

The building block of an AR experience, is the **ARSession**, and it’s the session is run with a {session configuration} object.

There are two parts to running an ARSession:
1. Instantiate an ARSession object
2. Call `run()` on the ARSession object passing a session configuration object

**Code**
 - Creating an ARSession

{% highlight swift %}
let session = ARSession()
{% endhighlight %}

- Running the session

{% highlight swift %}
session.run(configuration)
{% endhighlight %}

##### ARSCNView
If you’re already familiar with SceneKit, the **ARSCNView** is a subclass of **SCNView** that comes with all what you need to setup an AR experience using SceneKit.

ARSCNView is used to display a camera and any 3D content that you will be adding to your scene. Pretty much, the view port of your AR experience.

How can you use it? It’s not very hard. You simply create an AR session, and bind that to the `session` property of your ARSCNview instance.

**Code**
- Creating an ARSCNView instance

{% highlight swift %}
let sceneView = ARSCNView(frame: UIScreen.main.bounds)
{% endhighlight %}

- Binding a session to it

{% highlight swift %}
let session = ARSession() 
sceneView.session = session
{% endhighlight %}

##### ARAnchor
Once you know the fundamentals of how to run an ARSession, let’s see what you can do with a running ARSession. You can add stuff to it. I mean, anything. Even 3D 💩, and Craig would even [appreciate] it.

There’s a concept of anchors in a session, it’s not rocket science — you basically add anchor objects to a AR session that’s running, and your device tracks its position and orientation of whatever you add to the session. Let’s go over how to create an ARAnchor object and add it to a session.

However, to create an AR Anchor, you need a matrix that encodes the position, orientation, and scale of the anchor relative to the world co-ordinate space of the session the anchor is in. That sounds tricky? A little. But all you need to understand is, that this matrix is of the type**matrix\_float4x4** in Swift. 

This value can be obtained based on where you want to place your anchor, one of the ways is to grab the **ARCamera**’s transform property and bind that to the anchor, to place an anchor in front of the camera.

**Code**
- Getting a transform (_let’s grab the camera transform_)

{% highlight swift %}
let transform = sceneView.session.currentFrame.camera.transform
{% endhighlight %}

_Here, you grab the camera from the currentFrame (an ARFrame) and use its transform property_

- Creating an ARAnchor

{% highlight swift %}
let anchor = ARAnchor(transform: transform)
{% endhighlight %}

- Adding an anchor to the session

{% highlight swift %}
sceneView.session.add(anchor: anchor)
{% endhighlight %}

Yes, it’s as simple as that.
	
##### ARPlaneAnchor
The **ARPlaneAnchor** is a subclass of an **ARAnchor**, with one cool fact — 

_ARPlaneAnchor objects get added to the session automatically when plane detection is enabled._

We’ll go into detail about how ARPlaneAnchor objects are added to the session, and how they can be used when we’re discussing about detecting planes in the next part of the guide.

An ARPlaneAnchor has some useful properties:
- `alignment`
- `center`
- `extent`

These properties help place 3D planes to the scene when plane surfaces are detected. Yes, really.

----
<br>
### Moving on
I’ve tried to put together the fundamentals of **ARKit** in this part, and we’ll be going over more ARKit-y stuff in the coming parts. In the next part we’ll be discussing how to detect plane surfaces.

Index of the whole series of ARKit posts:
(I’ll update the links here as they’re ready, but for now feel free to leave any feedback. 🤓)
- Part I: Introducing ARKit
- Part II: Detecting Planes
- Part III: Adding 3D content to your scene
- Part IV: Lighting with SceneKit

That's all folks.
