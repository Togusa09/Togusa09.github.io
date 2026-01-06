---
layout: post
title: Creating dynamically coloured models in Unity using Blender
categories: unity blender C#
---

Colour can be used in games to convey useful contextual information about objects in the world. For example, if you have storage tanks that can each be used to store one of a variety of different resources, colour can be used to provide a clear visual indication of which resource each tank contains. 
This post will cover creating a simple model to use as the tank, importing it to unity, then adding configuration to the object to allow the runtime colour to be changed.

After creating the new blender model, check that "Meters" is set as the unit.  
![BlenderUnits]({{ "/images/UnityMaterialColor/Units.png" | absolute_url }})  

>Quick summary of some useful Blender controls  
>A - Select All  
>L - Select connected  
>Shift + Right Click - Add to selection  
>Alt + Right Click - Select Loop (Edges or faces)  

## Creating the main tank section (dynamic colour)

- Create UV Sphere
![BlenderSphere1]({{ "/images/UnityMaterialColor/Sphere1.png" | absolute_url }})  
- Switch to edit mode (tab)
- Switch to edge select, and select the perimiter edges (alt + right click)  
![BlenderSphere2]({{ "/images/UnityMaterialColor/Sphere2.png" | absolute_url }}) 
- Split into seperate hemispheres by pressing space, and choosing "Edge Split"
- Select the top hemispwhere by clicking on part of it, then pressing "L"
- Translate the selected hemisphere directly upwards  
![BlenderSphere3]({{ "/images/UnityMaterialColor/Sphere3.png" | absolute_url }}) 
- Select the edge loops from both hemispheres using alt + right click on the first edge, then shift + alt + right click on the second
- Use "Bridge Edge Loops" to fill the gap between the two hemispheres  
![BlenderSphere4]({{ "/images/UnityMaterialColor/Sphere4.png" | absolute_url }})  
- Select the whole model by pressing "A"
- Select the texture tab on the top right, then create a new texture by clicking the "+", then "Create"
- Rename the material "Dynamic". This will be the material getting replaced, so its defined colour will only be seen in the editor.  
![BlenderSphere5]({{ "/images/UnityMaterialColor/Sphere5.png" | absolute_url }})  

## Creating the outer frame (Fixed colour)

- Create a cylinder, then resize it to be a narrow band that has a diamater slighly larger than the main tank. (Pressing "N" brings up a transform menu that is useful for this)  
![BlenderFrame1]({{ "/images/UnityMaterialColor/Frame1.png" | absolute_url }})  
- Select the whole cylinder (Click one face, then press "L") 
- Press "Space" and choose "Duplicate". The duplicate will be created in a the same location, so you will have to translate it to the new position.  
![BlenderFrame2]({{ "/images/UnityMaterialColor/Frame2.png" | absolute_url }}) 
- Add a cube, then resize to make a post that reaches from the top cylinder to the ground  
![BlenderFrame3]({{ "/images/UnityMaterialColor/Frame3.png" | absolute_url }})  
- Duplicate the post three times, and move the around the tank  
![BlenderFrame4]({{ "/images/UnityMaterialColor/Frame4.png" | absolute_url }})  
- Select the frame sections, then create a new Material name "Frame". I suggest setting this material to a grey to make it look like metal.  
![BlenderFrame5]({{ "/images/UnityMaterialColor/Frame5.png" | absolute_url }}) 

Final Model. A copy of the model can be downloaded from [here]({{ "/assets/SampleTank.blend" | absolute_url }})
![TankFinal]({{ "/images/UnityMaterialColor/TankFinal.png" | absolute_url }})  


## Setting dynamic colour in Unity
- Create a new Unity project
- Copy the Blender model file into the Assets folder. Unity is able to import models straight from a blender file.
- Create a new scene, then place the tank model into the scene
- Create a new script called "DynamicTank"
- Open the script using the editor of you choice, and add the following:

{% highlight c# %}
public enum TankResourceType
{
    Water,
    Oxygen,
    Hydrogen
}

public class DynamicTank : MonoBehaviour {
    public TankResourceType Resource;


    // Use this for initialization
    void Start()
    {
        var renderer = GetComponent<Renderer>();
        // Get the material to dynamically colour
        var material = renderer.sharedMaterials.First(m => m.name == "Dynamic");
        // Create new material instance, so that setting the material colour doesn't affect everything else using the material
        renderer.material = new Material(material);

        // An alternative option which will return a new instance of the first material on the model. This may be a simpler convention.
        //material = renderer.material;

        // Set the material colour based on the resource type
        switch (Resource)
        {
            case TankResourceType.Water:
                material.color = Color.blue;
                break;
            case TankResourceType.Oxygen:
                material.color = Color.red;
                break;
            case TankResourceType.Hydrogen:
                material.color = Color.white;
                break;
        }
    }
}
{% endhighlight %}

In editor
![Unity1]({{ "/images/UnityMaterialColor/Unity1.png" | absolute_url }})  
In Play mode
![Unity2]({{ "/images/UnityMaterialColor/Unity2.png" | absolute_url }})  