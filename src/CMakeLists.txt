if(TARGET Qt6::Core OR TARGET Qt6::Widgets)
    set(FILE_SORT_QT_MAJOR 6)
elseif(TARGET Qt5::Core OR TARGET Qt5::Widgets)
    set(FILE_SORT_QT_MAJOR 5)
else()
    find_package(QT NAMES Qt6 Qt5 REQUIRED COMPONENTS Core)
    set(FILE_SORT_QT_MAJOR ${QT_VERSION_MAJOR})
endif()

set(FILE_SORT_QT_COMPONENTS
    Widgets
    Concurrent
    Sql
    LinguistTools
)

if(FILE_SORT_QT_MAJOR EQUAL 5)
    list(APPEND FILE_SORT_QT_COMPONENTS Script)
    if(NOT TARGET Qt5::Widgets)
        find_package(Qt5 REQUIRED COMPONENTS ${FILE_SORT_QT_COMPONENTS})
    endif()
elseif(FILE_SORT_QT_MAJOR GREATER_EQUAL 6)
    list(APPEND FILE_SORT_QT_COMPONENTS Qml)
    find_package(Qt${FILE_SORT_QT_MAJOR} REQUIRED COMPONENTS ${FILE_SORT_QT_COMPONENTS})

    if(NOT TARGET Qt5::Widgets)
        add_library(Qt5::Widgets INTERFACE IMPORTED)
        set_property(TARGET Qt5::Widgets PROPERTY
            INTERFACE_LINK_LIBRARIES Qt${FILE_SORT_QT_MAJOR}::Widgets)
    endif()
endif()

file(STRINGS "${CMAKE_CURRENT_LIST_DIR}/../release_version.txt" X_PROJECT_VERSION)
project(FileSort VERSION ${X_PROJECT_VERSION} LANGUAGES CXX)

if(NOT PROJECT_VERSION_PATCH)
    set(PROJECT_VERSION_PATCH 0)
endif()
if(NOT PROJECT_VERSION_TWEAK)
    set(PROJECT_VERSION_TWEAK 0)
endif()

set(X_COMPANYNAME       "ntinfo")
set(X_PROJECTNAME       "FileSort")
set(X_MAINTAINER        "horsicq@gmail.com")
set(X_DESCRIPTION       "FileSort utility")
set(X_HOMEPAGE          "http://ntinfo.biz")
set(X_ORIGINAL_FILENAME "FileSort")

if(NOT DEFINED MACOSX_BUNDLE_GUI_IDENTIFIER OR "${MACOSX_BUNDLE_GUI_IDENTIFIER}" STREQUAL "")
    set(MACOSX_BUNDLE_GUI_IDENTIFIER "com.ntinfo.FileSort")
endif()
set(BUNDLE_ID_OPTION "${MACOSX_BUNDLE_GUI_IDENTIFIER}")

set(X_RESOURCE_DIR                "${CMAKE_CURRENT_LIST_DIR}/../res")
set(X_WINDOWS_ICON                "${X_RESOURCE_DIR}/windows/main.ico")
set(X_WINDOWS_RESOURCE_RC_IN      "${X_RESOURCE_DIR}/windows/resource.rc.in")
set(X_WINDOWS_RESOURCE_ICON_RC_IN "${X_RESOURCE_DIR}/windows/resource_icon.rc.in")
set(X_WINDOWS_RESOURCE_RC         "${X_RESOURCE_DIR}/windows/resource.rc")
set(X_WINDOWS_RESOURCE_ICON_RC    "${X_RESOURCE_DIR}/windows/resource_icon.rc")
set(X_MACOS_ICON                  "${X_RESOURCE_DIR}/macos/main.icns")
set(X_MACOS_INFO_PLIST_IN         "${X_RESOURCE_DIR}/macos/Info.plist.in")
set(X_MACOS_INFO_PLIST            "${X_RESOURCE_DIR}/macos/Info.plist")

set(FILE_SORT_DEPLOY_INIT "${CMAKE_CURRENT_LIST_DIR}/../dep/build_tools/cmake/deploy_init.cmake")
if(EXISTS "${FILE_SORT_DEPLOY_INIT}"
   AND EXISTS "${X_WINDOWS_RESOURCE_RC_IN}"
   AND EXISTS "${X_WINDOWS_RESOURCE_ICON_RC_IN}"
   AND EXISTS "${X_MACOS_ICON}"
   AND EXISTS "${X_MACOS_INFO_PLIST_IN}")
    include("${FILE_SORT_DEPLOY_INIT}")
else()
    message(STATUS "Skipping deploy_init.cmake — packaging templates missing")
endif()

set(FILE_SORT_TRANSLATION_HELPER "${CMAKE_CURRENT_LIST_DIR}/../dep/build_tools/cmake/deploy_translation.cmake")
if(EXISTS "${FILE_SORT_TRANSLATION_HELPER}")
    include("${FILE_SORT_TRANSLATION_HELPER}")

    set(FILE_SORT_TS_FILES
        "${CMAKE_CURRENT_LIST_DIR}/../translations/FileSort_ar_AR.ts"
        "${CMAKE_CURRENT_LIST_DIR}/../translations/FileSort_bn_BN.ts"
        "${CMAKE_CURRENT_LIST_DIR}/../translations/FileSort_de_DE.ts"
        "${CMAKE_CURRENT_LIST_DIR}/../translations/FileSort_es_ES.ts"
        "${CMAKE_CURRENT_LIST_DIR}/../translations/FileSort_fa_FA.ts"
        "${CMAKE_CURRENT_LIST_DIR}/../translations/FileSort_fr_FR.ts"
        "${CMAKE_CURRENT_LIST_DIR}/../translations/FileSort_he_IL.ts"
        "${CMAKE_CURRENT_LIST_DIR}/../translations/FileSort_id_ID.ts"
        "${CMAKE_CURRENT_LIST_DIR}/../translations/FileSort_it_IT.ts"
        "${CMAKE_CURRENT_LIST_DIR}/../translations/FileSort_ja_JP.ts"
        "${CMAKE_CURRENT_LIST_DIR}/../translations/FileSort_ko_KR.ts"
        "${CMAKE_CURRENT_LIST_DIR}/../translations/FileSort_pl_PL.ts"
        "${CMAKE_CURRENT_LIST_DIR}/../translations/FileSort_pt_BR.ts"
        "${CMAKE_CURRENT_LIST_DIR}/../translations/FileSort_pt_PT.ts"
        "${CMAKE_CURRENT_LIST_DIR}/../translations/FileSort_ru_RU.ts"
        "${CMAKE_CURRENT_LIST_DIR}/../translations/FileSort_sv_SE.ts"
        "${CMAKE_CURRENT_LIST_DIR}/../translations/FileSort_tr_TR.ts"
        "${CMAKE_CURRENT_LIST_DIR}/../translations/FileSort_uk_UA.ts"
        "${CMAKE_CURRENT_LIST_DIR}/../translations/FileSort_vi_VN.ts"
        "${CMAKE_CURRENT_LIST_DIR}/../translations/FileSort_zh_CN.ts"
        "${CMAKE_CURRENT_LIST_DIR}/../translations/FileSort_zh_TW.ts"
    )

    deploy_add_translations(
        TARGET_NAME FileSort_translations
        ADD_TO_ALL
        SOURCE_DIRS "${CMAKE_CURRENT_LIST_DIR}"
        TS_FILES ${FILE_SORT_TS_FILES}
        INSTALL_DESTINATION lang
        WINDOWS_APPDATA_SUBDIR "${X_PROJECTNAME}"
    )
endif()

add_subdirectory("${CMAKE_CURRENT_LIST_DIR}/../dep/XCapstone" "${CMAKE_CURRENT_BINARY_DIR}/XCapstone")
add_subdirectory("${CMAKE_CURRENT_LIST_DIR}/../dep/XArchive"  "${CMAKE_CURRENT_BINARY_DIR}/XArchive")
add_subdirectory("${CMAKE_CURRENT_LIST_DIR}/../dep/XYara"     "${CMAKE_CURRENT_BINARY_DIR}/XYara")
add_subdirectory("${CMAKE_CURRENT_LIST_DIR}/../dep/Formats/xsimd" "${CMAKE_CURRENT_BINARY_DIR}/XSIMD")

add_subdirectory(gui)
