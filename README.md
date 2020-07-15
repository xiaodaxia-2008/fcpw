# FCPW

FCPW is a lightweight, header only C++ library for fast closest point and ray intersection queries. It is about 3x faster than <a href="https://www.embree.org">Embree</a> for closest point queries and is only slightly slower for ray intersection queries (0.8x) (see [Benchmarks](#Benchmarks)). FCPW uses a wide BVH with vectorized traversal to accelerate queries to geometric primitives, with additional support for constructive solid geometry (CSG) and instancing. The geometric primitives currently supported are triangles, line segments or a mixture of the two, though it is fairly easy to add support for other types of primitives in the library. FCPW uses the amazing <a href="https://github.com/mitsuba-renderer/enoki">Enoki</a> library for vectorization, though it falls back to <a href="http://eigen.tuxfamily.org/index.php?title=Main_Page">Eigen</a> if Enoki is not included in the project. In the latter case, FCPW performs queries with a non-vectorized binary BVH.

# Including FCPW

FCPW does not require compiling or installing, you just need to add the following lines

```
#define FCPW_USE_ENOKI ON
#define FCPW_SIMD_WIDTH 8
#include <fcpw/fcpw.h>
```

in your code. Alternatively, you can avoid the `#define`s by adding the following lines to your CMakeLists.txt file

```
add_subdirectory(fcpw)
target_link_libraries(YOUR_TARGET fcpw)
target_include_directories(YOUR_TARGET PUBLIC ${FCPW_EIGEN_INCLUDES})
if(FCPW_USE_ENOKI)
	target_include_directories(YOUR_TARGET PUBLIC ${FCPW_ENOKI_INCLUDES})
endif()
```

# API

The easiest and most direct way to use FCPW is through its `Scene` class that provides methods to load the geometry, build the acceleration structure and perform geometric queries. Here is an example of doing this with a geometric object consisting of triangles

```
// initialize the scene
Scene<3> scene;

// set the types of primitives the objects in the scene contain;
// in this case, we have a single object consisting of only triangles
scene.setObjectTypes({{PrimitiveType::Triangle}});

// set the vertex and triangle count of the (0th) object
scene.setObjectVertexCount(nVertices, 0);
scene.setObjectTriangleCount(nTriangles, 0);

// specify the vertex positions
for (int i = 0; i < nVertices; i++) {
	scene.setObjectVertex(position[i], i, 0);
}

// specify the triangle indices
for (int i = 0; i < nTriangles; i++) {
	scene.setObjectTriangle(&indices[3*i], i, 0);
}

// now that the geometry has been specified, build the acceleration structure
scene.build(AggregateType::Bvh_SurfaceArea, true); // the second boolean argument enables vectorization

// perform closest point query
Interaction<3> interaction;
scene.findClosestPoint(queryPoint, interaction);

// perform ray intersection query
std::vector<Interaction<3>> interactions;
scene.intersect(queryRay, interactions, false, true); // don't check for occlusion, and record all hits
```

Notice that `Scene` is templated on dimension, enabling FCPW to work with geometric data in any dimension out of the box as long the geometric primitives are specialized to the dimension of interest as well. The `Interaction` object stores all the relevant information pertaining to the query, such as the parametric distance to the geometric primitive, the point of intersection or closest point on the primitive, the local uv coordinates of that point, as well as the primitive's index. FCPW can additionally compute the normal at the point of intersection or closest point, though this must be explicitly requested through the `computeObjectNormals` method in the `Scene`. Furthermore, it is possible to load multiple objects, possibly with mixed primitives and instance transforms, into the `Scene`. A CSG tree can also be built via the `setCsgTreeNode` method. More details can be found in `fcpw.h`.

# Benchmarks

TODO

# Author
[Rohan Sawhney](http://www.rohansawhney.io), with support from Ruihao Ye and Johann Korndoerfer

# License

Released under the [MIT License](https://opensource.org/licenses/MIT)
