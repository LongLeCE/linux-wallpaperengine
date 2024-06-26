cmake_minimum_required(VERSION 3.12)
project(linux-wallpaperengine)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/CMakeModules")
set(OpenGL_GL_PREFERENCE "LEGACY")
set(DATADIR ${CMAKE_INSTALL_PREFIX}/share/${PROJECT_NAME})
set(PATCHESDIR ${DATADIR}/patches/)

if(NOT ERRORONLY)
    set(ERRORONLY 0)
endif()

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -march=native -O3 -pipe -Wno-narrowing")

# if you're developing you might find this debug option useful for shader output, although RenderDoc is encouraged
add_compile_definitions(ERRORONLY=${ERRORONLY})
add_compile_definitions(DATADIR="${DATADIR}")

find_package(X11 REQUIRED)
find_package(Xrandr REQUIRED)
find_package(OpenGL REQUIRED)
find_package(GLEW REQUIRED)
find_package(GLUT REQUIRED)
find_package(ZLIB REQUIRED)
find_package(SDL2 REQUIRED)
find_package(MPV REQUIRED)
find_package(LZ4 REQUIRED)
find_package(FFMPEG REQUIRED)
find_package(FreeImage REQUIRED)
find_package(PulseAudio REQUIRED)

include_directories(
    ${MPV_INCLUDE_DIR}
    ${X11_INCLUDE_DIR}
    ${XRANDR_INCLUDE_DIR}
    ${GLEW_INCLUDE_DIR}
    ${LZ4_INCLUDE_DIR}
    ${SDL2_INCLUDE_DIRS}
    ${FFMPEG_INCLUDE_DIR}
    ${FREEIMAGE_INCLUDE_DIR}
    ${PULSEAUDIO_INCLUDE_DIR}
    src
    ${CMAKE_SOURCE_DIR}
    include)

# try to enable wayland builds when possible
pkg_check_modules(WAYLAND_SUPPORT wayland-cursor wayland-protocols egl wayland-egl)

if(WAYLAND_SUPPORT_FOUND)
    find_program(WaylandScanner NAMES wayland-scanner)
    message(STATUS "Found WaylandScanner at ${WaylandScanner}")
    execute_process(
        COMMAND pkg-config --variable=pkgdatadir wayland-protocols
        WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
        OUTPUT_VARIABLE WAYLAND_PROTOCOLS_DIR
        OUTPUT_STRIP_TRAILING_WHITESPACE)
        message(STATUS "Found wayland-protocols at ${WAYLAND_PROTOCOLS_DIR}")

    message(STATUS "Building protocols...")
    execute_process(
        COMMAND ${WaylandScanner} client-header ${CMAKE_SOURCE_DIR}/protocols/wlr-layer-shell-unstable-v1.xml wlr-layer-shell-unstable-v1-protocol.h
        WORKING_DIRECTORY ${CMAKE_SOURCE_DIR})
    execute_process(
        COMMAND ${WaylandScanner} private-code ${CMAKE_SOURCE_DIR}/protocols/wlr-layer-shell-unstable-v1.xml wlr-layer-shell-unstable-v1-protocol.c
        WORKING_DIRECTORY ${CMAKE_SOURCE_DIR})
    execute_process(
        COMMAND ${WaylandScanner} client-header ${WAYLAND_PROTOCOLS_DIR}/stable/xdg-shell/xdg-shell.xml xdg-shell-protocol.h
        WORKING_DIRECTORY ${CMAKE_SOURCE_DIR})
    execute_process(
        COMMAND ${WaylandScanner} private-code ${WAYLAND_PROTOCOLS_DIR}/stable/xdg-shell/xdg-shell.xml xdg-shell-protocol.c
        WORKING_DIRECTORY ${CMAKE_SOURCE_DIR})

    add_compile_definitions(ENABLE_WAYLAND)
    set(WAYLAND_SOURCES
            "src/WallpaperEngine/Render/Drivers/CWaylandOpenGLDriver.h"
            "src/WallpaperEngine/Render/Drivers/CWaylandOpenGLDriver.cpp"
            "src/WallpaperEngine/Render/Drivers/Detectors/CWaylandFullScreenDetector.cpp"
            "src/WallpaperEngine/Render/Drivers/Detectors/CWaylandFullScreenDetector.h"
            "src/WallpaperEngine/Render/Drivers/Output/CWaylandOutput.cpp"
            "src/WallpaperEngine/Render/Drivers/Output/CWaylandOutput.h"
            "src/WallpaperEngine/Render/Drivers/Output/CWaylandOutputViewport.cpp"
            "src/WallpaperEngine/Render/Drivers/Output/CWaylandOutputViewport.h"
            "src/WallpaperEngine/Input/Drivers/CWaylandMouseInput.cpp"
            "src/WallpaperEngine/Input/Drivers/CWaylandMouseInput.h"
            "xdg-shell-protocol.c"
            "wlr-layer-shell-unstable-v1-protocol.c")
endif()

add_executable(
    linux-wallpaperengine
    main.cpp

    src/External/Android/fft.cpp
    src/External/Android/fft.h

    src/Steam/FileSystem/FileSystem.h
    src/Steam/FileSystem/FileSystem.cpp

    src/WallpaperEngine/Logging/CLog.cpp
    src/WallpaperEngine/Logging/CLog.h

    src/WallpaperEngine/Application/CApplicationContext.cpp
    src/WallpaperEngine/Application/CApplicationContext.h
    src/WallpaperEngine/Application/CWallpaperApplication.cpp
    src/WallpaperEngine/Application/CWallpaperApplication.h

    src/WallpaperEngine/Assets/CPackageLoadException.cpp
    src/WallpaperEngine/Assets/CPackageLoadException.h
    src/WallpaperEngine/Assets/CAssetLoadException.cpp
    src/WallpaperEngine/Assets/CAssetLoadException.h
    src/WallpaperEngine/Assets/CContainer.h
    src/WallpaperEngine/Assets/CContainer.cpp
    src/WallpaperEngine/Assets/CVirtualContainer.h
    src/WallpaperEngine/Assets/CVirtualContainer.cpp
    src/WallpaperEngine/Assets/CCombinedContainer.h
    src/WallpaperEngine/Assets/CCombinedContainer.cpp
    src/WallpaperEngine/Assets/CDirectory.h
    src/WallpaperEngine/Assets/CDirectory.cpp
    src/WallpaperEngine/Assets/CPackage.h
    src/WallpaperEngine/Assets/CPackage.cpp
    src/WallpaperEngine/Assets/ITexture.h
    src/WallpaperEngine/Assets/CTexture.h
    src/WallpaperEngine/Assets/CTexture.cpp

    src/WallpaperEngine/Core/Core.h
    src/WallpaperEngine/Core/Core.cpp

    src/WallpaperEngine/Audio/Drivers/Recorders/CPulseAudioPlaybackRecorder.cpp
    src/WallpaperEngine/Audio/Drivers/Recorders/CPulseAudioPlaybackRecorder.h
    src/WallpaperEngine/Audio/Drivers/Recorders/CPlaybackRecorder.cpp
    src/WallpaperEngine/Audio/Drivers/Recorders/CPlaybackRecorder.h

    src/WallpaperEngine/Audio/Drivers/Detectors/CPulseAudioPlayingDetector.cpp
    src/WallpaperEngine/Audio/Drivers/Detectors/CPulseAudioPlayingDetector.h
    src/WallpaperEngine/Audio/Drivers/Detectors/CAudioPlayingDetector.cpp
    src/WallpaperEngine/Audio/Drivers/Detectors/CAudioPlayingDetector.h

    src/WallpaperEngine/Audio/Drivers/CAudioDriver.cpp
    src/WallpaperEngine/Audio/Drivers/CAudioDriver.h
    src/WallpaperEngine/Audio/Drivers/CSDLAudioDriver.cpp
    src/WallpaperEngine/Audio/Drivers/CSDLAudioDriver.h
    src/WallpaperEngine/Audio/CAudioContext.cpp
    src/WallpaperEngine/Audio/CAudioContext.h
    src/WallpaperEngine/Audio/CAudioStream.cpp
    src/WallpaperEngine/Audio/CAudioStream.h

    src/WallpaperEngine/Input/CInputContext.cpp
    src/WallpaperEngine/Input/CInputContext.h
    src/WallpaperEngine/Input/CMouseInput.cpp
    src/WallpaperEngine/Input/CMouseInput.h
    src/WallpaperEngine/Input/Drivers/CGLFWMouseInput.cpp
    src/WallpaperEngine/Input/Drivers/CGLFWMouseInput.h

    src/WallpaperEngine/Render/Shaders/Variables/CShaderVariable.h
    src/WallpaperEngine/Render/Shaders/Variables/CShaderVariable.cpp
    src/WallpaperEngine/Render/Shaders/Variables/CShaderVariableFloat.h
    src/WallpaperEngine/Render/Shaders/Variables/CShaderVariableFloat.cpp
    src/WallpaperEngine/Render/Shaders/Variables/CShaderVariableInteger.h
    src/WallpaperEngine/Render/Shaders/Variables/CShaderVariableInteger.cpp
    src/WallpaperEngine/Render/Shaders/Variables/CShaderVariableVector2.h
    src/WallpaperEngine/Render/Shaders/Variables/CShaderVariableVector2.cpp
    src/WallpaperEngine/Render/Shaders/Variables/CShaderVariableVector3.h
    src/WallpaperEngine/Render/Shaders/Variables/CShaderVariableVector3.cpp
    src/WallpaperEngine/Render/Shaders/Variables/CShaderVariableVector4.h
    src/WallpaperEngine/Render/Shaders/Variables/CShaderVariableVector4.cpp

    src/WallpaperEngine/Render/Shaders/Compiler.h
    src/WallpaperEngine/Render/Shaders/Compiler.cpp

    src/WallpaperEngine/Render/Helpers/CContextAware.cpp
    src/WallpaperEngine/Render/Helpers/CContextAware.h

    src/WallpaperEngine/Render/Drivers/Detectors/CFullScreenDetector.cpp
    src/WallpaperEngine/Render/Drivers/Detectors/CFullScreenDetector.h
    src/WallpaperEngine/Render/Drivers/Detectors/CX11FullScreenDetector.cpp
    src/WallpaperEngine/Render/Drivers/Detectors/CX11FullScreenDetector.h

    src/WallpaperEngine/Render/Drivers/Output/COutput.cpp
    src/WallpaperEngine/Render/Drivers/Output/COutput.h
    src/WallpaperEngine/Render/Drivers/Output/CX11Output.cpp
    src/WallpaperEngine/Render/Drivers/Output/CX11Output.h
    src/WallpaperEngine/Render/Drivers/Output/CGLFWWindowOutput.cpp
    src/WallpaperEngine/Render/Drivers/Output/CGLFWWindowOutput.h
    src/WallpaperEngine/Render/Drivers/Output/COutputViewport.cpp
    src/WallpaperEngine/Render/Drivers/Output/COutputViewport.h
    src/WallpaperEngine/Render/Drivers/Output/CX11OutputViewport.cpp
    src/WallpaperEngine/Render/Drivers/Output/CX11OutputViewport.h
    src/WallpaperEngine/Render/Drivers/CX11OpenGLDriver.h
    src/WallpaperEngine/Render/Drivers/CX11OpenGLDriver.cpp
    src/WallpaperEngine/Render/Drivers/CVideoDriver.h
    src/WallpaperEngine/Render/Drivers/CVideoDriver.cpp
    src/WallpaperEngine/Render/CRenderContext.h
    src/WallpaperEngine/Render/CRenderContext.cpp
    src/WallpaperEngine/Render/CTextureCache.h
    src/WallpaperEngine/Render/CTextureCache.cpp

    src/WallpaperEngine/Render/Helpers/CContextAware.cpp
    src/WallpaperEngine/Render/Helpers/CContextAware.h

    src/WallpaperEngine/Render/CWallpaper.h
    src/WallpaperEngine/Render/CWallpaper.cpp
    src/WallpaperEngine/Render/CWallpaperState.h
    src/WallpaperEngine/Render/CWallpaperState.cpp
    src/WallpaperEngine/Render/CScene.h
    src/WallpaperEngine/Render/CScene.cpp
    src/WallpaperEngine/Render/CVideo.h
    src/WallpaperEngine/Render/CVideo.cpp
    src/WallpaperEngine/Render/CCamera.h
    src/WallpaperEngine/Render/CCamera.cpp
    src/WallpaperEngine/Render/CObject.h
    src/WallpaperEngine/Render/CObject.cpp

    src/WallpaperEngine/Render/Objects/CImage.h
    src/WallpaperEngine/Render/Objects/CImage.cpp
    src/WallpaperEngine/Render/Objects/CSound.h
    src/WallpaperEngine/Render/Objects/CSound.cpp
    src/WallpaperEngine/Render/Objects/CEffect.h
    src/WallpaperEngine/Render/Objects/CEffect.cpp

    src/WallpaperEngine/Render/CFBO.h
    src/WallpaperEngine/Render/CFBO.cpp
    src/WallpaperEngine/Render/Objects/Effects/CPass.h
    src/WallpaperEngine/Render/Objects/Effects/CPass.cpp
    src/WallpaperEngine/Render/Objects/Effects/CMaterial.h
    src/WallpaperEngine/Render/Objects/Effects/CMaterial.cpp

    src/WallpaperEngine/FileSystem/FileSystem.cpp
    src/WallpaperEngine/FileSystem/FileSystem.h

    src/WallpaperEngine/Core/UserSettings/CUserSettingValue.cpp
    src/WallpaperEngine/Core/UserSettings/CUserSettingValue.h
    src/WallpaperEngine/Core/UserSettings/CUserSettingBoolean.cpp
    src/WallpaperEngine/Core/UserSettings/CUserSettingBoolean.h
    src/WallpaperEngine/Core/UserSettings/CUserSettingVector3.cpp
    src/WallpaperEngine/Core/UserSettings/CUserSettingVector3.h
    src/WallpaperEngine/Core/UserSettings/CUserSettingFloat.cpp
    src/WallpaperEngine/Core/UserSettings/CUserSettingFloat.h

    src/WallpaperEngine/Core/CProject.cpp
    src/WallpaperEngine/Core/CProject.h
    src/WallpaperEngine/Core/CWallpaper.cpp
    src/WallpaperEngine/Core/CWallpaper.h
    src/WallpaperEngine/Core/CScene.cpp
    src/WallpaperEngine/Core/CScene.h
    src/WallpaperEngine/Core/CVideo.cpp
    src/WallpaperEngine/Core/CVideo.h
    src/WallpaperEngine/Core/CObject.cpp
    src/WallpaperEngine/Core/CObject.h

    src/WallpaperEngine/Core/Projects/CProperty.h
    src/WallpaperEngine/Core/Projects/CProperty.cpp
    src/WallpaperEngine/Core/Projects/CPropertyColor.h
    src/WallpaperEngine/Core/Projects/CPropertyColor.cpp
    src/WallpaperEngine/Core/Projects/CPropertyBoolean.h
    src/WallpaperEngine/Core/Projects/CPropertyBoolean.cpp
    src/WallpaperEngine/Core/Projects/CPropertySlider.h
    src/WallpaperEngine/Core/Projects/CPropertySlider.cpp
    src/WallpaperEngine/Core/Projects/CPropertyCombo.h
    src/WallpaperEngine/Core/Projects/CPropertyCombo.cpp
    src/WallpaperEngine/Core/Projects/CPropertyText.h
    src/WallpaperEngine/Core/Projects/CPropertyText.cpp

    src/WallpaperEngine/Core/Scenes/CCamera.cpp
    src/WallpaperEngine/Core/Scenes/CCamera.h
    src/WallpaperEngine/Core/Scenes/CProjection.cpp
    src/WallpaperEngine/Core/Scenes/CProjection.h

    src/WallpaperEngine/Core/Objects/CImage.cpp
    src/WallpaperEngine/Core/Objects/CImage.h
    src/WallpaperEngine/Core/Objects/CSound.cpp
    src/WallpaperEngine/Core/Objects/CSound.h
    src/WallpaperEngine/Core/Objects/CEffect.cpp
    src/WallpaperEngine/Core/Objects/CEffect.h
    src/WallpaperEngine/Core/Objects/CParticle.cpp
    src/WallpaperEngine/Core/Objects/CParticle.h

    src/WallpaperEngine/Core/Objects/Effects/CFBO.h
    src/WallpaperEngine/Core/Objects/Effects/CFBO.cpp
    src/WallpaperEngine/Core/Objects/Effects/CBind.h
    src/WallpaperEngine/Core/Objects/Effects/CBind.cpp

    src/WallpaperEngine/Core/Objects/Effects/Constants/CShaderConstant.h
    src/WallpaperEngine/Core/Objects/Effects/Constants/CShaderConstant.cpp
    src/WallpaperEngine/Core/Objects/Effects/Constants/CShaderConstantFloat.h
    src/WallpaperEngine/Core/Objects/Effects/Constants/CShaderConstantFloat.cpp
    src/WallpaperEngine/Core/Objects/Effects/Constants/CShaderConstantVector3.h
    src/WallpaperEngine/Core/Objects/Effects/Constants/CShaderConstantVector3.cpp
    src/WallpaperEngine/Core/Objects/Effects/Constants/CShaderConstantVector4.h
    src/WallpaperEngine/Core/Objects/Effects/Constants/CShaderConstantVector4.cpp
    src/WallpaperEngine/Core/Objects/Effects/Constants/CShaderConstantInteger.h
    src/WallpaperEngine/Core/Objects/Effects/Constants/CShaderConstantInteger.cpp

    src/WallpaperEngine/Core/Objects/Particles/CControlPoint.cpp
    src/WallpaperEngine/Core/Objects/Particles/CControlPoint.h
    src/WallpaperEngine/Core/Objects/Particles/CEmitter.cpp
    src/WallpaperEngine/Core/Objects/Particles/CEmitter.h
    src/WallpaperEngine/Core/Objects/Particles/CInitializer.cpp
    src/WallpaperEngine/Core/Objects/Particles/CInitializer.h

    src/WallpaperEngine/Core/Objects/Particles/Initializers/CLifeTimeRandom.cpp
    src/WallpaperEngine/Core/Objects/Particles/Initializers/CLifeTimeRandom.h
    src/WallpaperEngine/Core/Objects/Particles/Initializers/CSizeRandom.cpp
    src/WallpaperEngine/Core/Objects/Particles/Initializers/CSizeRandom.h
    src/WallpaperEngine/Core/Objects/Particles/Initializers/CRotationRandom.cpp
    src/WallpaperEngine/Core/Objects/Particles/Initializers/CRotationRandom.h
    src/WallpaperEngine/Core/Objects/Particles/Initializers/CVelocityRandom.cpp
    src/WallpaperEngine/Core/Objects/Particles/Initializers/CVelocityRandom.h
    src/WallpaperEngine/Core/Objects/Particles/Initializers/CColorRandom.cpp
    src/WallpaperEngine/Core/Objects/Particles/Initializers/CColorRandom.h
    src/WallpaperEngine/Core/Objects/Particles/Initializers/CAlphaRandom.cpp
    src/WallpaperEngine/Core/Objects/Particles/Initializers/CAlphaRandom.h
    src/WallpaperEngine/Core/Objects/Particles/Initializers/CAngularVelocityRandom.cpp
    src/WallpaperEngine/Core/Objects/Particles/Initializers/CAngularVelocityRandom.h
    src/WallpaperEngine/Core/Objects/Particles/Initializers/CTurbulentVelocityRandom.cpp
    src/WallpaperEngine/Core/Objects/Particles/Initializers/CTurbulentVelocityRandom.h

    src/WallpaperEngine/Core/Objects/Images/CMaterial.cpp
    src/WallpaperEngine/Core/Objects/Images/CMaterial.h

    src/WallpaperEngine/Core/Objects/Images/Materials/CPass.cpp
    src/WallpaperEngine/Core/Objects/Images/Materials/CPass.h

    ${WAYLAND_SOURCES}
)

target_link_libraries(linux-wallpaperengine
    ${X11_LIBRARIES}
    ${Xrandr_LIBRARIES}
    ${X11_Xxf86vm_LIB}
    ${OPENGL_LIBRARIES}
    ${GLEW_LIBRARIES}
    ${GLUT_LIBRARIES}
    ${ZLIB_LIBRARIES}
    ${LZ4_LIBRARY}
    ${SDL2_LIBRARIES}
    ${FFMPEG_LIBRARIES}
    ${FREEIMAGE_LIBRARIES}
    ${MPV_LIBRARY}
    ${PULSEAUDIO_LIBRARY}
    glfw)

if (WAYLAND_SUPPORT_FOUND)
    target_link_libraries(linux-wallpaperengine
        pthread
        wayland-cursor
        wayland-client
        wayland-egl
        ${OPENGL_egl_LIBRARY})
endif()

file(CREATE_LINK linux-wallpaperengine wallengine SYMBOLIC)

set(CMAKE_REQUIRED_LIBRARIES ${X11_LIBRARIES})

# make sure some of the X11 functions we'll use are available
check_function_exists(XSetIOErrorExitHandler HAVE_XSETIOERROREXITHANDLER)

if(HAVE_XSETIOERROREXITHANDLER)
    add_compile_definitions(HAVE_XSETIOERROREXITHANDLER=1)
endif()

# set some install parameters if not in debug mode
install(TARGETS linux-wallpaperengine)
install(DIRECTORY share/ DESTINATION share/${PROJECT_NAME})
