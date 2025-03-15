cmake_minimum_required(VERSION 3.22)
enable_language(C ASM)

set(Drivers_Defines
)

# 驱动头文件
set(Drivers_Hearers
    "${CMAKE_CURRENT_LIST_DIR}/rt-thread/inc"
    "${CMAKE_CURRENT_LIST_DIR}/finsh/inc"
)

# 源文件
file(GLOB Drivers_Sources
    "${CMAKE_CURRENT_LIST_DIR}/rt-thread/src/*.c"
    "${CMAKE_CURRENT_LIST_DIR}/rt-thread/src/*.S"
    "${CMAKE_CURRENT_LIST_DIR}/finsh/src/*.c"
)

add_library(rtthread INTERFACE)

target_include_directories(rtthread INTERFACE ${Drivers_Hearers})
target_compile_definitions(rtthread INTERFACE ${Drivers_Defines})

# create drivers object library
add_library(rtthread_driver OBJECT)

# message(STATUS "Drivers_Sources: ${Drivers_Sources}")
target_sources(rtthread_driver PRIVATE ${Drivers_Sources})
target_link_libraries(rtthread_driver PUBLIC rtthread)
target_compile_options(rtthread_driver PRIVATE -Wno-error -Wno-pedantic -Wno-extra)

# Add libraries to the project
target_link_libraries(${CMAKE_PROJECT_NAME} PRIVATE rtthread_driver)