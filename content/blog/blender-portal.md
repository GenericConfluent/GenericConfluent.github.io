+++
date = '2025-08-06T10:12:09-06:00'
updated = '2025-08-15T12:53:41-06:00'
title = 'Blender Portal'
description = "Quest for a decent portal setup in blender. No stylization, just infinite recursion and mesh teleporation."
tags = ['blender', 'math', 'effects']
draft = false
+++

{{ img(id="/blog/blender-portal/result.gif") }}

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
- Origin is a point from the portals face.

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

It turns out that offsetting the exiting ray from the out portal with the normal
is a very good idea because it ensures that the cycles doesn't think the ray hit
the portal the next time it is supposed to be moving.

{{ img(id="/blog/blender-portal/portal-store-transform.svg") }}

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
    let otherInfo = objectInfo(
        // NOTE: How object is transformed relative to self object.
        transformSpace: .relative,
        object: connectedPortal,
        asInstance: false,
    )

    geometry.storeNamedAttribute(
        dataType: .vector,
        domain: .face,
        name: "offset_to_other",
        value: otherInfo.location,
    )

    let rotateToOther = otherInfo.rotation.toEuler()
    geometry.storeNamedAttribute(
        dataType: .vector,
        domain: .face,
        name: "rotate_to_other",
        value: rotateToOther,
    )

    geometry.storeNamedAttribute(
        dataType: .vector,
        domain: .face,
        name: "scale_to_other",
        value: otherInfo.scale,
    )

    // Compute and cache normal correction
    let otherPortalNormal = vec3(0.0, 0.0, 1.0)
      .vectorRotate(
          type: .euler,
          invert: false,
          rotation: rotateToOther,
      )

    return geometry
}
```

My version of both of these is slightly different from the tutorial video I
linked to above. I only store relative transforms because that's all you really
need, and also I modified the position transform to accomodate my need to
support portal scaling.


{{ img(id="/blog/blender-portal/portal-material-nodes.svg") }}

```swift
@shader
func portal(
    geometry: Geometry,
    objectInfo: ObjectInfo,
    attributes: Attributes
) -> MaterialOutput {
    let offsetToOther: Vec3 = attributes.geometry("offset_to_other")
    let rotateToOther: Vec3 = attributes.geometry("rotate_to_other")
    let scaleToOther: Vec3 = attributes.geometry("scale_to_other")

    let normalCorrection: Vec3 = -0.001 * geometry.normal

    // Direction ray should exit out-portal.
    let exitDirection = -1.0 * geometry.incoming
        .vectorRotate(
            type: .euler,
            invert: false,
            center: Vec3.zero,
            rotation: rotateToOther
        )

    // Location of the ray on the output portal surface
    let exitPosition = mapping(
        type: .point,
        vector: geometry.position + normalCorrection,
        location: offsetToOther,
        rotation: rotateToOther,
        scale: scaleToOther,
    )

    // Move the ray and change direction
    let surface = rayPortalBSDF(
        color: .white,
        position: exitPosition,
        direction: exitDirection
    )

    return MaterialOutput(
        surface: surface
    )
}
```

Note that we Must use Cycles since Ray Portal BSDF is only available in Cycles
([Ray Portal
BSDF](https://docs.blender.org/manual/en/latest/render/shader_nodes/shader/ray_portal.html)).
That's fine though since a ray tracer is more suited to portal rendering than a
rasterizer (EEVEE).

## Important Nodes
### Ray Portal BSDF
This is a shader that lets you arbitrarily set the direction and position of a
ray that hits the shaded object. If you don't supply a direction or position it
will not be changed. A Ray Portal BSDF without supplied `Position` or
`Direction` is equivalent to a Transparent BSDF.

### Geometry

`Position`: Place the ray hit on the surface of the object (world space).

`Normal`: Surface normal at the hit position. (Normalized)

`Incoming`: A vector that points towards the direction the ray came from. (Normalized)

{{ img(id="/blog/blender-portal/vector-overview-light.png") }}

### Object Info
`Location` here is the position of the object origin. (world space)

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

1. Divide the mesh into geometry that should stay and geometry that should be portaled. (hard)
2. Make the portaled geometry disappear at the in portal and appear at the out portal. (easy)

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

2. For fully connected meshes we can select the edges that intersect with the
portal face then isolate mesh components which lie on either side of the portal.

Approach #1 is not it for me. To isolate all the mesh in a volume the initial
instinct would be to do a mesh intersection. Except the problem with that is
that it will nuke your mesh attributes because it may be the case that non of
the vertices after the intersection was in the original mesh. And so to play
nice with shading we need to restrict ourselves to mesh differences. And now
considering the approach I suppose you could do a cut by making your volume and
flipping the faces on it and going through each face and doing a boolean
difference (after making sure you join the original planar face back to the
extrusion flipped correctly, convex-hull is a one node solution to this but
fixing the geometry manually isn't much a hassle either and is faster ), but
that would be incredibly slow unless you implement the difference yourself and
just delete edges from the mesh. Which in that case would make the final mesh
pieces appear pretty jagged near the portal surface and would not be a viable
solution for meshes with little geometry.

I went with approach #2. Cut the mesh in half, isolate the islands that are left
on the other side of the portal and move them to the out portal. Unfortunatly mesh
slow&mdash;like really really slow. Luckily Blender 4.5 introduces a solver optimized
for booleans of manifold meshes which is significantly faster. So I have basically two
versions of my mesh portal nodes. A fast one for manifold geometry and a significantly
slower one which is set to use the exact boolean solver.

The rough template looks like this:
{{ img(id="/blog/blender-portal/portal-cut-geometry.svg") }}

```swift
@nodeGroup
func cutGeometryWithSurface(
    geometry: Geometry,
    surface: Object,
) -> Geometry {
    // This is the smallest I could get without breakage. But that was probably
    // because I set my merge distance to 0.001 also.
    let cutWidth = 0.001;
    let surfaceInfo = objectInfo(surface)

    // Build the cutting cylinder.
    let cylinder: Geometry = transformGeometry(
        // NOTE: Pay attention to all here when in the context of manifold portal.
        // If we don't merge globally we'll get holes.
        geometry: mergeByDistance(
            mode: .all,
            distance: 0.001,
            geometry: joinGeometry(geometry: [flipFaces(mesh: surfaceInfo.geometry), extrudeMesh(mesh: surfaceInfo.geometry, offsetScale: cutWidth)])
        ),
        translation: combineXYZ(z: cutWidth / -2.0).rotateVector(rotation: surfaceInfo.rotation),
    )

    // If we have non manifold geometry we can just swap the solver.
    // Unfortunately I would need to duplicate the node and add logic for this.
    // And I don't want to do that <(｀^´)>
    return meshBoolean(
        operation: .difference,
        solver: .manifold,
        mesh1: geometry,
        mesh2: cylinder,
    )
}
```

Ok so we have cut the mesh like a hot metal ruler cuts through air. Now we need
to consider the disjoint pieces of the mesh and decide which pieces should be
moved. The obvious thing to do is just find chunks of geometry whose vertices
are all past the portal surface. But we should be specific about what "past"
means.

The most basic past means on a certain side of the plane containing the portal
surface. Using this definition of past though would result in the mesh being
transported to the out portal whenever the modifier is active. We could
constrain ourselves a little bit and say that the closest point to the plane
must lie within the portal boundary and the offset from that point to our vertex
must be normal to the plane. That would give you an approach #1 flavoured result
where all the vertices of the mesh piece would need to be within an infinetly
extruded prism along the portal normal. That's pretty easy to do with Geometry
Proximity. You could also require that at least one vertex is within a certain
distance of the portal. Or say that all the vertices of the mesh must have a
certain weight in a given vertex group as I suggested earlier.

The criteria you chouse of course depends on the use case. I'm trying to go generic. So I'll only use the most generic criteria: all your vertices are on
a certain side of the plane.

{{ img(id="/blog/blender-portal/group-signed-distance.svg") }}
{{ img(id="/blog/blender-portal/portal-teleport-geometry.svg") }}
```swift
@nodeGroup
func signedDistanceFromPlane(
    planeNormal: Vec3,
    pointOnPlane: Vec3,
    vector: Vec3,
) -> float {
    return planeNormal.dot(vector) - planeNormal.dot(pointOnPlane)
}

@geometryNodes
func portalManifoldObject(
    geometry: Geometry,
    position: Position,
    meshIsland: MeshIsland,
    inPortal: Object,
    outPortal: Object,
) -> Geometry {
    let inPortalInfo = objectInfo(object: inPortal, transformSpace: .relative)
    let inPortalNormal = vec3(z: 1.0).rotateVector(rotation: inPortalInfo.rotation)

    let dist = signedDistanceFromPlane(
        planeNormal: inPortalNormal,
        pointOnPlane: inPortalInfo.location,
        // I didn't actually use the evaluate on domain, this is just for clarity
        vector: evaluateOnDomain(type: .vector, domain: .point, value: position)
    )

    let criteria = accumulateField(
        type: .float
        domain: .point,
        value: dist < 0.0 && notEqual(A: dist, B: 0.0, epsilon: 0.01),
        groupId: meshIsland.islandIndex,
    )

    let pieces = separateGeometry(
        geometry: cutGeometryWithSurface(geometry: geometry, surface: inPortal),
        selection: criteria,
    )

    return joinGeometry([
        relativeTransformGeometry(fromObject: inPortal, toObject: outPortal, geometry: pieces.selected),
        pieces.inverted,
    ])
}
```

## Portal Geometry

This is trivially easy compared to division and on a nice note since we're in
geometry nodes now we can actually use matrices which makes our transform
really nice:

{{ img(id="/blog/blender-portal/transform-relative-geometry.svg") }}
```swift
@nodeGroup
func relativeTransformGeometry(
    fromObject: Object,
    toObject: Object,
    geometry: Geometry,
) -> Geometry {
    let selfToFrom = objectInfo(transformSpace: .relative, object: fromObject).transform
    let selfToTo = objectInfo(transformSpace: .relative, object: toObject).transform

    // I wrote it like this to be clear about the argument order for the
    // node. Order is as you would expect, with (mat0, mat1) -> mat0 * mat1
    return multiplyMatrices(
        selfToTo, invertMatrix(selfToFrom).matrix,
    )
}
```

And that is about all.
