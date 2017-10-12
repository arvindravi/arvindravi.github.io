---
title: 'ARKit: A Noobs Guide - Part II'
layout: post
date: 2017-09-27
tags: swift
---
### Part II: Detecting Planes 🛫

### The Basics

In the [previous](../posts/arkit-a-noobs-guide-part-one) part, we went through the basics of ARKit, and some of its most basic classes that help with building AR experiences. In this part, we’ll look at more exciting stuff like detecting flat surfaces in your scene. Around you. Or anywhere really.

Let’s get right into it.

### The Exciting Stuff: Plane Detection

Let’s quickly go over the basics of building an AR experience:
1. Create an **ARWorldTrackingConfiguration** object
2. Create an **ARSession** 
3. Bind the **ARSession** to a **ARSCNView**
4. Run the **ARSession** with the configuration object

ARKit has the ability to detect planar surfaces, not necessarily horizontal plane surfaces, but it is what we are able to detect at this point. Support for vertical surfaces, I believe, will be added in the future. Oh, the [glorious future](http://magicleap.com).

**Code**

To enable plane detection, we simply set a property indicating the type of plane the session will detect, when the configuration object is created:
{% highlight swift %}
let configuration = ARWorldTrackingConfiguration()
configuration.planeDetection = .horizontal
{% endhighlight %}

The `planeDetection` property on `ARWorldTrackingConfiguration` is an enum with one value `ARPlaneDetectionHorizontal`  at the time of writing. 

And by default, plane detection is turned off.

Now you expect to see horizontal surfaces around you in the session. But, you don’t. Let’s see why.

### Show them planes

If plane detection is enabled on the configuration object, and the session is run, ARKit analyses feature points and detects horizontal planar surfaces in the scene, adds `ARPlaneAnchor` objects to the session.

Let’s look into what the logic actually is like. 

An `ARSession` is able to track `ARAnchor`’s position and orientation in 3D space. When plane detection is enabled, `ARPlaneAnchor` (_which is a subclass of ARAnchor_) objects get added to the session whenever a flat surface is detected.

This information is available to the session, but this is not visible to the user unless we make it. The `ARPlaneAnchor` objects contain some useful properties like `alignment`, `center`, and `extent` which can be used to add a visual plane to the scene. 

Every time an ARPlaneAnchor is added, the session notifies your `ARSessionDelegate`, `ARSCNViewDelegate` or `ARSKViewDelegate`. Let’s do this with **SceneKit**, since we’re familiar already with ARSCNView. 

Most often, the View Controller that is handling your `ARSCNView`, would be your `ARSCNViewDelegate`, and you can implement methods on it that will be called by the session when it needs information. 
As for our case, at any point during the session, whenever an `ARAnchor` is added to the session, the session adds an empty `SCNNode` at the anchor position and notifies the delegate with this method:

{% highlight swift %}
func renderer(_ renderer: SCNSceneRenderer, didAdd node: SCNNode, for anchor: ARAnchor)
{% endhighlight %}

We simply check if the anchor added is an `ARPlaneAnchor`, and use the anchor’s properties to add a visual plane onto the scene by adding it to the empty node.

**Code**

There are two parts to showing detected plane surfaces:
- Check if the anchor that was added was a `ARPlaneAnchor` 
- and `addPlane()` if the anchor added is an `ARPlaneAnchor`
	
{% highlight swift %}
func renderer(_ renderer: SCNSceneRenderer, didAdd node: SCNNode, for anchor: ARAnchor) {
	guard let anchor = anchor as? ARPlaneAnchor else { return }
	// Here `anchor` is an ARPlaneAnchor
	addPlane(for: node, at: anchor)
}
{% endhighlight %}

For a detected `ARPlaneAnchor`, we add a plane which is an `SCNNode`:

{% highlight swift %}
func addPlane(for node: SCNNode, at anchor: ARPlaneAnchor) {
  // Create a new node
  let planeNode = SCNNode()

  let w = CGFloat(anchor.extent.x)
  let h = 0.01
  let l = CGFloat(anchor.extent.z)

  // Box Geometry with a minimum height
  let geometry   = SCNBox(width: w, height: h, length: l, chamferRadius: 0.0)   
 
  // Translucent white plane
  geometry.firstMaterial?.diffuse.contents = UIColor.white.withAlphaComponent(0.5)
 
  // Set Position
  // Use the `center` property to find the bounds of the anchor to place the node
  planeNode.position = SCNVector3(
    anchor.center.x,
    anchor.center.y,
    anchor.center.z
  )
 
  // Keep a reference to plane you're adding, so you can update it later
  planes[anchor] = planeNode
 
  // Add PlaneNode to your node
  node.addChildNode(planeNode)
 }
{% endhighlight %}

With that, you should have planes being detected and displayed in the scene around you.

I prefer to abstract the plane logic to its own class. You can take a look at my Plane implementation [here](https://gist.github.com/arvindravi/4a938f38455299c39ec1e482ff3f7f71). 

----
<br>

That’s not all, folks. We still have one thing left --

When planes are detected in the scene, ARKit constantly analyses the feature points for better results, and thereby the same plane that was detected could have a better result, by result I mean the boundaries and the position could be more accurate. To handle this and update the UI accordingly, we need a reference to the planes that we add to our session:

{% highlight swift %}
var planes = [ARPlaneAnchor: SCNNode]()
{% endhighlight %}

I use a dictionary to keep a reference of the planes when they’re added to the session, so I can refer back to the plane that needs updating using its anchor.

When boundaries for existing planes are updated as you get better input from the camera, the following delegate method is called:

{% highlight swift %}
func renderer(_ renderer: SCNSceneRenderer, didUpdate node: SCNNode, for anchor: ARAnchor)
{% endhighlight %}

**Code**

- Updating an existing plane node

{% highlight swift %}
func updatePlane(for anchor: ARPlaneAnchor) {

  // Pull the plane that needs to get updated 
  let plane = self.planes[anchor]

  // Update its geometry
  if let geometry = plane.geometry as? SCNBox {
    geometry.width  = CGFloat(anchor.extent.x)
    geometry.length = CGFloat(anchor.extent.y)
    geometry.height = 0.01
  }

  // Update its position
  plane.position = SCNVector3(
    anchor.center.x,
    anchor.center.y,
    anchor.center.z
  )
}
{% endhighlight %}

So, now that we have a function that updates the plane’s geometry and position for an `ARPlaneAnchor`, it can be called when the delegate is notified of an update:

{% highlight swift %}
func renderer(_ renderer: SCNSceneRenderer, didUpdate node: SCNNode, for anchor: ARAnchor) {
  // Verify the updated anchor is an ARPlaneAnchor
  guard let anchor = anchor as? ARPlaneAnchor else { return }

  //  Update the plane
  updatePlane(for: anchor)
}
{% endhighlight %}

And with that you should have an understanding of how planes are detected and updated in an ARSession. 


### Moving on

You should pretty much be able to write an AR app that detects flat surfaces around you. But what good is it if you can't put stuff in 3D you say? We'll see how to do exactly that in the next part. Feel free to leave any feedback you have.

Index of the series of ARKit posts:
- [Part I: Introducing ARKit](../posts/arkit-a-noobs-guide-part-one)
- Part II: Detecting Planes
- Part III: Adding 3D content to your scene
- Part IV: Lighting with SceneKit

That's all folks.

