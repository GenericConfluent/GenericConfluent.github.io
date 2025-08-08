+++
date = '2025-08-06T10:12:09-06:00'
title = 'Blender Portal'
description = "Quest for a decent portal setup in blender. No stylization, just infinite recursion and mesh teleporation."
tags = ['blender', 'math', 'effects']
draft = true
+++
I somehow got possesed by the idea that I *needed* to implement a portal in blender, and a proper explanation of the
theory surrounding the implementation details seemed quite lacking&mdash;and I like to have a concrete handle on what I'm
doing when I work on something. So here we are. The goal of this post is not quite to create a radical new and better
portal impl, but moreso to pick among the pieces people have already developed and assemble them into a good and reusable
impl.

Here is the feature list I want from the implementation:
1. Portal transforms can be modified without breaking the setup.
2. Transport geometry between portals so there is no need to duplicate objects travelling through.
3. When we see a portal through the portal, there will be a portal&trade;

Here are the restrictions:
- A Portal and its origin lie in a 2D plane.
- Connected portals have the same geometry.
- Origin lies in the spot on the mesh for each portal.

I'll break up implementation into two sub problems:
1. Portal Shading
2. Geometry Teleportation

# Portal Shading
Portal shading can be done with RayPortal3D BSDF Node, and most random youtube
tutorials do a decent job showcasing it. My choice for this is [Ray Portal BSDF
Node | Blender Tutorial](https://www.youtube.com/watch?v=cPSlFcKpW_Y) and I
personally found it to be quite helpful. In particular the info about the need
for a normal correction to offset the rays slightly from the portal surface so
they don't end up jumping back through the portal was particularly helpful (that
would have taken me awhile to figure out).

## Tutorial Node Setup
I'm going to write this with Rust like syntax because I'm working in a text medium but this is
of course done in nodes. Note that the `Geometry` objects are different between shader and geometry
node contexts.

```rust
#[geometry_node]
fn portal_transform(inputs: GroupInput, self_object: Object) -> Geometry {
    let mut geometry = inputs.get("Geometry");
    let connected_portal = inputs.get("Object");

    // NOTE: The tutorial names these differently.
    geometry.store_named_attribute(
        "location_other",
        connected_portal.object_info().location()
    );
    geometry.store_named_attribute(
        "rotation_other",
        connected_portal.object_info().rotation().to_euler()
    );

    geometry.store_named_attribute(
        "rotation_other_relative",
        connected_portal.object_info().relative_rotation(self_object).to_euler()
    );

    geometry.store_named_attribute(
        "rotation_self",
        self_object.object_info().rotation().to_euler()
    );

    return geometry;
}
```

And then the following shader needs to be put on each portal object.
```rust
// This runs every single time a ray hits our object since it relies on the
// incoming attribute on the Geometry Node.
#[shader]
fn portal(
    geometry: Geometry,
    object_info: ObjectInfo,
    attributes: Attributes,
) -> MaterialOutput {
    let rotation_self = attributes.geometry("rotation_self");

    let location_other = attributes.geometry("location_other");
    let rotation_other = attributes.geometry("rotation_other");
    let rotation_other_relative = attributes.geometry("rotation_other_relative");

    // Compute the direction a ray should exit out-portal.
    let exit_direction = (-1.0 * geometry.incoming)
        .vector_rotate(A {
            inverse: true,
            kind: RotationKind::Euler,
            center: Vec3::ZERO,
            rotation: rotation_self,
        })
        .vector_rotate(A {
            inverse: false,
            kind: RotationKind::Euler,
            center: Vec3::ZERO,
            rotation: rotation_other,
        })

    // Compute corrective normal so rays don't accidentally hit the out portal once moved.
    let alignment = vec3(0.0, 0.0, 1.0)
        .vector_rotate(A {
            inverse: false,
            kind: RotationKind::Euler,
            center: Vec3::ZERO,
            rotation: rotation_other_relative,
        })
        .dot(vec3(0.0, 0.0, 1.0));
    let offset_direction = if alignment > 0.0 { 1.0 } else { -1.0 };
    let slight_offset_from_out_portal = geometry.normal * 0.001 * offset_direction;

    // Location ray should exit out-portal.
    let offset_to_other = location_other - object_info.location;
    let exit_position = (geometry.position + offset_to_other)
        .vector_rotate(A {
            inverse: true,
            kind: RotationKind::Euler,
            center: location_other,
            rotation: rotation_self,
        })
        .vector_rotate(A {
            inverse: false,
            kind: RotationKind::Euler,
            center: location_other,
            rotation: rotation_other_relative,
        }) - slight_offset_from_out_portal;

    // Move the ray and change direction
    let surface = ray_portal_bsdf(Color::WHITE, exit_position, exit_direction);

    return MaterialOutput {
        surface,
        ..Default::default()
    };
}
```
Note the following consequences of this implementation:

- Must use Cycles since Ray Portal BSDF is only available in Cycles ([Ray Portal
BSDF](https://docs.blender.org/manual/en/latest/render/shader_nodes/shader/ray_portal.html)).
That's fine though since a ray tracer is more suited to portal rendering than a
rasterizer (EEVEE).
- Still doesn't transport geometry.

## Important Nodes
### Ray Portal BSDF
This is a shader that lets you arbitrarily set the direction and position of a ray that hits the shaded object.
If you don't supply a direction or position it will not be changed. A Ray Portal BSDF without supplied `Position`
or `Direction` is equivalent to a Transparent BSDF.

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

### Geometry
`Position`: Place the ray hit on the surface of the object.

`Normal`: Surface normal at the hit position.

`Incoming`: A vector (normalized I think) that points towards the viewer (active
camera in renders, and viewport camera in viewport).

## Portal Transform
We can think of transforming the in-portal so that it adopts the translation,
rotation, and scale of the out-portal. Intuitively we can just apply the same
transformation to any ray or object we want to move to our out-portal.

The shader the tutorial provides does this suboptimally since it computes the
rotation for every single ray (which probably adds up). The corrective normal
computation in the shader is also unnecessary since it is uniform across a
portals face (Blender or whatever it uses for shader compilation may be able to
cache it out through some advanced magic, but I'll cache it in an attribute to
guarantee that happens).

## Clean, Optimize, and Scale
So here is my modified geometry node.
```rust
#[geometry_node]
fn portal_transform(
    inputs: GroupInput,
    self_object: Object,
    normal: Normal,
) -> Geometry {
    let mut geometry = inputs.get("Geometry");
    let object = inputs.get("Object");

    let other_info = ObjectInfo {
        object,
        as_instance: false,
    }.eval(ObjectInfoProps {
        transform_space: TransformSpace::Relative,
    });

    // Ok now store the actual transforms
    let other_info = connected_portal.object_info()
        .relative(self_object);

    geometry.store_named_attribute(Attribute {
        name: "offset_to_other",
        domain: Domain::Face,
        value: other_info.location,
    });

    let rotate_to_other = other_info.rotation.to_euler();
    geometry.store_named_attribute(Attribute {
        name: "rotate_to_other",
        domain: Domain::Face,
        value: rotate_to_other,
    });

    geometry.store_named_attribute(
        name: "scale_to_other",
        domain: Domain::Face,
        value: other_info.scale
    );

    // Compute and cache normal correction
    let other_portal_normal = vec3(0.0, 0.0, 1.0).euler_rotate(rotate_to_other);
    let dir = if out_portal_normal.dot(vec3(0.0, 0.0, 1.0)) > 0.0 { -1.0 } else { 1.0 };

    let normal_correction = normal.normal
        .capture_attribute(Domain::Face)
        .get("normal") * 0.001 * dir;

    geometry.store_named_attribute(Attribute {
        name: "normal_correction",
        domain: Domain::Face,
        value: normal_correction,
    });

    return geometry;
}
```

For the computation of `dir` the &plusmn;1 is flipped to avoid needing to do a
subtraction in the shader (not that I think it would make a substantial
difference). And we only store the relative transforms since the original shader
is really only ever using the relative rotation and location (offset).

```rust
#[shader]
fn portal(
    geometry: Geometry,
    attributes: Attributes,
) -> MaterialOutput {
    let offset_to_other: Vec3 = attributes.geometry("offset_to_other");
    let rotate_to_other: Vec3 = attributes.geometry("rotate_to_other");
    let scale_to_other: Vec3 = attributes.geometry("scale_to_other");

    let normal_correction: Vec3 = attributes.geometry("normal_correction");

    // Direction ray should exit out-portal.
    let exit_direction = -1.0 * geometry.incoming.euler_rotate(rotate_to_other);

    // Location ray should exit out-portal.
    let exit_position = (geometry.position * scale_to_other)
        .euler_rotate({
            center: object_info.location,
            rotation: rotate_to_other,
        }) + offset_to_other + slight_offset_from_out_portal;

    // Move the ray and change direction
    let surface = ray_portal_bsdf(Color::WHITE, exit_position, exit_direction);

    return MaterialOutput {
        surface,
        ..Default::default()
    };
}
```
Now that cleans things up quite nicely. Do be careful thinking about the calculation for
exit position though. `geometry.position` is i

# Geometry Teleportation
