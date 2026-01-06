---
layout: post
title: Update on AR-Kit
---

Now that AR-Kit and iOS are out of Beta, Apple have unveiled some additional new features exclusive to the new iPhone X, as well as guidelines on creating VR Experiences.

The iPhone X has the addition of Apples "TrueDepth" camera, combining colour and infrared cameras with a structured light projector to gather a detailed map of the object in front of it. The data captured from these sensors is used to provide facial recognition, as well as "Face-Based AR Experiences". Until I am able to get hold of an iPhone x, or another device is released with an equivalent camera, for the moment I will provide a summary of the features it exposes.

# New features

## ARFaceAnchor
ARFaceAnchor is a new type of AR anchor that is centered on the users head, providing position, orientation, geometry and expression information. This is similar to the face recognition functionality provided in the vision framework, so I'm not sure why these features are limited to the front facing camera. The dedicated hardware probably provides a greater amount of speed and accuracy, but a less accurate version that could provide anchors when it recognises a person would be a useful function.
[Docs](https://developer.apple.com/documentation/arkit/arfaceanchor)

## ARFaceGeometry
ARFaceGeometry provides a detailed model of the users face. This allows AR entities to be created that track the position and changing facial expressions of the user. This is similar to the model required for applications like snapchat, but much higher quality. Due to the information required, it is understandable that it is only available on the front facing camera, as there would be difficulty creating and tracking that level of detail for a large scene. 
[Docs](https://developer.apple.com/documentation/arkit/arfacegeometry)

## Directional Lighting
ARDirectionalLightingEstimate makes use of the additional sensors and proximity its subject to be able to create a lighting model far more detailed than for ambient scenes. This provides a estimation of the intensity and direction of the the primary light source, allowing a light source to be created for accurately lighting the scene. 
[Docs](https://developer.apple.com/documentation/arkit/ardirectionallightestimate)

## ARTrackable
The ARTrackable interface provides an extension point for including real world objects into the AR Scene. At the moment it is only implemented by ARFaceAnchor, but in future I foresee it being used to help bridge with objects tracked by the vision library, or IoT devices.
[Docs](https://developer.apple.com/documentation/arkit/artrackable)

# Interface Guidelines

Apple have released a set of interface guidelines for creating AR applications. I've summaries them below

 - Project a "Dashed" square onto the surface to show plane detection in progress
 - Project "Solid" square onto the surface showing that a plane is found
 - Provide an option to reset the AR experience visible a all times
 - Provide instructions to the user to scan more of the environment if a plane is required but not yet found
 - Allow objects to be placed in world, even if no plane is detected, then move onto plane if one is found
 - Respond to standard gestures performed close to the object, except zoom, as scaling may be confused with moving the object
 - Gestures and direct interaction with AR objects is preferable to a normal UI
 - Object rotations should be perpendicular to the plane that the object sits on, rather than based on the camera perspective

[Human Interface Guidelines for Augmented Reality](https://developer.apple.com/ios/human-interface-guidelines/technologies/augmented-reality/)


[Handling 3D interaction and UI controls in augmented reality](https://developer.apple.com/documentation/arkit/handling_3d_interaction_and_ui_controls_in_augmented_reality)


[Creating face based AR experience](https://developer.apple.com/documentation/arkit/creating_face_based_ar_experiences)