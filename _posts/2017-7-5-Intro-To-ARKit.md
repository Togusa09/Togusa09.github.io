---
layout: post
title: Intro to Apple AR-Kit
---

Special thanks to Tod for chasing an iPhone 6 for me, and apologies to him for telling him I needed an iPhone 6 when I needed a 6S.

- Include comment about Readify pd allowance if claim is approved - 

Requirements for developing Apple AR-Kit
* iOS device with an A9 or A10 processor and iOS 11 Beta. This includes the iPhone 6S (emphasis on S), iPhone 7, iPad Pro or iPad (2017).
* Mac OS computer running Mac OS version 10.12.4 or above

https://developer.apple.com/documentation/arkit

Include note on signing profile setup just in case

To begin, I wanted to see how easy it was to get started with running a bare minimum sample AR-Kit app. Fortunately the default template includes a bare bones example that contains a pre-populated model to show off the positional tracking capabilities. 

Document steps for creating new app

After deploying to the iPad, I could run the demo:

<iframe width="560" height="315" src="https://www.youtube.com/embed/NWPTpF8klxY" frameborder="0" allowfullscreen></iframe>

The tracking is surprisingly stable given the much fewer sensors compared to the Hololens. The model does jump around a little when there are obstructions or rapid movement.


Section showing off hit tracking. Not perfect, but quite good considering.
<iframe width="560" height="315" src="https://www.youtube.com/embed/eiJvvcJokhM" frameborder="0" allowfullscreen></iframe>

Add intermediate section on plane detection


Unity section https://unity3d.com/learn/tutorials/topics/mobile-touch/building-your-unity-game-ios-device-testing

Unity build process is similar to the one of the hololens, run a build in unity, which exports to a folder, then open in xcode and run on device.
<iframe width="560" height="315" src="https://www.youtube.com/embed/ntliuPz-tHg" frameborder="0" allowfullscreen></iframe>
<iframe width="560" height="315" src="https://www.youtube.com/embed/Sv7ztYfaogI" frameborder="0" allowfullscreen></iframe>