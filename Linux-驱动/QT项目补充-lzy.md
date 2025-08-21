## QT项目补充

### 使用cmake来编译QT项目

1. 把QT项目中的源文件复制到rknn的示例代码的src目录中

   ![image-20240731093813252](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240731093813252.png)

2. 把ui转为头文件（如果开启自动转换，就不用做）

   ```shell
   //自己的qt路径/bin/uic ui文件  -o   h文件
   /opt/qt5.15.2/bin/uic mainwindow.ui -o ui_mainwindow.h
   ```

3. 编译CMakeLists.txt

   ```cmake
   cmake_minimum_required(VERSION 3.4.1)
   
   project(rknn_yolov5_demo)
   
   set(CMAKE_AUTOUIC ON)
   set(CMAKE_AUTOMOC ON)
   set(CMAKE_AUTORCC ON)
   
   set(CMAKE_CXX_STANDARD 11)
   set(CMAKE_CXX_STANDARD_REQUIRED ON)
   
   # skip 3rd-party lib dependencies
   set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -Wl,--allow-shlib-undefined")
   
   # install target and libraries
   set(CMAKE_INSTALL_PREFIX ${CMAKE_SOURCE_DIR}/install/rknn_yolov5_demo_${CMAKE_SYSTEM_NAME})
   
   set(CMAKE_SKIP_INSTALL_RPATH FALSE)
   set(CMAKE_BUILD_WITH_INSTALL_RPATH TRUE)
   set(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib")
   
   find_package(Qt5 COMPONENTS Widgets REQUIRED)
   
   # rknn api
   if(TARGET_SOC STREQUAL "rk356x")
     set(RKNN_API_PATH ${CMAKE_SOURCE_DIR}/../../runtime/RK356X/${CMAKE_SYSTEM_NAME}/librknn_api)
   elseif(TARGET_SOC STREQUAL "rk3588")
     set(RKNN_API_PATH ${CMAKE_SOURCE_DIR}/../../runtime/RK3588/${CMAKE_SYSTEM_NAME}/librknn_api)
   else()
     message(FATAL_ERROR "TARGET_SOC is not set, ref value: rk356x or rk3588 or rv110x")
   endif()
   
   if (CMAKE_SYSTEM_NAME STREQUAL "Android")
     set(RKNN_RT_LIB ${RKNN_API_PATH}/${CMAKE_ANDROID_ARCH_ABI}/librknnrt.so)
   else()
     if (CMAKE_C_COMPILER MATCHES "aarch64")
       set(LIB_ARCH aarch64)
     else()
       set(LIB_ARCH armhf)
     endif()
     set(RKNN_RT_LIB ${RKNN_API_PATH}/${LIB_ARCH}/librknnrt.so)
   endif()
   include_directories(${RKNN_API_PATH}/include)
   include_directories(${CMAKE_SOURCE_DIR}/../3rdparty)
   
   # opencv
   if (CMAKE_SYSTEM_NAME STREQUAL "Android")
       set(OpenCV_DIR ${CMAKE_SOURCE_DIR}/../3rdparty/opencv/OpenCV-android-sdk/sdk/native/jni/abi-${CMAKE_ANDROID_ARCH_ABI})
   else()
     if(LIB_ARCH STREQUAL "armhf")
       set(OpenCV_DIR ${CMAKE_SOURCE_DIR}/../3rdparty/opencv/opencv-linux-armhf/share/OpenCV)
     else()
       set(OpenCV_DIR ${CMAKE_SOURCE_DIR}/../3rdparty/opencv/opencv-linux-aarch64/share/OpenCV)
     endif()
   endif()
   find_package(OpenCV REQUIRED)
   
   #rga
   if(TARGET_SOC STREQUAL "rk356x")
     set(RGA_PATH ${CMAKE_SOURCE_DIR}/../3rdparty/rga/RK356X)
   elseif(TARGET_SOC STREQUAL "rk3588")
     set(RGA_PATH ${CMAKE_SOURCE_DIR}/../3rdparty/rga/RK3588)
   else()
     message(FATAL_ERROR "TARGET_SOC is not set, ref value: rk356x or rk3588")
   endif()
   if (CMAKE_SYSTEM_NAME STREQUAL "Android")
     set(RGA_LIB ${RGA_PATH}/lib/Android/${CMAKE_ANDROID_ARCH_ABI}/librga.so)
   else()
     if (CMAKE_C_COMPILER MATCHES "aarch64")
       set(LIB_ARCH aarch64)
     else()
       set(LIB_ARCH armhf)
     endif()
     set(RGA_LIB ${RGA_PATH}/lib/Linux//${LIB_ARCH}/librga.so)
   endif()
   include_directories( ${RGA_PATH}/include)
   
   set(CMAKE_INSTALL_RPATH "lib")
   
   # rknn_yolov5_demo
   include_directories( ${CMAKE_SOURCE_DIR}/include)
   
   add_executable(rknn_yolov5_demo
           #src/main.cc
           #src/postprocess.cc
           src/main.cpp
           src/mainwindow.h
           src/mainwindow.cpp
           src/mainwindow.ui
   )
   
   target_link_libraries(rknn_yolov5_demo
     ${RKNN_RT_LIB}
     ${RGA_LIB}
     ${OpenCV_LIBS}
     Qt5::Widgets
   )
   
   # install target and libraries
   set(CMAKE_INSTALL_PREFIX ${CMAKE_SOURCE_DIR}/install/rknn_yolov5_demo_${CMAKE_SYSTEM_NAME})
   install(TARGETS rknn_yolov5_demo DESTINATION ./)
   
   install(PROGRAMS ${RKNN_RT_LIB} DESTINATION lib)
   install(PROGRAMS ${RGA_LIB} DESTINATION lib)
   install(DIRECTORY model DESTINATION ./)
   ```

   

### 在瑞星微的Yolov5示例中使用opencv

1. 编写使用opencv打开摄像头的代码

2. 由于瑞星微提供的示例中，缺少了摄像头相关的库，所以要自己编译opencv来替换

   1. 下载并解压opencv

   2. 在ubtunu中安装cmake-gui

      ```
      apt-get install cmake-gui
      ```

   3. 运行cmake-gui
   
      ![image-20240731145059963](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240731145059963.png)
   
      ![image-20240731145200244](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240731145200244.png)
   
      ![image-20240731145239714](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240731145239714.png)
   
      ![image-20240731145716798](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240731145716798.png)
   
   
   4. 进入后面选择的要生成的目录中：make
   5. 再make install
   6. 再复制install目录中编译出来的opecv,替换示例代码中3rdparty\opencv\opencv-linux-aarch64\的所有文件