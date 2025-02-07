---
title:  Triplet files
description: Understand the use and content of triplets in vcpkg.
ms.date: 11/30/2022
---
# Triplet files

*Triplet* is a standard term used in cross compiling as a way to completely capture the target environment (cpu, os, compiler, runtime, etc.) in a single, convenient name.

In vcpkg, we use triplets to describe an imaginary "target configuration set" for every library. Within a triplet, libraries are generally built with the same configuration, but it is not a requirement. For example, you could have one triplet that builds `openssl` statically and `zlib` dynamically, one that builds them both statically, and one that builds them both dynamically (all for the same target OS and architecture).

A single build will consume files from up to two triplets: the target triplet and the host triplet. If you need to apply different settings for different libraries, you must make a single custom triplet with that combination of settings. See [Per-port customization](#per-port-customization) below for how to accomplish this.

vcpkg comes with pre-defined triplets for many common platforms and configurations. Run `vcpkg help triplet` to get the list available in your environment.

## Triplet selection

To select the target triplet,
- With the CLI in [Classic mode](classic-mode.md), you can qualify package references with the triplet name, such as `zlib:x64-windows-static-md`.
- With the CLI, you can pass [`--triplet=<triplet>`](../commands/common-options.md#triplet).
- With CMake, you can set [`VCPKG_TARGET_TRIPLET`](buildsystems/cmake-integration.md#vcpkg_target_triplet).
- With MSBuild, you can set [`VcpkgTriplet`](buildsystems/msbuild-integration.md#vcpkgtriplet).

To select the host triplet for the current machine,
- With the CLI, you can pass [`--host-triplet=<triplet>`](../commands/common-options.md#host-triplet).
- With CMake, you can set [`VCPKG_HOST_TRIPLET`](buildsystems/cmake-integration.md#vcpkg_host_triplet).
- With MSBuild, you can set [`VcpkgHostTriplet`](buildsystems/msbuild-integration.md#vcpkghosttriplet).

## Community triplets

Triplets contained in the `triplets\community` folder are not tested by the curated catalog's continuous integration, but are commonly requested by the community. Because we do not have continuous coverage, port updates may break compatibility with community triplets. We will gladly accept and review contributions that aim to solve issues with these triplets.

When using a community triplet, a message like the following one will be printed during a package install:

```console
-- Using community triplet x86-uwp. This triplet configuration is not guaranteed to succeed.
-- [COMMUNITY] Loading triplet configuration from: D:\src\vcpkg\triplets\community\x86-uwp.cmake
```

## Adding or replacing triplets

You can extend vcpkg by replacing in-the-box triplets or creating new triplets for your particular project.

First, copy a built-in triplet file from the `triplets\` directory into a project local location. Then, add that directory to the list of overlay triplet paths when interacting with vcpkg.

- In [Manifest mode](manifests.md), you can use [`$.vcpkg-configuration.overlay-triplets`](../reference/vcpkg-configuration-json.md#overlay-triplets)
- When using vcpkg from CMake, you can set [`VCPKG_OVERLAY_TRIPLETS`](buildsystems/cmake-integration.md#vcpkg_overlay_triplets)
- When using vcpkg from MSBuild, you can add [`--overlay-triplets=...`][overlay-triplets] to [MSBuild Additional Options](buildsystems/msbuild-integration.md#vcpkg-additional-install-options).
- When using the CLI directly, you can pass [`--overlay-triplets=...`][overlay-triplets]
- You can set the [`$VCPKG_OVERLAY_TRIPLETS`](config-environment.md#vcpkg_overlay_triplets) environment variable to a list of overlay paths.

See our [overlay triplets example](examples/overlay-triplets-linux-dynamic.md) for a more detailed walkthrough.

[overlay-triplets]: ../commands/common-options.md#overlay-triplets

## Variables

### VCPKG_TARGET_ARCHITECTURE

Specifies the target machine architecture.

Valid options are `x86`, `x64`, `arm`, `arm64` and `wasm32`.

### VCPKG_CRT_LINKAGE

Specifies the desired CRT linkage (for MSVC).

Valid options are `dynamic` and `static`.

### VCPKG_LIBRARY_LINKAGE

Specifies the preferred library linkage.

Valid options are `dynamic` and `static`. Libraries can ignore this setting if they do not support the preferred linkage type.

### VCPKG_BUILD_TYPE

You can set this value to `release` to only build release versions of the ports. By default this value is empty and release and debug versions of a port are built.

### VCPKG_CMAKE_SYSTEM_NAME

Specifies the target platform.

Valid options include any CMake system name, such as:

- Empty (Windows Desktop for legacy reasons)
- `WindowsStore` (Universal Windows Platform)
- `MinGW` (Minimalist GNU for Windows)
- `Darwin` (Mac OSX)
- `iOS` (iOS)
- `Linux` (Linux)
- `Emscripten` (WebAssembly)

### VCPKG_CMAKE_SYSTEM_VERSION

Specifies the target platform system version.

This field is optional and, if present, will be passed into the build as `CMAKE_SYSTEM_VERSION`.

See also the CMake documentation for `CMAKE_SYSTEM_VERSION`: https://cmake.org/cmake/help/latest/variable/CMAKE_SYSTEM_VERSION.html.

### VCPKG_CHAINLOAD_TOOLCHAIN_FILE

Specifies an alternate CMake Toolchain file to use.

This (if set) will override all other compiler detection logic. By default, a toolchain file is selected from `scripts/toolchains/` appropriate to the platform.

See also the CMake documentation for toolchain files: https://cmake.org/cmake/help/v3.11/manual/cmake-toolchains.7.html.

### VCPKG_CXX_FLAGS

Sets additional compiler flags to be used when not using `VCPKG_CHAINLOAD_TOOLCHAIN_FILE`.

This option also has forms for configuration-specific and C flags:

- `VCPKG_CXX_FLAGS`
- `VCPKG_CXX_FLAGS_DEBUG`
- `VCPKG_CXX_FLAGS_RELEASE`
- `VCPKG_C_FLAGS`
- `VCPKG_C_FLAGS_DEBUG`
- `VCPKG_C_FLAGS_RELEASE`

### VCPKG_LINKER_FLAGS

Sets additional linker flags to be used while building dynamic libraries and
executables in the absence of `VCPKG_CHAINLOAD_TOOLCHAIN_FILE`.

This option also has forms for configuration-specific flags:

- `VCPKG_LINKER_FLAGS`
- `VCPKG_LINKER_FLAGS_DEBUG`
- `VCPKG_LINKER_FLAGS_RELEASE`

### VCPKG_MESON_CONFIGURE_OPTIONS

Set additional Meson configure options that are appended to the configure command (in [`vcpkg_configure_meson`](../maintainers/functions/vcpkg_configure_meson.md)).

This field is optional.

Also available as build-type specific `VCPKG_MESON_CONFIGURE_OPTIONS_DEBUG` and `VCPKG_MESON_CONFIGURE_OPTIONS_RELEASE` variables.

### VCPKG_MESON_NATIVE_FILE_RELEASE

Provide an additional configuration dependent file as a meson cross/native file. Can be used to override settings provided by vcpkg since it will be passed after vcpkg's generated cross/native files are passed.

Especially useful to provide your own build_machine and host_machine entries.

### VCPKG_MESON_NATIVE_FILE_DEBUG

See [VCPKG_MESON_NATIVE_FILE_RELEASE](#vcpkg_meson_native_file_release).

### VCPKG_MESON_CROSS_FILE_RELEASE

See [VCPKG_MESON_NATIVE_FILE_RELEASE](#vcpkg_meson_native_file_release).

### VCPKG_MESON_CROSS_FILE_DEBUG

See [VCPKG_MESON_NATIVE_FILE_RELEASE](#vcpkg_meson_native_file_release).

### VCPKG_CMAKE_CONFIGURE_OPTIONS

Set additional CMake configure options that are appended to the configure command (in [`vcpkg_cmake_configure`](../maintainers/functions/vcpkg_cmake_configure.md)).

This field is optional.

Also available as build-type specific `VCPKG_CMAKE_CONFIGURE_OPTIONS_DEBUG` and `VCPKG_CMAKE_CONFIGURE_OPTIONS_RELEASE` variables.

### VCPKG_MAKE_CONFIGURE_OPTIONS

Set additional automake / autoconf configure options that are appended to the configure command (in [`vcpkg_configure_make`](../maintainers/functions/vcpkg_configure_make.md)).

This field is optional.

For example, to skip certain libtool checks that may errantly fail:

```cmake
set(VCPKG_MAKE_CONFIGURE_OPTIONS "lt_cv_deplibs_check_method=pass_all")
```

Also available as build-type specific `VCPKG_MAKE_CONFIGURE_OPTIONS_DEBUG` and `VCPKG_MAKE_CONFIGURE_OPTIONS_RELEASE` variables.

### VCPKG_DEP_INFO_OVERRIDE_VARS

[!INCLUDE [experimental](../../includes/experimental.md)]

Replaces the default computed list of triplet "Supports" terms.

This option (if set) will override the default set of terms used for [Platform Expression](../reference/vcpkg-json.md#platform-expression) evaluation.

See the [`"supports"`](../reference/vcpkg-json.md#supports) manifest file field documentation for more details.

> Implementers' note: this list is extracted via the `vcpkg_get_dep_info` mechanism.

### VCPKG_DISABLE_COMPILER_TRACKING

When this option is set to `TRUE`, `ON`, or `1`, the compiler will not be tracked as part of the package abis.

This will cause [Binary Caching](binarycaching.md) to reuse builds from older or newer compilers.

## Windows-specific Variables

### VCPKG_ENV_PASSTHROUGH

Instructs vcpkg to allow additional environment variables into the build process.

On Windows, vcpkg builds packages in a special clean environment that is isolated from the current command prompt to
ensure build reliability and consistency. This triplet option can be set to a list of additional environment variables
that will be added to the clean environment. The values of these environment variables will be hashed into the package
abi -- to pass through environment variables without abi tracking, see `VCPKG_ENV_PASSTHROUGH_UNTRACKED`.

See also the `vcpkg env` command for how you can inspect the precise environment that will be used.

> Implementers' note: this list is extracted via the `vcpkg_get_tags` mechanism.

### VCPKG_ENV_PASSTHROUGH_UNTRACKED

Instructs vcpkg to allow additional environment variables into the build process without abi tracking.

See `VCPKG_ENV_PASSTHROUGH`.

### <a name="VCPKG_VISUAL_STUDIO_PATH"></a> VCPKG_VISUAL_STUDIO_PATH

Specifies the Visual Studio installation to use.

To select the precise combination of Visual Studio instance and toolset version, we walk through the following algorithm:

1. Determine the setting for `VCPKG_VISUAL_STUDIO_PATH` from the triplet, or the environment variable `VCPKG_VISUAL_STUDIO_PATH`, or consider it unset
1. Determine the setting for `VCPKG_PLATFORM_TOOLSET` from the triplet or consider it unset
1. Gather a list of all pairs of Visual Studio Instances with all toolsets available in those instances
   - This is ordered first by instance type (Stable, Prerelease, Legacy) and then by toolset version (v143, v142, v141, v140)
1. Filter the list based on the settings for `VCPKG_VISUAL_STUDIO_PATH` and `VCPKG_PLATFORM_TOOLSET`.
1. Select the best remaining option

The path should be absolute, formatted with backslashes, and have no trailing slash:

```cmake
set(VCPKG_VISUAL_STUDIO_PATH "C:\\Program Files (x86)\\Microsoft Visual Studio\\Preview\\Community")
```

### VCPKG_PLATFORM_TOOLSET

Specifies the VS-based C/C++ compiler toolchain to use.

See [`VCPKG_VISUAL_STUDIO_PATH`](#VCPKG_VISUAL_STUDIO_PATH) for the full selection algorithm.

Valid settings:

- The Visual Studio 2022 platform toolset is `v143`.
- The Visual Studio 2019 platform toolset is `v142`.
- The Visual Studio 2017 platform toolset is `v141`.
- The Visual Studio 2015 platform toolset is `v140`.

### VCPKG_PLATFORM_TOOLSET_VERSION

Specifies the detailed MSVC C/C++ compiler toolchain to use.

By default, [`VCPKG_PLATFORM_TOOLSET`] always chooses the latest installed minor version of the selected toolset.
If you need more granularity, you can use this variable.
Valid values are, for example, `14.25` or `14.27.29110`.

### VCPKG_LOAD_VCVARS_ENV

Determines whether vcpkg will search for and use an instance of Visual Studio as part of the triplet environment.

By default, this is `ON` for Windows triplets that do not specify [`VCPKG_CHAINLOAD_TOOLCHAIN_FILE`](#vcpkg_chainload_toolchain_file). For non-Windows triplets and triplets specifying `VCPKG_CHAINLOAD_TOOLCHAIN_FILE`, this defaults to `OFF`.

## Linux Variables

### VCPKG_FIXUP_ELF_RPATH

When this option is set to (true|1|on), vcpkg will add `$ORIGIN` and `$ORIGIN/<path_relative_to_lib>` to the `RUNPATH` header of executables and shared libraries. This allows packages to be relocated on Linux.

## MacOS Variables

### VCPKG_INSTALL_NAME_DIR

Sets the install name used when building macOS dynamic libraries. Default value is `@rpath`. See the CMake documentation for [CMAKE_INSTALL_NAME_DIR](https://cmake.org/cmake/help/latest/variable/CMAKE_INSTALL_NAME_DIR.html) for more information.

### VCPKG_OSX_DEPLOYMENT_TARGET

Sets the minimum macOS version for compiled binaries. This also changes what versions of the macOS platform SDK that CMake will search for. See the CMake documentation for [CMAKE_OSX_DEPLOYMENT_TARGET](https://cmake.org/cmake/help/latest/variable/CMAKE_OSX_DEPLOYMENT_TARGET.html) for more information.

### VCPKG_OSX_SYSROOT

Set the name or path of the macOS platform SDK that will be used by CMake. See the CMake documentation for [CMAKE_OSX_SYSROOT](https://cmake.org/cmake/help/latest/variable/CMAKE_OSX_SYSROOT.html) for more information.

### VCPKG_OSX_ARCHITECTURES

Set the macOS / iOS target architecture which will be used by CMake. See the CMake documentation for [CMAKE_OSX_ARCHITECTURES](https://cmake.org/cmake/help/latest/variable/CMAKE_OSX_ARCHITECTURES.html) for more information.

## Per-port customization

The CMake Macro `PORT` will be set when interpreting the triplet file and can be used to change settings (such as `VCPKG_LIBRARY_LINKAGE`) on a per-port basis.

Example:

```cmake
set(VCPKG_LIBRARY_LINKAGE static)
if(PORT MATCHES "qt5-")
    set(VCPKG_LIBRARY_LINKAGE dynamic)
endif()
```

This will build all the `qt5-*` libraries as DLLs, but every other library as a static library.

For an example in a real project, see https://github.com/Intelight/vcpkg/blob/master/triplets/x86-windows-mixed.cmake.

## Remarks

The default triplet when running any vcpkg command is `%VCPKG_DEFAULT_TRIPLET%` or a platform-specific choice if that environment variable is undefined.

- Windows: `x86-windows`
- Linux: `x64-linux`
- OSX: `x64-osx`

We recommend using a systematic naming scheme when creating new triplets. The Android toolchain naming scheme is a good source of inspiration: https://developer.android.com/ndk/guides/standalone_toolchain.html.
