---
layout: post
title: Intro to Apple AR-Kit
---

Special thanks to Tod for chasing an iPhone 6 for me, and apologies to him for telling him I needed an iPhone 6 when I needed a 6S.



Requirements for developing Apple AR-Kit
* iOS device with an A9 or A10 processor and iOS 11 Beta. This includes the iPhone 6S (emphasis on S), iPhone 7, iPad Pro or iPad (2017).
* Mac OS computer running Mac OS version 10.12.4 or above

https://developer.apple.com/documentation/arkit

Include note on signing profile setup just in case

To begin, I wanted to see how easy it was to get started with running a bare minimum sample AR-Kit app. Fortunately the default template includes a bare bones example that contains a pre-populated model to show off the positional tracking capabilities. 

Document steps for creating new app

## Demo 1 - Fixed Model

After deploying to the iPad, I could run the demo:

<iframe width="560" height="315" src="https://www.youtube.com/embed/NWPTpF8klxY" frameborder="0" allowfullscreen></iframe>

The tracking is surprisingly stable given the much fewer sensors compared to the Hololens. The model does jump around a little when there are obstructions or rapid movement.


Section showing off hit tracking. Not perfect, but quite good considering.

## Demo 2 - Hit detection

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

<iframe width="560" height="315" src="https://www.youtube.com/embed/eiJvvcJokhM" frameborder="0" allowfullscreen></iframe>
 

## Demo 3 - Plane Detection

Add intermediate section on plane detection

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

## Unity Demo

Unity have created a plugin that provides a wrapper around the AR Kit functions so that they can be used in the unity engine. The store plugin provides some example scenes to demonstrate this capability.

 https://unity3d.com/learn/tutorials/topics/mobile-touch/building-your-unity-game-ios-device-testing

The Unity build process is similar to the process used for building HoloLens apps. The build in the unity editor creates an XCode project, that you open in XCode, then use that to build the iOS app.

<iframe width="560" height="315" src="https://www.youtube.com/embed/ntliuPz-tHg" frameborder="0" allowfullscreen></iframe>
<iframe width="560" height="315" src="https://www.youtube.com/embed/Sv7ztYfaogI" frameborder="0" allowfullscreen></iframe>