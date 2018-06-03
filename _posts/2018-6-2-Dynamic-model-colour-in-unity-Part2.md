---
layout: post
title: Creating dynamically coloured models in Unity - Part 2
categories: unity blender C#
---

Som additional methods for setting object colour

## Setting colour from shader

The default shader used by Unity includes a parameter _Color that is used to override the rednering colour for an object. Unfortunately it still creates additional texture instances. It should be possible to work around this by using [GPU instancing](https://docs.unity3d.com/540/Documentation/Manual/GPUInstancing.html) for the rendering. 

Setting colour by 

{% highlight c# %}
public class ShaderColourRenderer : MonoBehaviour
{
    public Color Color;

    private Renderer _renderer;

    void Awake()
    {
        _renderer = GetComponent<Renderer>();
    }

    void Update () {
        // Uses the first materials
	    var mat = _renderer.material;
	    mat.SetColor("_Color", Color);
    }
}
{% endhighlight %}

[MaterialPropertyBlock](https://docs.unity3d.com/ScriptReference/MaterialPropertyBlock.html) provides an alternate way of setting the shader properties. This works fine for objects with single textures, as the property is set for the shaders on all materials, it will override the properties of other meterials. This could be resolved by customising the shader so that the property name isn't shared between the material shaders. However, if customising the shader, the colour replacement could be done by doing by substituting a colour value in the objects texture, or by using a second texture that defines the area for the colour replacement. 

## MaterialPropertyBlock example:

{% highlight c# %}
public class SetMaterialPropertyBlock : MonoBehaviour
{
    public Color Color;

    private Renderer _renderer;
    private MaterialPropertyBlock _propBlock;

    void Awake()
    {
        _propBlock = new MaterialPropertyBlock();
        _renderer = GetComponent<Renderer>();
    }

    void Update()
    {
        // Get the current value of the material properties in the renderer.
         _renderer.GetPropertyBlock(_propBlock);
        // Assign our new value.
        _propBlock.SetColor("_Color", Color);
        // Apply the edited values to the renderer.
        _renderer.SetPropertyBlock(_propBlock);
    }
}
{% endhighlight %}

## Using a material manager class

Of course, if the only need is to prevent the creation of a new texture for every object and colour change, then the simplest method is to focus on the on preventing the additional textures being created for every colour change and object. Since this is a relatively simple use case, the coloured materials could be maintained through use of a manager class:

{% highlight c# %}
public static class MaterialManager
{
    static Dictionary<Color, Material> Materials = new Dictionary<Color, Material>();

    public static Material GetMaterial(Color colour, Material objectMaterial)
    {
        if (Materials.ContainsKey(colour))
            return Materials[colour];
        var material = new Material(objectMaterial);
        Materials[colour] = material;
        return material;
    }
}
{% endhighlight %}