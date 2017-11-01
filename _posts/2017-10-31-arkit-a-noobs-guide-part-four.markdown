---
title: "ARKit: A Noob's Guide - Part IV"
layout: post
date: 2017-10-31
tags: swift
---
### Part IV: Lighting with SceneKit💡

In the last few parts of the series, we’ve discussed how to setup an AR Session, detecting planes, and adding 3D objects to your session. To make your 3D content all better, we’ll work with lighting in this section.

#### **Fill — How?**
The 3D content, which are nothing but `SCNNode` objects that we add to the session, have an array property — `materials` that hold an array of `SCNMaterial` objects. To be able to fill an object with a solid color, all there is to do is:

* Create a `SCNMaterial` object
* Assign a color to its `diffuse` property
* Add the created `SCNMaterial` object to the `materials` array of the `SCNNode`

And that’s all there is to it.

**Code**:

{% highlight swift %}
// Cube Node
let boxGeometry = SCNBox(width: 0.1, height: 0.1, length: 0.1, chamferRadius: 0)
let cube = SCNNode(geometry: boxGeometry)

// Material Object
let material = SCNMaterial()

// Assign a color to the material
material.diffuse.contents = UIColor.blue

// Add material to the cube node
cube.materials = [material]
{% endhighlight %}

This piece of code should accomplish adding a fill to your node. Let’s look into how to add a light node to the scene.

#### **Lighting with SceneKit**
The primary premise around Augmenting Reality is that you add virtual objects to the real one, and the objects we add need to look and feel like they’re part of the actual space you’re interacting with. Lighting your objects right help you with this.

Lighting adds a layer of realism to your 3D objects. With SceneKit, this is accomplished by adding `SCNLight` objects to your scene.

There are different types of SCNLight, that can be used based on the kind of lighting you want in your scene:

- Omni
- Ambient
- Directional
- Spot
- IES
- Probe

Using one or more lights in combination in your scene will give you all different kinds of interesting lighting. 

To light a 3D object in the scene, very simply, we need two types lights in the scene:
- Spot (or) Directional Light
- Ambient

We add an Ambient light source to our scene because in real life, there are multiple light sources and light bounces of every surface lighting all sides of an object. We simulate this by adding an Ambient Light along with a Spot light or a Directional Light to light the object up.

**Code**

<ol>
1. Creating a Spot Light
{% highlight swift %}
let spotLight = SCNLight()
spotLight.type = .spot
spotLight.spotInnerAngle = 45
spotLight.spotOuterAngle = 45
spotLight.intensity = 40
{% endhighlight %}

2. Creating an Ambient Light
{% highlight swift %}
let ambientLight = SCNLight()
ambientLight.type = .ambient
ambientLight.intensity = 40
{% endhighlight %}

3. Adding them to your scene
{% highlight swift %}
sceneView.scene.rootNode.addChildNode(spotLight)
sceneView.scene.rootNode.addChildNode(ambientLight)
{% endhighlight %}
</ol>

`SCNLight` has a property called `intensity` that signifies the luminous flux in lumens, or the total brightness of the light. 

`SCNView` has a boolean property called [`automaticallyUpdatesLighting`](https://developer.apple.com/documentation/arkit/arscnview/2887446-automaticallyupdateslighting), which when `true` adds an Omni lightsource to the scene pointing in the direction of the camera and sets its intensity to `1000` by default.

We can manipulate realism, or how realistic an object is by actively controlling the intensity property of our lights in the scene. To be able to do this, let’s turn the default lighting off:

`sceneView.autoenablesDefaultLighting = false`

The `ARConfiguration` instance that we use to setup an `ARSession` has a boolean property called [`lightEstimationEnabled`](https://developer.apple.com/documentation/arkit/arconfiguration/2923546-islightestimationenabled), which when true gives us a light estimate value of every `ARFrame` that’s captured. This is very useful because we can leverage the [`lightEstimate`](https://developer.apple.com/documentation/arkit/arframe/2878306-lightestimate) property to update our lights with the right intensity based on the environment.

**Code**
<ol>
1. Enabling  Light Estimation in the scene
{% highlight swift %}
// AR Session Configuration: configuration
configuration.lightEstimationEnabled = true
{% endhighlight %}

2. Using light estimates to update lighting
{% highlight swift %}
func renderer(_ renderer: SCNSceneRenderer, updateAtTime time: TimeInterval) {
  guard let lightEstimate = sceneView.session.currentFrame?.lightEstimate else { return } 
  lightEstimate.ambientIntensity gives us an estimate based on the real world lighting
  spotLight.intensity = lightEstimate.ambientIntensity/1000
}
{% endhighlight %}
</ol>

Using Light Estimation this way helps with realism to a certain extent, and works most times if the user is engaged within the experience in a well-lit environment.

#### **Advanced Lighting with SceneKit: PBR**

We have seen how to use basic lighting to light up your 3D objects, and make it look realistic. We can now go one step further with a technique called **Physically Based Rendering** to texture our objects and make it convincing.

![](https://i.imgur.com/1ObQtqv.png)

These are the different [lighting models](https://developer.apple.com/documentation/scenekit/scnmaterial.lightingmodel) that we could use to light our objects up. We’ll look at how to use PBR to make a realistic looking objects.

Simply put, [PBR lighting](https://developer.apple.com/documentation/scenekit/scnmaterial.lightingmodel/1640553-physicallybased) leverages the following material properties to compute lighting and texture on the object:

* `diffuse`:
    The diffuse property provides the base color of the material. Also known as _albedo_ in some authoring tools.
* `roughness`:
    This provides an approximation of the microscopic detail in a real-world surface.
* `metalness`:
    This provides an approximation of how shiny the material is going to be.

More specifics on the lighting terms are described excellently [here](https://developer.apple.com/documentation/scenekit/scnmaterial.lightingmodel/1640553-physicallybased).

The process to apply materials to SCNNode is similar to how we applied a fill in our object earlier, except for the PBR lighting we will be providing the relevant properties we discussed above.

We provide texture maps to the properties `diffuse`, `roughness` and `metalness`  as an image. These texture maps can be found online, and some free ones at [FreePBR](http://freepbr.com).

**Code**
<ol>
1. Creating a material
{% highlight swift %}
let material = SCNMaterial()
{% endhighlight %}

2. Assigning Lighting Model and Texture maps
{% highlight swift %}
material.lightingModel = .physicallyBased
material.diffuse.contents = UIImage(named: "albedo.png")
material.roughness.contents = UIImage(named: "roughness.png")
material.metalness.contents = UIImage(named: "metalness.png")
{% endhighlight %}

3. Adding them to the object
{% highlight swift %}
cube.materials = [material]
{% endhighlight %}

<p>The final step is to tell your scene to use an environment map to figure out how to light the object based on its environment.</p>

4. Using an environment map
{% highlight swift %}
let env = UIImage(named: "environmentMap.png")
 
// Set Environment Map
sceneView.scene.lightingEnvironment.contents = env
{% endhighlight %}

<p>
This process is similar to how we set a fill earlier and should be straightforward. The objects should look way better with this kind of lighting.

Now to use the light estimates like earlier with PBR, we just set the `intensity` of the `lightingEnvironment` in the following method:

{% highlight swift %}
func renderer(_ renderer: SCNSceneRenderer, updateAtTime time: TimeInterval)
{% endhighlight %}

The `lightingEnvironment` takes value of `1.0` for neutral and ARKit returns a value of `1000` to represent neutral lighting. So scaling this value is essential before assigning it to the lighting environment.
</p>

5. Using light estimates
{% highlight swift %}
let intensity = lightEstimate.ambientIntensity / 1000.0 

// Set intensity of the lighting environment
sceneView.scene.lightingEnvironment.intensity = intensity
{% endhighlight %}
</ol>

This should already provide you convincing results with lighting your objects.

Note: After having used this, I’ve obtained some good results although I came across this note on the documentation which says SceneKit falls back to the .blinn lighting model if the `renderingAPI` is not metal. Feel free to try it out and see what works best for you.

![](https://i.imgur.com/3mNMX7J.png)

#### **Moving On**

With that, the series on ARKit comes to an end.

Sorry for the delay in finishing this series, I’ve been busy with an [AR app](http://itunes.apple.com/us/app/artx/id1296941506?ls=1&mt=8) that I’ve been working on. Do check it out!

Thanks for following along, I hope the series helped understand how ARKit works and how to get things going at your end. Feel free to leave any comments and feedback!

Index of the series of ARKit posts:
- [Part I: Introducing ARKit](../posts/arkit-a-noobs-guide-part-one)
- [Part II: Detecting Planes](../posts/arkit-a-noobs-guide-part-two)
- [Part III: Adding 3D content to your scene](../posts/arkit-a-noobs-guide-part-three)
- [Part IV: Lighting with SceneKit](../posts/arkit-a-noobs-guide-part-four)

---

