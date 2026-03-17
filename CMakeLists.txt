cmake_minimum_required(VERSION 3.21)

project(NOX VERSION 0.1.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

include(GNUInstallDirs)

find_package(Qt6 6.2 REQUIRED COMPONENTS Core Gui Qml Quick QuickControls2 Concurrent)
find_package(X11 QUIET)

qt_standard_project_setup()

configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/packaging/NOX.desktop.in
    ${CMAKE_CURRENT_BINARY_DIR}/NOX.desktop
    @ONLY
)

qt_add_executable(nox
    main.cpp
    core/launcher/applicationindex.cpp
    core/search/fuzzymatcher.cpp
    core/search/searchcontroller.cpp
    core/search/searchmodel.cpp
    core/search/themeiconprovider.cpp
    core/clipboard/clipboardhistory.cpp
    core/files/fileindex.cpp
    core/plugin/pluginmanager.cpp
    core/hotkey/globalshortcut.cpp
    plugins/calculator/calculatorplugin.cpp
    plugins/system/systemplugin.cpp
)

target_include_directories(nox PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}
)

qt_add_qml_module(nox
    URI Nox
    VERSION 1.0
    QML_FILES
        ui/qml/Main.qml
        ui/qml/Theme.qml
        ui/qml/SearchBar.qml
        ui/qml/ResultsList.qml
        ui/qml/ResultItem.qml
        ui/qml/PreviewPanel.qml
    RESOURCES
        resources/fonts/Outfit-VariableFont_wght.ttf
        resources/fonts/OFL.txt
        resources/icons/nox.svg
)

target_sources(nox PRIVATE
    core/launcher/applicationindex.h
    core/search/fuzzymatcher.h
    core/search/searchcontroller.h
    core/search/searchmodel.h
    core/search/searchresult.h
    core/search/themeiconprovider.h
    core/clipboard/clipboardhistory.h
    core/files/fileindex.h
    core/plugin/searchplugin.h
    core/plugin/pluginmanager.h
    core/hotkey/globalshortcut.h
    plugins/calculator/calculatorplugin.h
    plugins/system/systemplugin.h
)

target_link_libraries(nox PRIVATE
    Qt6::Core
    Qt6::Gui
    Qt6::Qml
    Qt6::Quick
    Qt6::QuickControls2
    Qt6::Concurrent
)

if(X11_FOUND)
    target_link_libraries(nox PRIVATE X11::X11)
    target_compile_definitions(nox PRIVATE NOX_HAS_X11=1)
endif()

install(TARGETS nox
    BUNDLE DESTINATION .
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)

install(FILES
    ${CMAKE_CURRENT_BINARY_DIR}/NOX.desktop
    DESTINATION ${CMAKE_INSTALL_DATADIR}/applications
)

install(FILES
    ${CMAKE_CURRENT_SOURCE_DIR}/packaging/io.github.brsrk94.Nox.appdata.xml
    DESTINATION ${CMAKE_INSTALL_DATADIR}/metainfo
    RENAME NOX.appdata.xml
)

install(FILES
    ${CMAKE_CURRENT_SOURCE_DIR}/resources/icons/nox.svg
    DESTINATION ${CMAKE_INSTALL_DATADIR}/icons/hicolor/scalable/apps
)

set(CPACK_PACKAGE_NAME "nox")
set(CPACK_PACKAGE_VENDOR "brsrk94")
set(CPACK_PACKAGE_CONTACT "brsrk94@users.noreply.github.com")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Keyboard-first Linux launcher and command palette")
set(CPACK_PACKAGE_HOMEPAGE_URL "https://github.com/brsrk94/Nox")
set(CPACK_PACKAGE_VERSION ${PROJECT_VERSION})
set(CPACK_PACKAGE_FILE_NAME "nox-${PROJECT_VERSION}-linux-${CMAKE_SYSTEM_PROCESSOR}")

set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/resources/fonts/OFL.txt")

set(CPACK_GENERATOR "DEB")
set(CPACK_DEBIAN_FILE_NAME DEB-DEFAULT)
set(CPACK_DEBIAN_PACKAGE_SECTION "utils")
set(CPACK_DEBIAN_PACKAGE_SHLIBDEPS OFF)
set(CPACK_DEBIAN_PACKAGE_DEPENDS
    "libqt6core6 (>= 6.2), libqt6gui6 (>= 6.2), libqt6network6 (>= 6.2), libqt6opengl6 (>= 6.2), libqt6qml6 (>= 6.2), libqt6quick6 (>= 6.2), libqt6quickcontrols2-6 (>= 6.2), qml6-module-qtquick, qml6-module-qtquick-controls, qml6-module-qtquick-layouts, qml6-module-qtqml-workerscript, qt6-qpa-plugins, libx11-6, libgl1, libopengl0"
)
if(CMAKE_SYSTEM_PROCESSOR STREQUAL "x86_64")
    set(NOX_DEB_ARCHITECTURE "amd64")
elseif(CMAKE_SYSTEM_PROCESSOR STREQUAL "aarch64")
    set(NOX_DEB_ARCHITECTURE "arm64")
else()
    set(NOX_DEB_ARCHITECTURE ${CMAKE_SYSTEM_PROCESSOR})
endif()

set(CPACK_DEBIAN_PACKAGE_ARCHITECTURE ${NOX_DEB_ARCHITECTURE})

include(CPack)
