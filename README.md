# CleanPhysX
This is a repository of the PhysX engine—a mirror clone of the [QtQuick3DPhysics](https://github.com/qt/qtquick3dphysics/tree/dev/src/3rdparty/PhysX) PhysX backend with all hardware- and platform-specific code removed.

All patches created by Qt are applied in this repository. The only difference is that this repository is cleaned from platform-dependent code and is suitable for any standard C++ project.

If you want to integrate the PhysX engine into your project without any extra tricks, you can safely use this repository.

PhysX version 5.X

## Maintenance & Support

I plan to maintain and support this repository for the long term. My goal is to keep it up to date by regularly incorporating:

* Upstream NVIDIA changes: Tracking updates and fixes from the official NVIDIA PhysX repository.
* Qt patches: Integrating critical platform compatibility and optimization fixes from Qt.
* Community contributions: Accepting PRs and bug fixes from developers using this engine in their projects.

Contributions and bug reports are always welcome!

---

## CMake Integration

Add this repository as a git submodule to your project:

``` bash
git submodule add git@github.com:QuasarApp/CleanPhysX.git 
git submodule update --init --recursive
```

Add the following lines to your CMakeLists.txt:

``` cmake
add_subdirectory(CleanPhysX)

add_executable(my_project main.cpp)
target_link_libraries(my_project PRIVATE CleanPhysX)
```

---


### Useful Links

* 📖 [NVIDIA PhysX 5 SDK Documentation](https://nvidia-omniverse.github.io/PhysX/physx/latest/#) — Official API reference and integration guides.
* 💻 [NVIDIA PhysX GitHub Repository](https://github.com/NVIDIA-Omniverse/PhysX) — Upstream source repository by NVIDIA.
* 🌐 [NVIDIA GameWorks Developer Zone](https://developer.nvidia.com/physx-sdk) — NVIDIA PhysX overview and developer resources.
* 📦 [QtQuick3DPhysics Repository](https://github.com/qt/qtquick3dphysics/tree/dev) — Upstream Qt repository from which this backend is derived.
