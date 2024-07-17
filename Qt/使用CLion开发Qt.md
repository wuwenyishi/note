CMakeLists.txt 文件的配置

```
cmake_minimum_required(VERSION 3.16)
project(sur)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_PREFIX_PATH "D:/install/Qt/Qt5.9.9/5.9.9/mingw53_32")
#set(CMAKE_PREFIX_PATH "/Users/yons/Qt5.9.8/5.9.8/clang_64")
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_AUTOUIC ON)

set(QT_VERSION 5)
set(REQUIRED_LIBS Core Gui Widgets Sql SerialBus)
set(REQUIRED_LIBS_QUALIFIED Qt5::Core Qt5::Gui Qt5::Widgets Qt5::Sql Qt5::SerialBus)
include_directories(./server ./auth ./base ./common ./db ./entity ./sysm ./user ./calibrate ./param ./formula ./fonts ./image ./log ./qss ./report ./tendency ./home)
file(GLOB UI_FILES ./home/*.ui *.ui ./*.ui ./alarm/*.ui ./auth/*.ui ./base/*.ui ./calibrate/*.ui ./common/*.ui ./formula/*.ui ./log/*.ui ./param/*.ui ./report/*.ui ./sysm/*.ui ./tendency/*.ui ./user/*.ui)
file(GLOB SRC_FILES *.cpp ./server/*.cpp ./home/*.cpp ./*.cpp ./db/*.cpp ./alarm/*.cpp ./auth/*.cpp ./base/*.cpp ./calibrate/*.cpp ./common/*.cpp ./formula/*.cpp ./log/*.cpp ./param/*.cpp ./report/*.cpp ./sysm/*.cpp ./tendency/*.cpp ./user/*.cpp)
file(GLOB HEAD_FILES *.h ./server/*.h ./home/*.h ./*.h ./db/*.h ./entity/*.h ./alarm/*.h ./auth/*.h ./base/*.h ./calibrate/*.h ./common/*.h ./formula/*.h ./log/*.h ./param/*.h ./report/*.h ./sysm/*.h ./tendency/*.h ./user/*.h)
file(GLOB RESOURCES ./*.qrc )
add_executable(${PROJECT_NAME} ${UI_FILES} ${SRC_FILES} ${HEAD_FILES} ${RESOURCES} )

if (NOT CMAKE_PREFIX_PATH)
    message(WARNING "CMAKE_PREFIX_PATH is not defined, you may need to set it "
            "(-DCMAKE_PREFIX_PATH=\"path/to/Qt/lib/cmake\" or -DCMAKE_PREFIX_PATH=/usr/include/{host}/qt{version}/ on Ubuntu)")
endif ()

find_package(Qt${QT_VERSION} COMPONENTS ${REQUIRED_LIBS} REQUIRED)
target_link_libraries(${PROJECT_NAME} ${REQUIRED_LIBS_QUALIFIED})
if (WIN32)
    set(DEBUG_SUFFIX)
    if (CMAKE_BUILD_TYPE MATCHES "Debug")
        set(DEBUG_SUFFIX "d")
    endif ()
    set(QT_INSTALL_PATH "${CMAKE_PREFIX_PATH}")
    if (NOT EXISTS "${QT_INSTALL_PATH}/bin")
        set(QT_INSTALL_PATH "${QT_INSTALL_PATH}/..")
        if (NOT EXISTS "${QT_INSTALL_PATH}/bin")
            set(QT_INSTALL_PATH "${QT_INSTALL_PATH}/..")
        endif ()
    endif ()
    if (EXISTS "${QT_INSTALL_PATH}/plugins/platforms/qwindows${DEBUG_SUFFIX}.dll")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E make_directory
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/platforms/")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E copy
                "${QT_INSTALL_PATH}/plugins/platforms/qwindows${DEBUG_SUFFIX}.dll"
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/platforms/")
    endif ()
    foreach (QT_LIB ${REQUIRED_LIBS})
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E copy
                "${QT_INSTALL_PATH}/bin/Qt${QT_VERSION}${QT_LIB}${DEBUG_SUFFIX}.dll"
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>")
    endforeach (QT_LIB)
endif ()

```

