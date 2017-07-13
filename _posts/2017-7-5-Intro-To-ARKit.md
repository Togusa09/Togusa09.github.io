---
layout: post
title: Intro to Apple AR-Kit
---

Special thanks to Tod for chasing an iPhone 6 for me, and apologies to him for telling him I needed an iPhone 6 when I needed a 6S.


Requirements for developing Apple AR-Kit
* iOS device with an A9 or A10 processor and the iOS 11 Beta. This includes the iPhone 6S (emphasis on S), iPhone 7, iPad Pro or iPad (2017).
* Mac OS computer running Mac OS version 10.12.4 or above
* XCode 9 Beta

https://developer.apple.com/documentation/arkit

Include note on signing profile setup just in case

To begin, I wanted to see how easy it was to get started with running a bare minimum sample AR-Kit app. Fortunately the default template includes a bare bones example that contains a pre-populated model to show off the positional tracking ability. 

Document steps for creating new app

## Demo 1 - Fixed Model

After deploying to the iPad, I could run the demo:

<iframe width="560" height="315" src="https://www.youtube.com/embed/NWPTpF8klxY" frameborder="0" allowfullscreen></iframe>

The tracking is surprisingly stable given the much fewer sensors compared to the Hololens. The model does jump around a little when there are obstructions or rapid movement.

{% highlight swift %}
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Set the view's delegate
        sceneView.delegate = self
        
        // Show statistics such as fps and timing information
        sceneView.showsStatistics = true
        
        // Create a new scene
        let scene = SCNScene(named: "art.scnassets/ship.scn")!
        
        // Set the scene to the view
        sceneView.scene = scene
    }
{% endhighlight %}

## Demo 2 - Hit detection

AR Kit generates an environment map based on information gathered through its camera. While this environment map does not seem to be directly available like the one supplied by the HoloLens, you can perform a raycast hit test to determine the in world position of corresponding to points on the screen. This demo is an updated version of the previous app, and demonstrates creating an object at a real world position. The user touches the screen, and the app projects the position into the world to determine where it collides with a surface, and creates a cube there.

{% highlight swift %}

override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?){
        if let touch = touches.first{
            let results = sceneView.hitTest(touch.location(in: sceneView), types: [ARHitTestResult.ResultType.featurePoint])
            if let anchor = results.first{
                let hitPointTransform = SCNMatrix4(anchor.worldTransform)
                let hitPointPosition = SCNVector3Make(hitPointTransform.m41, hitPointTransform.m42, hitPointTransform.m43)
                let cubeNode = SCNNode(geometry: SCNBox(width: 0.1, height: 0.1, length: 0.1, chamferRadius: 0.01))
                cubeNode.position = hitPointPosition
                sceneView.scene.rootNode.addChildNode(cubeNode)
            }
        }
    }

{% endhighlight %}

The intersection calculation isn't perfect, but once again it is surprisingly good for just a single optical camera. As this is still only a beta, I expect this to improve

<iframe width="560" height="315" src="https://www.youtube.com/embed/eiJvvcJokhM" frameborder="0" allowfullscreen></iframe>
 
## Demo 3 - Plane Detection

As with the HoloLens, AR Kit provides a mechanism for detecting flat surfaces, and providing them to the program as a plane. This feature will be useful for many apps, and can provide a flat, bounded environment to work with. Currently AR Kit does not have wall plane detection, so can only detect floors and desks. If wall detection is required, a raycast hit test can be be used. As more of the environment is scanned, AR Kit builds a more complete model and recalculates the plane, performing a callback when it does so. As it is unable to detect walls, it may combine areas, even if they are separated by a physical obstacle, such as walls. The following demo displays and updates the planes as they are updated.

{% highlight swift %}
    var redMaterial : SCNMaterial!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Set the view's delegate
        sceneView.delegate = self
        
        // Show statistics such as fps and timing information
        sceneView.showsStatistics = true
        sceneView.debugOptions = [ARSCNDebugOptions.showFeaturePoints, ARSCNDebugOptions.showWorldOrigin]
        
        redMaterial = SCNMaterial()
        redMaterial.diffuse.contents = UIColor.red
        
        redMaterial.diffuse.contents = UIColor(red: 0.67, green: 0.0, blue: 0.0, alpha: 0.5)
    }
    
    func renderer(_ renderer: SCNSceneRenderer, didAdd node: SCNNode, for anchor: ARAnchor)
    {
        if let planeAnchor = anchor as? ARPlaneAnchor{
            print("anchor")
            print(planeAnchor.extent)
            print(planeAnchor.center)
            print(planeAnchor.alignment)
            
            let box = SCNBox(width: CGFloat(planeAnchor.extent.x), height: CGFloat(planeAnchor.extent.y), length: CGFloat(planeAnchor.extent.z), chamferRadius: 0.01)
            box.materials = [redMaterial]
            
            node.geometry = box;
        }
    }
    
    func renderer(_ renderer: SCNSceneRenderer, didUpdate node: SCNNode, for anchor: ARAnchor){
        if let planeAnchor = anchor as? ARPlaneAnchor{
            if let box = node.geometry as? SCNBox{
                box.width = CGFloat(planeAnchor.extent.x);
                box.height = CGFloat(planeAnchor.extent.y);
                box.length = CGFloat(planeAnchor.extent.z);
            }
        }
    }
{% endhighlight %}

<iframe width="560" height="315" src="https://www.youtube.com/embed/eK8pSn-hedI" frameborder="0" allowfullscreen></iframe>

At the moment AR Kit is not as advanced as the technology we see in the HoloLens, but so far I haven't seen anything take full advantage of that technology, so I believe both are fairly evenly matched for the moment. AR Kit experiences are far less immersive as they are on a hand held device, but the cheapest device it can run on is six times cheaper than a HoloLens, and iPhones are quite common, making for a much lower barrier to entry. I think that AR Kit will really help Mixed Reality applications take off, especially if Apple release an equivalent of the Samsung Gear or Google Dream so that the user can be immersed.

## Unity Demo

Unity have created a plugin that provides a wrapper around the AR Kit functions so that they can be used in the unity engine. The store plugin provides some example scenes to demonstrate this capability.

 https://unity3d.com/learn/tutorials/topics/mobile-touch/building-your-unity-game-ios-device-testing

The Unity build process is similar to the process used for building HoloLens apps. The build in the unity editor creates an XCode project, that you open in XCode, then use that to build the iOS app.

<iframe width="560" height="315" src="https://www.youtube.com/embed/ntliuPz-tHg" frameborder="0" allowfullscreen></iframe>
<iframe width="560" height="315" src="https://www.youtube.com/embed/Sv7ztYfaogI" frameborder="0" allowfullscreen></iframe>