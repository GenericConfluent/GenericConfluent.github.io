+++
date = '2025-08-06T10:12:09-06:00'
title = 'Blender Portal'
description = "Quest for a decent portal setup in blender. No stylization, just infinite recursion and mesh teleporation."
tags = ['blender', 'math', 'effects']
draft = false
+++

I somehow decided to implement a portal in blender, and a proper explanation of
the theory surrounding the implementation details seemed quite lacking&mdash;and
I like to have a concrete handle on what I'm doing when I work on something. So
here we are. The goal of this post is not quite to create a radical new and
better portal impl, but moreso to pick among the pieces people have already
developed and assemble them into a good and reusable impl.

Here is the feature list I want from the implementation:
1. Portal transforms can be modified without breaking the setup.
2. Transport geometry between portals so there is no need to duplicate objects travelling through.
3. When we see a portal through the portal, there will be a portal&trade;

Here are the restrictions:
- A Portal is an N-Gon which lies in the XY plane when ignoring rotation and translation.
- Connected portals have the same geometry.
- Origin lies in the spot on the mesh for each portal.

I'll break up implementation into three sub problems:
1. Portal Transform: Thinking about what going from one portal to another means.
1. Portal Shading: What the transform means for rays.
2. Geometry Teleportation: What the transform means for geometry.

And finally before we talk about portals I should define that the front of a
portal is the side whose normal points up and the back of a portal is the side
that points down when rotation is zeroed.

# Portal Transform
{{ img(id="/blog/blender-portal/portal-ray-connect.gif") }}

We can think of transforming the in-portal so that it adopts the translation,
rotation, and scale of the out-portal. This can be seen in the image above, the
circles are portals and the lines between them can be thought as how we move the
rays to the other portal.

As usual we can see we need SRT for transformation order. Only once portals are
the same size and parallel can we actually move the rays between them, since
thats when the offset is constant between. This transform only applies to world
space we can just do rotation and scaling in local space for the incoming vector
for example.

# Portal Shading

Portal shading can be done with RayPortal3D BSDF Node, and most random youtube
tutorials do a decent job showcasing it. I ended up watching [Ray Portal BSDF
Node | Blender Tutorial](https://www.youtube.com/watch?v=cPSlFcKpW_Y) and I was
quite lucky to see this because if not I would have spent forever trying to figure
out why just doing the portal transform is insufficient for shading.

{{ img(id="/blog/blender-portal/offset-overview-light.png") }}

## Normal Correction

```swift
@geometryNode
func portalTransform(
    inputs: GroupInput,
    selfObject: Object,
    normal: Normal
) -> Geometry {
    var geometry = inputs.get("Geometry")
    let object = inputs.get("Object")
    
    let otherInfo = objectInfo(
        object: object,
        asInstance: false,
        transformSpace: .relative
    )
    
    // Ok now store the actual transforms
    let otherInfo = connectedPortal.objectInfo()
        .relative(selfObject)
    
    geometry.storeNamedAttribute(
        name: "offset_to_other",
        domain: .face,
        value: otherInfo.location
    )
    
    let rotateToOther = otherInfo.rotation.toEuler()
    geometry.storeNamedAttribute(
        name: "rotate_to_other", 
        domain: .face,
        value: rotateToOther
    )
    
    geometry.storeNamedAttribute(
        name: "scale_to_other",
        domain: .face, 
        value: otherInfo.scale
    )
    
    // Compute and cache normal correction
    let otherPortalNormal = vec3(0.0, 0.0, 1.0).eulerRotate(rotateToOther)
    // NOTE: This got flipped so I could use a + later.
    let dir = outPortalNormal.dot(vec3(0.0, 0.0, 1.0)) > 0.0 ? -1.0 : 1.0
    
    let normalCorrection = normal.normal
        .captureAttribute(domain: .face)
        .get("normal") * 0.001 * dir
    
    geometry.storeNamedAttribute(
        name: "normal_correction",
        domain: .face,
        value: normalCorrection
    )
    
    return geometry
}
```

For the computation of `dir` the &plusmn;1 is flipped to avoid needing to do a
subtraction in the shader (not that I think it would make a substantial
difference). And we only store the relative transforms since the original shader
is really only ever using the relative rotation and location (offset).

```swift
@shader
func portal(
    geometry: Geometry,
    attributes: Attributes
) -> MaterialOutput {
    let offsetToOther: Vec3 = attributes.geometry("offset_to_other")
    let rotateToOther: Vec3 = attributes.geometry("rotate_to_other")
    let scaleToOther: Vec3 = attributes.geometry("scale_to_other")
    
    let normalCorrection: Vec3 = attributes.geometry("normal_correction")
    
    // Direction ray should exit out-portal.
    let exitDirection = -1.0 * geometry.incoming.eulerRotate(rotateToOther)
    
    // Location ray should exit out-portal.
    let exitPosition = (geometry.position * scaleToOther)
        .eulerRotate(
            center: objectInfo.location,
            rotation: rotateToOther
        ) + offsetToOther + slightOffsetFromOutPortal
    
    // Move the ray and change direction
    let surface = rayPortalBSDF(
        color: .white,
        exitPosition: exitPosition,
        exitDirection: exitDirection
    )
    
    return MaterialOutput(
        surface: surface
    )
}
```

Do be careful thinking about the calculation for exit position though.
`geometry.position` is i

Note that we Must use Cycles since Ray Portal BSDF is only available in Cycles
([Ray Portal
BSDF](https://docs.blender.org/manual/en/latest/render/shader_nodes/shader/ray_portal.html)).
That's fine though since a ray tracer is more suited to portal rendering than a
rasterizer (EEVEE).

and I
personally found it to be quite helpful. In particular the info about the need
for a normal correction to offset the rays slightly from the portal surface so
they don't end up jumping back through the portal was particularly helpful (that
would have taken me awhile to figure out).

The shader the tutorial provides does this suboptimally since it computes the
rotation for every single ray (which probably adds up). The corrective normal
computation in the shader is also unnecessary since it is uniform across a
portals face (Blender or whatever it uses for shader compilation may be able to
cache it out through some advanced magic, but I'll cache it in an attribute to
guarantee that happens).



## Important Nodes
### Ray Portal BSDF
This is a shader that lets you arbitrarily set the direction and position of a ray that hits the shaded object.
If you don't supply a direction or position it will not be changed. A Ray Portal BSDF without supplied `Position`
or `Direction` is equivalent to a Transparent BSDF.

### Geometry
`Position`: Place the ray hit on the surface of the object.

`Normal`: Surface normal at the hit position.

`Incoming`: A vector (normalized I think) that points towards the viewer (active
camera in renders, and viewport camera in viewport).

{{ img(id="/blog/blender-portal/vector-overview-light.png") }}

### Object Info
`Location` here is the position of the object origin.

`Rotation` is of course the rotation of the object. The Blender UI presents
rotations in euler format, but they are internally stored as quaternions (hence
the need to perform a `to_euler` in the graph above).

>`Transform Space`: The transformation of the vector and geometry outputs.
> - `Original`: Output the geometry relative to the input object transform, and the location, rotation and scale relative to the world origin.
> - `Relative`: Bring the input object geometry, location, rotation and scale into the modified object, maintaining the relative position between the two objects in the scene.

Quick clafiying note on the transform space property (`Relative` or `Original`),
the [docs
explanation](https://docs.blender.org/manual/en/latest/modeling/geometry_nodes/input/scene/object_info.html)
has some clunky wording. `Original` is what you would see in the n-panel with
the object transforms, `Relative` is the difference between the transforms for
two objects. It is the transformation from the object with the geometry node
modifier to the object passed to `Object Info`.


# Geometry Teleportation
Here we actualy don't have so conventional a solution in Blender. There is a rough set
of things that needs to be done before this is working.

1. Divide the mesh into geometry that should stay and geometry that should be portaled.
2. Make the portaled geometry disappear at the in portal and appear at the out portal.

## Divide Geometry
Step one is a bit troublesome because we want our portals to work for static renders. And
presumably which side of the portal the geometry should exit on is determined by which
side it goes in on, but in a render we don't really have a concept of movement. To fix
this we need to define outside of the system which side of the system we want the geometry
moved on. I'll do this by accepting a vertex group and pulling the weights. A weight below
0.5 will mean it will exit the back of the out-portal and above and including 0.5 means
it will exit out the front of  the out-portal.

So we can figure out which geometry should be selected once it passes a certain side but
because I'm trying to store the weights in a single vertex group per mesh we need to perform
additional selection. Alot of implementation seem to just use a plane to cut the mesh and
everything on the other side of the plane is moved. That is not flexible enough for what I want.

1. The portal is just a face so we can extrude it by some depth along it's
normal and then the geometry past the face which lies inside of the prism is
what we want to transport.

2. For fully connected meshes we can select the edges that intersect with the portal face
then grow the selection "as much as we can."

Both have drawbacks. The first is problematic because it allows for the object to enter the portal from behind potentially even if it is not touching the portal face,
and the second is likely to be a very computationally expensive, requires source code modification/complex or hacky geometry node use. And both have upsides, the first is cheap to compute and easy to implement 
And the second will produce excellent results.

I'm going to go with the first and allow users to animate the depth property on each individual object to allow for teleporting or not teleporting the object.

## Teleport Geometry 
The best way to do this probably depends on the mesh type and portal set up. The best results will probably be achieved with an intersection boolean modifier, 
For dense meshes, it will likely be best to instance the object and on the instances if possible change the faces we want to be hidden to use the transparent BSDF shader.
Potentially a simple delete geometry might be solution as well.

I'm going to go with intersection because it is the most robust solution in the general case (e.g. it allows properly transporting the default cube).
