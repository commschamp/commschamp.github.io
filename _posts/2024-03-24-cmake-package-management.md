---
date: 2024-03-22 00:00:01
title: CMake bundle of all CommsChampion Ecosystem projects
categories:
  - news
---

New [cc.cmake](https://github.com/commschamp/cc.cmake) project was created.
It bundles all the [CommsChampion Ecosystem](https://commschamp.github.io/) projects
into a single CMake one, and is expected to be used in another CMake project that has some
[CommsChampion Ecosystem](https://commschamp.github.io/) dependencies and built using
the [ExternalProject_Add()](https://cmake.org/cmake/help/latest/module/ExternalProject.html) cmake
function with an appropriate configuration.
