---
title: "ARKit: A Noob's Guide - Part III"
layout: post
date: 2017-10-11
tags: Swift ARKit
---
### Part III: Adding 3D Objects 📦

In parts [one](../posts/arkit-a-noobs-guide-part-one) and [two](../posts/arkit-a-noobs-guide-part-two), we went through how to get started with ARKit, and detect plane surfaces around you. What good is just doing that, you ask?

### Hi, SceneKit.
On iOS, there are two ways to work with 3D:
- SceneKit
- Metal

**SceneKit**, simply put, is a high-level 3D framework for creating 3D objects and working with 3D scenes. It includes a physics engine, and a particle generator to make it as easy as it can be to work with 3D objects and scenes.

**Metal** is a low-level API to the GPU-accelerated hardware on Apple devices. It is designed to be extremely efficient on Apple hardware. Although not the easiest to deal with if you’re looking to render simple 3D without GPU-acceleration.

Let’s deal with SceneKit to add 3D objects to our scene.
### Anchory Sessions
You should be familiar with setting up an AR Session by now. The fundamental concept behind adding 3D objects to an AR Session is adding Anchors to the scene and have the device track the anchors inside of the AR Session.

The AR Session object in your View Controller can be used to add anchors to your session. Like so:

{% highlight swift %}
session.add(anchor: ARAnchor)
{% endhighlight %}

An AR Anchor is the real-world position and orientation that can be used for placing objects in an AR scene. When plane detection is enabled, ARKit adds ARAnchor (more specifically ARPlaneAnchor) objects to the session.

### SceneKit, meet ARKit
There are two ways to add 3D content to your session:
- SCNView’s child node
- ARAnchor

To understand how each of them works let’s make sure we’re through with the fundamental concept behind this first.

Any 3D content modeled with SceneKit can be used with ARKit. ARKit makes this easy with ARAnchors. The idea behind this is — AR Anchor objects that are added to the scene can hold or show 3D content in their position in real-world, which is tracked by your device. That’s it. That’s all there is to it.

Let’s use SceneKit to create a simple cube and see how each of the methods to add 3D content works.

#### Creating a Cube

If you’re new to SceneKit, here’s what you need to know to create 3D content:

1. All 3D content are depicted by nodes: `SCNNode`
2. To create a node, you need to specify a 3D geometry after which the node is modeled

To create a cube, we use SceneKit’s `SCNBox` geometry and model our node with it.

{% highlight swift %}
// create a box geometry
let boxGeometry = SCNBox(width: 0.1, height: 0.1, length: 0.1, chamferRadius: 0) 

// create a node with box geometry
let cube = SCNNode(geometry: boxGeometry)
{% endhighlight %}

It’s as simple as that.

#### Adding 3D geometry to the AR Session
Now that we have our `cube` we’ll look into how to add it to our scene.

Like I mentioned earlier, there are two ways to add 3D content to an AR session.  But before we jump into how we do that, we need a position for the 3D content that you want to place. Let’s get to that.

Since we’re dealing with 3D space, there are two choices that we have to place 3D content:
- On Plane
- Off Plane

With SceneKit, a position of an object in the world is denoted with a `SCNVector3`, this is nothing but a vector object with 3 axes: `x`, `y`, and `z`.

##### On Plane
To place an object on a plane that has been detected, the user generally taps on the screen within the extent of the plane that has been detected. The tap point we have to work with is a `CGPoint`. 

How then do we get a `SCNVector3` with three elements, you ask?

###### Hit-Testing
With a `CGPoint` and an `ARPlaneAnchor` to work with, we can hit-test the point against the plane anchor to get its position in 3D space. This is the general approach towards how objects are placed on a plane. If that sounds complicated, don’t worry, just follow along:


With ARKit, a 2D point can be hit-tested against a plane with ARSCNView’s `hitTest()` method:

{% highlight swift %}
sceneView.hitTest(point: CGPoint, types: ARHitTestResult.ResultType)
{% endhighlight %}

The Result type that will be passed is an enum on `ARHitTestResult`, with the following values:

- estimatedHorizontalPlane
- featurePoint
- existingPlane
- existingPlaneUsingExtent

`estimatedHorizontalPlane` is a real-world planar surface detected by the search (without a corresponding anchor), whose orientation is perpendicular to gravity.

`featurePoint` is a point automatically identified by ARKit as part of a continuous surface, but without a corresponding anchor.

What we are interested in to add objects to a plane, are the plane anchors already added to the session by plane detection. To use this we use:

`existingPlaneUsingExtent` type when hit-testing. `UsingExtent` makes the hit-test respect the plane's limited size unlike simply `existingPlane` type.

Now that we have the hit-test method, we get back a `ARHitTestResult` object that contains the position information. Not exactly. But we can use the information in it to build a `SCNVector3`, which denotes a position in 3D space like:

{% highlight swift %}
// ARHitRestResult
result = sceneView.hitTest(tapPoint, types: .existingPlaneUsingExtent)
 

// Build a SCNVector3 with the result
let position = SCNVector3(
  result.worldTransform.columns.3.x,
  result.worldTransform.columns.3.y,
  result.worldTransform.columns.3.z,
)
{% endhighlight %}

The `worldTransform` property of the `ARHitTestResult` contains information about the object in its 3rd column that we use to transpose into a position vector.

So, no we have a position on the plane we can work with.

##### Off Plane
To add an object off plane, the usual approach is to add it in front of the device’s camera where the user taps. This can easily be accomplished by building a `SCNVector3` using the camera transform.

To obtain the camera transform:

{% highlight swift %}
func getCameraTransform(for sceneView: ARSCNView) -> MDLTransform {
  guard let transform = sceneView.session.currentFrame?.camera.transform else { return }
  return MDLTransform(matrix: transform)
}
{% endhighlight %}

Once you have the `cameraTransform` (MDLTransform), all there’s left to build the `SCNVector3` is:

{% highlight swift %}
let position = SCNVector3(
  cameraTransform.translation.x, 
  cameraTransform.translation.y, 
  cameraTransform.translation.z
 )
{% endhighlight %}

We now have a position in 3D space for your object off-plane.

Let’s go over how to add it to the scene.

#### **ARSCNView**
Now that we have the position of the 3D object as a vector, we can assign that to our SCNNode:

{% highlight swift %}
cube.position = position
{% endhighlight %}

Adding the `cube` to the scene is all there’s left to adding your 3D content:

{% highlight swift %}
sceneView.scene.rootNode.addChildNode(cube)
{% endhighlight %}

This should place a 3D cube in your world and your device should now track it.

#### **AR Anchor**
The other method to add 3D content is adding raw anchors, and showing SCNNode in their position.

The idea behind this method is simple:
1. Create an AR Anchor
2. Add it to the AR Session
3. Implement `ARSCNView`’s delegate method to return a `SCNNode` for an anchor 

To get hold of an anchor to add:
- On Plane
	- The `ARHitTestResult` object from the hit-test method has an `anchor` property which can be added to the session using the `session.add(anchor: ARAnchor>)` method.

- Off Plane
	- Using `ARCamera`’s `transform` property, an anchor can be created like:
{% highlight swift %}
let anchor = ARAnchor(transform: matrix_float4x4) 
{% endhighlight %}

and can be added using the `session.add(anchor: ARAnchor)` method.

Once anchors are added to the session, to display 3D content in their place, we need to implement `ARSCNViewDelegate`’s method:

{% highlight swift %}
func renderer(_ renderer: SCNSceneRenderer, nodeFor anchor: ARAnchor) -> SCNNode?
{% endhighlight %}

This method returns an SCNNode for an anchor in the session, an anchor’s identifier property can be used to distinguish between different anchors added to the session. We simply return a SCNNode in this method, for the required anchor to add 3D content.

### The Code
Let’s look at how all of this comes together in code:

1. Adding 3D content on plane:
{% highlight swift %}
// Inserting 3D Geometry for ARHitTestResult
func insertGeometry(for result: ARHitTestResult) {
 let boxGeometry = SCNBox(width: 0.1, height: 0.1, length: 0.1, chamferRadius: 0)
 let cube = SCNNode(geometry: boxGeometry)
 
 // Method 1: Add Anchor to the scene
 sceneView.session.add(anchor: result.anchor)
 
 // OR
 
 // Method 2: Add SCNNode at position
 let position = SCNVector3(
   result.worldTransform.columns.3.x,
   result.worldTransform.columns.3.y,
   result.worldTransform.columns.3.z,
 )
 cube.position = position
 sceneView.scene.rootNode.addChildNode(cube)
}

// Intercept a touch on screen and hit-test against a plane surface
override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
 guard let touch = touches.first else { return }
 let point = touch.location(in: sceneView)
 
 let result = sceneView.hitTest(point, types: .existingPlaneUsingExtent)
 guard result.count > 0 else {
   print("No plane surfaces found")
   return
 }
 
 insertGeometry(for: result)
}
{% endhighlight %}

Incase you’re adding anchors to the session (Method 1), implementing the delegate method like this is necessary to display 3D content:

{% highlight swift %}
func renderer(_ renderer: SCNSceneRenderer, nodeFor anchor: ARAnchor) -> SCNNode? {
  let boxGeometry = SCNBox(width: 0.1, height: 0.1, length: 0.1, chamferRadius: 0)
  let cube = SCNNode(geometry: boxGeometry)
  return cube
}
{% endhighlight %}

2. Adding 3D content off-plane
{% highlight swift %}
// Get transform using ARCamera
func getCameraTransform(for camera: ARCamer) -> MDLTransform {
  return MDLTransform(transform: camera.transform)
}

// Intercept touch and place object 
override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
  guard let touch = touches.first else { return }
  guard touch.tapCount == 1 else { return }

  guard let camera = sceneView.session.currentFrame?.camera else { return }
  let transform = getCameraTranform(for: camera)
  let boxGeometry = SCNBox(width: 0.1, height: 0.1, length: 0.1, chamferRadius: 0)
  let cube = SCNNode(geometry: boxGeometry)
  let position = SCNVector3(
   transform.translation.x,
   transform.translation.y,
   transform.translation.z,
  )
  cube.position = position
  sceneView.scene.rootNode.addChildNode(cube)
}
{% endhighlight %}

So, that’s about how to add 3D content on and off-plane with ARKit.

### Moving On
In this part we have seen how to add 3D content to your AR session, we'll look into adding some lights and making your objects more interesting in the next part. Thanks for following along, and feel free to leave any feedback!

Index of the series of ARKit posts:
- [Part I: Introducing ARKit](../posts/arkit-a-noobs-guide-part-one)
- [Part II: Detecting Planes](../posts/arkit-a-noobs-guide-part-two)
- Part III: Adding 3D content to your scene
- Part IV: Lighting with SceneKit

