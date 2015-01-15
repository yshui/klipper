project(plasma-workspace)
set(PROJECT_VERSION "5.2.90")
set(PROJECT_VERSION_MAJOR 5)

cmake_minimum_required(VERSION 2.8.12 FATAL_ERROR)

set(QT_MIN_VERSION "5.3.0")
set(KF5_MIN_VERSION "5.4.0")
set(INSTALL_SDDM_THEME TRUE)
find_package(Qt5 ${QT_MIN_VERSION} CONFIG REQUIRED COMPONENTS Widgets Quick QuickWidgets Concurrent Test Script Network)
find_package(ECM 0.0.11 REQUIRED NO_MODULE)
set(CMAKE_MODULE_PATH ${ECM_MODULE_PATH} ${ECM_KDE_MODULE_DIR})

find_package(KF5 ${KF5_MIN_VERSION} REQUIRED COMPONENTS KDELibs4Support)

#find_package(KF5TextEditor)
#find_package(KWinDBusInterface CONFIG REQUIRED)
find_package(KF5WebKit)

include(KDEInstallDirs)
include(KDECMakeSettings)
include(KDECompilerSettings)
include(ECMPackageConfigHelpers)
include(ECMMarkNonGuiExecutable)
include(CMakePackageConfigHelpers)
include(WriteBasicConfigVersionFile)
include(CheckIncludeFiles)
include(FeatureSummary)

find_package(Qt5WebKitWidgets ${QT_MIN_VERSION} CONFIG)

set_package_properties(Qt5WebKitWidgets PROPERTIES
                       URL "git://gitorious.org/qt/qtwebkit.git"
                       DESCRIPTION "Qt Webkit module (web browsing engine)"
                       TYPE OPTIONAL
                       PURPOSE "Used by the HTML-based GUI ksysguard library, therefore required for systemmonitor"
                      )

find_package(ZLIB)
set_package_properties(ZLIB PROPERTIES DESCRIPTION "Support for gzip compressed files and data streams"
                       URL "http://www.zlib.net"
                       TYPE REQUIRED
                      )

find_package(dbusmenu-qt5 CONFIG)
set_package_properties(dbusmenu-qt5 PROPERTIES DESCRIPTION "Support for notification area menus via the DBusMenu protocol"
                       URL "https://launchpad.net/libdbusmenu-qt"
                       TYPE OPTIONAL
                      )

find_package(X11)
set_package_properties(X11 PROPERTIES DESCRIPTION "X11 libraries"
                        URL "http://www.x.org"
                        TYPE OPTIONAL
                        PURPOSE "Required for building the X11 based workspace")

if(X11_FOUND)
  find_package(XCB MODULE REQUIRED COMPONENTS XCB KEYSYMS)
  set_package_properties(XCB PROPERTIES TYPE REQUIRED)
  if(NOT X11_SM_FOUND)
    message(FATAL_ERROR "\nThe X11 Session Management (SM) development package could not be found.\nPlease install libSM.\n")
  endif(NOT X11_SM_FOUND)

  find_package(Qt5 ${QT_MIN_VERSION} CONFIG REQUIRED COMPONENTS X11Extras)
endif()

if(X11_FOUND AND XCB_XCB_FOUND)
  set(HAVE_X11 1)
endif()

include(ConfigureChecks.cmake)

include_directories("${CMAKE_CURRENT_BINARY_DIR}")

#configure_file(config-workspace.h.cmake ${CMAKE_CURRENT_BINARY_DIR}/config-workspace.h)
#configure_file(config-unix.h.cmake ${CMAKE_CURRENT_BINARY_DIR}/config-unix.h )
configure_file(config-X11.h.cmake ${CMAKE_CURRENT_BINARY_DIR}/config-X11.h)

#configure_file(plasma.desktop.cmake ${CMAKE_CURRENT_BINARY_DIR}/plasma.desktop)

set(KLIPPER_VERSION_STRING ${PROJECT_VERSION})
add_definitions(-DTRANSLATION_DOMAIN=\"klipper\")

set(libklipper_common_SRCS
    klipper.cpp
    urlgrabber.cpp
    configdialog.cpp
    history.cpp
    historyitem.cpp
    historymodel.cpp
    historystringitem.cpp
    klipperpopup.cpp
    popupproxy.cpp
    historyimageitem.cpp
    historyurlitem.cpp
    actionstreewidget.cpp
    editactiondialog.cpp
    clipcommandprocess.cpp
)

find_package(Prison "1.2.0" CONFIG)
set_package_properties(Prison PROPERTIES DESCRIPTION "Prison library"
                       URL "http://projects.kde.org/prison"
                       TYPE OPTIONAL
                       PURPOSE "Needed to create mobile barcodes from clipboard data (no stable release currently)"
                      )
if (PRISON_FOUND)
    include_directories(${PRISON_INCLUDE_DIR})
endif ()

set(HAVE_PRISON ${PRISON_FOUND})
configure_file(config-klipper.h.cmake ${CMAKE_CURRENT_BINARY_DIR}/config-klipper.h )

kde4_add_app_icon(libklipper_common_SRCS "${KDE4_ICON_INSTALL_DIR}/oxygen/*/apps/klipper.png")
ki18n_wrap_ui(libklipper_common_SRCS generalconfig.ui actionsconfig.ui editactiondialog.ui)
kconfig_add_kcfg_files(libklipper_common_SRCS klippersettings.kcfgc)

set(klipper_KDEINIT_SRCS ${libklipper_common_SRCS} main.cpp tray.cpp)


kf5_add_kdeinit_executable(klipper ${klipper_KDEINIT_SRCS})

target_link_libraries(kdeinit_klipper
    KF5::Completion # klineedit - port away?
    KF5::ConfigGui
    KF5::CoreAddons
    KF5::DBusAddons
    KF5::GlobalAccel
    KF5::IconThemes
    KF5::Notifications
    KF5::Service
    KF5::TextWidgets
    KF5::WindowSystem
    KF5::WidgetsAddons
    KF5::XmlGui
    ${ZLIB_LIBRARY}
)
if (X11_FOUND)
  target_link_libraries(kdeinit_klipper XCB::XCB Qt5::X11Extras)
endif()
if (PRISON_FOUND)
  target_link_libraries(kdeinit_klipper ${PRISON_LIBRARIES})
endif ()

install(TARGETS kdeinit_klipper ${INSTALL_TARGETS_DEFAULT_ARGS})
install(TARGETS klipper         ${INSTALL_TARGETS_DEFAULT_ARGS})

install(PROGRAMS org.kde.klipper.desktop DESTINATION ${XDG_APPS_INSTALL_DIR})
install(PROGRAMS org.kde.klipper.desktop DESTINATION ${AUTOSTART_INSTALL_DIR})

add_subdirectory(autotests)
