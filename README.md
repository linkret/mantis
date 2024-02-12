<div align="center">
<img src="logo.svg" alt="Mantis Logo" width="400" height="400">
</div>


Mantis is a blazing fast C++ library designed for distance to mesh queries. 
It implements the [P2M algorithm](https://yuemos.github.io/Projects/P2M/pdf/P2M_author.pdf) and uses SIMD instructions to significantly accelerate query performance.

## Features

- **Blazingly Fast**: Mantis is about 2x faster than the original P2M implementation and ca. 4x faster than FCPW.

- **MIT Licensed**: Mantis is permissively licensed under MIT license.

- **Simple to Build**: Thanks to [geograms](https://github.com/BrunoLevy/geogram) pluggable delaunay module, Mantis only consists of four files and doesn't require any external dependencies.

- **Fast Preprocessing**: Not yet there, but on the roadmap. 

## Getting Started

### Prerequisites

To build mantis you will need a C++ Compiler with C++11 support.

### Building

Mantis consists of four files `mantis.h`, `mantis.cpp`, `Delaunay_psm.h` and `Delaunay_psm.cpp`.
You can just grab these files and add them to your build system of choice. 
For convenience, you can also use the provided CMakeLists.txt. For example 
to fetch and use with CMake's `FetchContent`:

```cmake
FetchContent_Declare(
  mantis
  GIT_REPOSITORY 
)

FetchContent_MakeAvailable(mantis)

target_link_libraries(my_target PRIVATE mantis)
```

## Example Usage

Here is a simple example of how to use mantis.

```cpp
#include <mantis.h>

// you're triangle mesh
std::vector<std::array<float, 3>> points;
std::vector<std::array<uint32_t, 3>> triangles;

// build the acceleration structure
mantis::AccelerationStructure accelerator(points, triangles);

// choose a query point
std::array q = {0.f, 1.f, 2.f};

// perform the query
mantis::Result result = accelerator.calc_closest_point(q);

printf("distance to mesh is %f\n", std::sqrt(result.distance_squared));

```

The example folder contains two more complex examples how mantis can be used.
walk of spheres algorithm and how mantis can be used to turn a triangle mesh into a signed distance field.

## Benchmarks

These benchmarks compare the performance of mantis with the original P2M implementation and FCPW for three different meshes (which can be found in the assets folder). 
The test meshes are normalized to be contained in a unit bounding box and the queries are sampled uniformly at random from the bounding box [-5, 5]^3.
The benchmark was run on a MacBook Pro with an M2 Max. For FCPW the simd width was set to 4. On top of every bar
you can see the slowdown compared to mantis.

<div align="center">
<img src="bench.png" alt="Performance Comparison with original P2M and FCPW" width="1000">
</div>

## Caviats

- Currently, mantis is only tested with meshes which don't have duplicate vertices.
- For vertices with long interception lists, the original P2M paper suggests to implement an R-tree to speed up the query. This is not implemented in Mantis yet. This could potentially lead to a slowdown for highly symmetric shapes such as a perfect sepeher.
- Even with the R-tree optimizations, shapes such as a perfect sphere will consume much more memory than a bvh since every interception list will contain ever face.