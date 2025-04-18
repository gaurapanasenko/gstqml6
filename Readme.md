# gstqml6 (CMake version)

This project provides a CMake-based build system for the `gstqml6` plugin from the GStreamer `gst-plugins-good` module. The original source is taken from:

https://gitlab.freedesktop.org/gstreamer/gstreamer/-/tree/1.26/subprojects/gst-plugins-good/ext/qt6

Branch: `1.26`

The goal is to build the `gstqml6` plugin using prebuilt GStreamer and Qt binaries, without Cerbero or Meson.

## Build Instructions (Linux)

1. Copy `enviroment.sh.example` to `enviroment.sh`
2. Edit `enviroment.sh` with correct paths:

```sh
export JAVA_HOME=/usr/lib64/jvm/java-17-openjdk
export ANDROID_HOME=/path/to/android
export QT_HOME=/path/to/qt
export QT_VERSION=6.8.3
export COPY_TO_GSTREAMER_ROOT=true
export GSTREAMER_ROOT_ANDROID=/path/to/gstreamer
```

- `JAVA_HOME`: path to Java 17
- `ANDROID_HOME`: path to Android SDK
- `QT_HOME`: path to Qt for Android (installed with Qt online/offline installer)
- `COPY_TO_GSTREAMER_ROOT`: if true, copies resulting `gstqml6` to the GStreamer prebuilt path
- `GSTREAMER_ROOT_ANDROID`: path to GStreamer prebuilt directory

3. Run the build script:
```sh
./build.sh
```

## Usage

To use `gstqml6` in your own project, refer to the `CMakeLists.txt` in this repository. Ensure that you include `qml6` in GStreamer components:

```cmake
find_package(GStreamerMobile COMPONENTS ${GSTREAMER_PLUGINS} fonts qml6 REQUIRED)
```

After adding Qt to your project, link Qt Quick to GStreamer:

```cmake
target_link_libraries(GStreamerMobile PRIVATE Qt6::Quick)
```

Also, you must initialize the `gstqml6` shader resources in your main function after creating the `QGuiApplication`:

```cpp
QGuiApplication app(argc, argv);
Q_INIT_RESOURCE(gstqml6_shaders);
```

GStreamerMobile CMake script strips Qt resources, so the easiest way to include them is to link the `gstqml6` target directly to your application:

```cmake
target_link_libraries(qtapp PRIVATE ${_gst_gstqml6})
```
