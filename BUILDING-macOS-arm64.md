# Building FaCT++ on macOS / Apple Silicon (aarch64) for Protégé 5.6.9

## Requirements

* macOS on Apple Silicon (tested on an M3, Darwin 27)
* CMake >= 3.10, Apple clang with C++17
* JDK 17 or newer to *build* (CI uses 21; tested up to 25). See the bytecode note below.
* Maven 3.9+

## Toolchain notes

If `clang` reports *"You have not agreed to the Xcode license agreements"*, either run
`sudo xcodebuild -license`, or just build against the standalone Command Line Tools:

```sh
export DEVELOPER_DIR=/Library/Developer/CommandLineTools
```

Point Maven and CMake's `find_package(JNI)` at a JDK, e.g. 21:

```sh
export JAVA_HOME=$(/usr/libexec/java_home -v 21)
```

## Native build

```sh
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build -j8
```

On macOS `CMAKE_OSX_ARCHITECTURES` defaults to the host architecture, so this produces
arm64 binaries, and `CMAKE_OSX_DEPLOYMENT_TARGET` defaults to 11.0 so they also load on
older macOS releases (check with `otool -l <file> | grep minos`). For a universal build:

```sh
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release -DCMAKE_OSX_ARCHITECTURES="arm64;x86_64"
```

Artifacts:

| Target | Output |
| --- | --- |
| Kernel | `build/Kernel/libKernel.a` |
| CLI | `build/FaCT++/FaCT++` |
| JNI (Protégé) | `build/FaCT++.JNI/libFaCTPlusPlusJNI.jnilib` |
| C API | `build/FaCT++.C/libfact.dylib` |

Verify with `lipo -archs <file>`.

## Java / Protégé plug-in

Copy the freshly built JNI library into the plug-in resources, then build:

```sh
cp build/FaCT++.JNI/libFaCTPlusPlusJNI.jnilib \
   FaCT++.Java/src/main/resources/lib/native/arm64/
mvn -f FaCT++.Java clean install
```

The `native-macos-arm64` Maven profile activates automatically on Apple Silicon and
points the tests' `java.library.path` at `lib/native/arm64`.

## Versions: build JDK vs. bytecode level

* OWL API is **4.5.29** — the exact version Protégé 5.6.9 ships.
* The Protégé compile-time dependency is **5.6.6**, the newest release published to
  Maven Central. It is API-compatible with the 5.6.9 desktop release and is `optional`,
  so it is not part of the plug-in's runtime closure — Protégé supplies those bundles
  through `Require-Bundle`.
* The plug-in is **built with a current JDK but emits Java 11 bytecode**
  (`maven.compiler.release=11`). The platform-specific Protégé 5.6.9 distributions
  still bundle their own Temurin **JRE 11** (a universal arm64/x86_64 build, so it runs
  natively on Apple Silicon). Targeting 11 keeps the plug-in loadable there *and* on
  Java 17/21/25, which Protégé 5.6.9 also supports.

  If you only ever run Protégé on an external JDK 17+, build with
  `-Dmaven.compiler.release=17`.

## Installing into Protégé

Copy `FaCT++.Java/target/factplusplus-*.jar` into the Protégé `plugins/` directory
(inside `Protégé.app/Contents/plugins` on macOS) and restart Protégé.

## Other platforms: CI

`.github/workflows/build.yml` builds the JNI library for every shipped platform on each
push to `master` and each pull request, runs the Java tests against it on that platform,
and assembles the plug-in jar. The plug-in is built with JDK 21 (`BUILD_JAVA` in the
workflow) and the tests run on Java 11, 17, 21 and 25 on every platform:

| Platform | Built on | Resource path |
| --- | --- | --- |
| Linux x86_64 | manylinux_2_28 container | `lib/native/64bit/libFaCTPlusPlusJNI.so` |
| Linux aarch64 | manylinux_2_28 container | `lib/native/arm64/libFaCTPlusPlusJNI.so` |
| Windows x64 | MSVC, static runtime | `lib/native/64bit/FaCTPlusPlusJNI.dll` |
| macOS arm64 | macOS 14 runner | `lib/native/arm64/libFaCTPlusPlusJNI.jnilib` |

The Linux libraries link the C++ runtime statically and need only glibc; the Windows DLL
needs no Visual C++ Redistributable. The run's `native-libraries` artifact holds all of
them in the resources layout, ready to commit.

There is no Intel macOS library: on an Intel Mac the plug-in bundle does not resolve and
FaCT++ is simply absent from Protégé's reasoner list.

32-bit binaries are no longer shipped: Protégé 5.6.9 needs Java 11+, which has no 32-bit
builds for these platforms.

## Running Protégé on Java 24+

JDK 24 and newer warn when a JNI library is loaded (JEP 472), and a future JDK will
refuse unless native access is enabled. When running Protégé on such a JDK, add this to
`~/.Protege/conf/jvm.conf`:

```
append=--enable-native-access=ALL-UNNAMED
```

Only do this when Protégé runs on Java 17 or newer: Java 11, including the JRE bundled
with the platform-specific Protégé downloads, rejects the option and will not start.
