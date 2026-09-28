# Building FaCT++ on macOS / Apple Silicon (aarch64) for Protégé 5.6.9

## Requirements

* macOS on Apple Silicon (tested on an M3, Darwin 27)
* CMake >= 3.10, Apple clang with C++17
* JDK 17 (used to *build*; see the bytecode note below)
* Maven 3.9+

## Toolchain notes

If `clang` reports *"You have not agreed to the Xcode license agreements"*, either run
`sudo xcodebuild -license`, or just build against the standalone Command Line Tools:

```sh
export DEVELOPER_DIR=/Library/Developer/CommandLineTools
```

Point Maven and CMake's `find_package(JNI)` at JDK 17:

```sh
export JAVA_HOME=$(/usr/libexec/java_home -v 17)
```

## Native build

```sh
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build -j8
```

On macOS `CMAKE_OSX_ARCHITECTURES` defaults to the host architecture, so this produces
arm64 binaries. For a universal build:

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

## Versions and the Java 11 / 17 split

* OWL API is **4.5.29** — the exact version Protégé 5.6.9 ships.
* The Protégé compile-time dependency is **5.6.6**, the newest release published to
  Maven Central. It is API-compatible with the 5.6.9 desktop release and is `optional`,
  so it is not part of the plug-in's runtime closure — Protégé supplies those bundles
  through `Require-Bundle`.
* The plug-in is **built with JDK 17 but emits Java 11 bytecode**
  (`maven.compiler.release=11`). The platform-specific Protégé 5.6.9 distributions
  still bundle their own Temurin **JRE 11** (a universal arm64/x86_64 build, so it runs
  natively on Apple Silicon). Targeting 11 keeps the plug-in loadable there *and* on
  Java 17/21/25, which Protégé 5.6.9 also supports.

  If you only ever run Protégé on an external JDK 17+, build with
  `-Dmaven.compiler.release=17`.

## Installing into Protégé

Copy `FaCT++.Java/target/factplusplus-*.jar` into the Protégé `plugins/` directory
(inside `Protégé.app/Contents/plugins` on macOS) and restart Protégé.
