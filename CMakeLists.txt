cmake_minimum_required(VERSION 3.16)
project(gstqml6 VERSION 1.0 LANGUAGES C CXX)

set(CMAKE_CXX_STANDARD 17)

set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)

if (ANDROID)
    set(CMAKE_POSITION_INDEPENDENT_CODE ON)

    if(DEFINED ENV{GSTREAMER_ROOT_ANDROID})
        set(GSTREAMER_ROOT_ANDROID $ENV{GSTREAMER_ROOT_ANDROID})
    endif()

    if(NOT DEFINED GSTREAMER_ROOT_ANDROID)
        message(FATAL_ERROR "GSTREAMER_ROOT_ANDROID is not defined!")
    endif()

    if(ANDROID_ABI STREQUAL "armeabi")
        set(GSTREAMER_ROOT_ABI "${GSTREAMER_ROOT_ANDROID}/arm")
    elseif(ANDROID_ABI STREQUAL "armeabi-v7a")
        set(GSTREAMER_ROOT_ABI "${GSTREAMER_ROOT_ANDROID}/armv7")
    elseif(ANDROID_ABI STREQUAL "arm64-v8a")
        set(GSTREAMER_ROOT_ABI "${GSTREAMER_ROOT_ANDROID}/arm64")
    elseif(ANDROID_ABI STREQUAL "x86")
        set(GSTREAMER_ROOT_ABI "${GSTREAMER_ROOT_ANDROID}/x86")
    elseif(ANDROID_ABI STREQUAL "x86_64")
        set(GSTREAMER_ROOT_ABI "${GSTREAMER_ROOT_ANDROID}/x86_64")
    else()
        message(FATAL_ERROR "Target arch ABI not supported: ${ANDROID_ABI}")
    endif()
    list(APPEND CMAKE_MODULE_PATH "${GSTREAMER_ROOT_ABI}/share/cmake")

    set(GSTREAMER_NDK_BUILD_PATH  "${GSTREAMER_ROOT_ABI}/share/gst-android/ndk-build/")
    include("${GSTREAMER_NDK_BUILD_PATH}/plugins.cmake")
    set(GSTREAMER_PLUGINS         ${GSTREAMER_PLUGINS_CORE})
    set(GStreamer_EXTRA_DEPS gstreamer-video-1.0 gstreamer-gl-1.0 gstreamer-gl-egl-1.0)

    find_package(GStreamerMobile COMPONENTS ${GSTREAMER_PLUGINS} fonts REQUIRED)
    include_directories(${GStreamer_INCLUDE_DIR})

    set(target_libs GStreamer::mobile ${ANDROID_LIB})
    set(target_defs HAVE_QT_ANDROID)
else()
    find_package(PkgConfig REQUIRED)
    pkg_search_module(gstreamer REQUIRED IMPORTED_TARGET gstreamer-1.0)
    pkg_search_module(gstreamer-gl REQUIRED IMPORTED_TARGET gstreamer-gl-1.0)
    pkg_search_module(gstreamer-gl-egl REQUIRED IMPORTED_TARGET gstreamer-gl-egl-1.0)
    pkg_search_module(gstreamer-gl-wayland IMPORTED_TARGET gstreamer-gl-wayland-1.0)
    pkg_search_module(gstreamer-gl-x11 IMPORTED_TARGET gstreamer-gl-x11-1.0)
    pkg_search_module(gstreamer-video REQUIRED IMPORTED_TARGET gstreamer-video-1.0)

    set(target_libs
      PkgConfig::gstreamer
      PkgConfig::gstreamer-gl
      PkgConfig::gstreamer-gl-egl
      PkgConfig::gstreamer-gl-wayland
      PkgConfig::gstreamer-gl-x11
      PkgConfig::gstreamer-video)
    set(target_defs
      GST_GL_HAVE_PLATFORM_EGL GST_GL_HAVE_PLATFORM_GLX GST_GL_HAVE_WINDOW_X11 HAVE_QT_X11)
endif()


find_package(Qt6 REQUIRED COMPONENTS Core Gui Qml Quick ShaderTools)
# OPTIONAL_COMPONENTS WaylandCompositor

# Source and header lists
set(qt6_sources
  gstplugin.cc
  gstqt6element.cc
  gstqsg6material.cc
  gstqt6glutility.cc
  gstqml6glsink.cc
  gstqml6glsrc.cc
  gstqml6glmixer.cc
  gstqml6gloverlay.cc
  qt6glitem.cc
  qt6glwindow.cc
  qt6glrenderer.cc
)

set(qt6_headers
  gstqml6glsink.h
  gstqt6elements.h
  qt6glrenderer.h
  gstqt6glutility.h
  qt6glitem.h
  gstqt6gl.h
  gstqsg6material.h
  qt6glwindow.h
  gstqml6glsrc.h
  gstqml6glmixer.h
  gstqml6gloverlay.h
)

set(moc_headers
  qt6glitem.h
  qt6glwindow.h
  qt6glrenderer.h
)

qt_wrap_cpp(moc_sources ${moc_headers})

qt_standard_project_setup(REQUIRES 6.5)

# Shader compilation
#find_program(QSB_EXECUTABLE NAMES qsb qsb-qt6)
#find_program(QSB_WRAPPER NAMES qsb-wrapper.py)

#if(NOT QSB_EXECUTABLE)
#  message(FATAL_ERROR "qsb not found")
#endif()

set(shader_sources
  vertex.vert
  RGBA.frag
  YUV_BIPLANAR.frag
  YUV_TRIPLANAR.frag
)

#set(compiled_shaders "")
#foreach(shader ${shader_sources})
#  set(qsb_out "${shader}.qsb")
#  add_custom_command(
#    OUTPUT ${qsb_out}
#    COMMAND ${QSB_EXECUTABLE} --glsl=100es,120,330 --batchable --output ${qsb_out} ${shader}
#    DEPENDS ${shader}
#  )
#  list(APPEND compiled_shaders ${qsb_out})
#endforeach()

# Handle special GLES variants
set(RGBA_100es "RGBA_gles.frag")
set(RGBA_qsb "RGBA.frag.qsb")
set(RGBA_external "RGBA.frag.qsb.external")
#add_custom_command(
#  OUTPUT ${RGBA_external}
#  COMMAND ${QSB_WRAPPER} ${QSB_EXECUTABLE} ${RGBA_external} ${RGBA_100es} ${RGBA_qsb}
#  DEPENDS ${RGBA_100es} ${RGBA_qsb}
#)
list(APPEND compiled_shaders ${RGBA_external})

# Define target
add_library(gstqml6 STATIC
  ${qt6_sources}
  ${moc_sources}
)

# Qt resources
#qt6_add_resources(gstqml6 resources.qrc)

qt6_add_shaders(gstqml6 "gstqml6_shaders"
    GLSL "100 es,120,330"
    BATCHABLE
    PREFIX
        "/org/freedesktop/gstreamer/qml6"
    FILES
        ${shader_sources}
)

target_include_directories(gstqml6 PUBLIC ${PROJECT_SOURCE_DIR} PRIVATE
  ${Qt6Gui_PRIVATE_INCLUDE_DIRS}
  #${gstreamer_INCLUDE_DIRS}
  #${gstreamer_INCLUDE_DIRS} ${gstreamer-gl_INCLUDE_DIRS} ${gstreamer-video_INCLUDE_DIRS}
)

#if (Qt6GuiPrivate_FOUND)
#    target_compile_definitions(gstqml6 PRIVATE
#      HAVE_QT_QPA_HEADER
#      QT_QPA_HEADER="<QtGui/qpa/qplatformnativeinterface.h>"
#    )
#endif()

target_compile_definitions(gstqml6 PRIVATE
  HAVE_QT_EGLFS ${target_defs}
)

target_link_libraries(gstqml6 PRIVATE
  Qt6::Core Qt6::Gui Qt6::GuiPrivate Qt6::Qml Qt6::Quick
  # Qt6::WaylandCompositor
  ${target_libs}
)

install(TARGETS gstqml6 DESTINATION lib)
