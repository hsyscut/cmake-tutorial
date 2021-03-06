# 设置运行此配置文件所需的CMake最低版本
cmake_minimum_required(VERSION 3.15)

# set the project name and version
# 设置项目名称和版本
project(Tutorial VERSION 1.0)

add_library(tutorial_compiler_flags INTERFACE)
target_compile_features(tutorial_compiler_flags INTERFACE cxx_std_11)

# add compiler warning flags just when building this project via
# the BUILD_INTERFACE genex
# 仅当通过BUILD_INTERFACE生成此项目时添加编译器警告标志
set(gcc_like_cxx "$<COMPILE_LANG_AND_ID:CXX,ARMClang,AppleClang,Clang,GNU>")
set(msvc_cxx "$<COMPILE_LANG_AND_ID:CXX,MSVC>")
target_compile_options(tutorial_compiler_flags INTERFACE
        "$<${gcc_like_cxx}:$<BUILD_INTERFACE:-Wall;-Wextra;-Wshadow;-Wformat=2;-Wunused>>"
        "$<${msvc_cxx}:$<BUILD_INTERFACE:-W3>>"
        )

# control where the static and shared libraries are built so that on windows
# we don't need to tinker with the path to run the executable
# 控制静态和共享库的构建位置，以便在Windows上我们无需修改运行可执行文件的路径
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}")

option(BUILD_SHARED_LIBS "Build using shared libraries" ON)

# configure a header file to pass the version number only
# 配置头文件且仅传递版本号
configure_file(TutorialConfig.h.in TutorialConfig.h)

# add the MathFunctions library
# 添加MathFunctions库
add_subdirectory(MathFunctions)

# add the executable
# 添加一个可执行文件
add_executable(Tutorial tutorial.cxx)
target_link_libraries(Tutorial PUBLIC MathFunctions)

# add the binary tree to the search path for include files
# so that we will find TutorialConfig.h
# 将二进制目录添加到包含文件的搜索路径中，以便我们找到TutorialConfig.h
target_include_directories(Tutorial PUBLIC
        "${PROJECT_BINARY_DIR}"
        )

# add the install targets
# 添加安装规则
install(TARGETS Tutorial DESTINATION bin)
install(FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h"
        DESTINATION include
        )

# enable dashboard scripting
# 启用仪表板脚本
include(CTest)

# does the application run
# 测试应用程序是否运行
add_test(NAME Runs COMMAND Tutorial 25)

# does the usage message work?
# 测试消息是否工作？
add_test(NAME Usage COMMAND Tutorial)
set_tests_properties(Usage
        PROPERTIES PASS_REGULAR_EXPRESSION "Usage:.*number"
        )

# define a function to simplify adding tests
# 定义一个函数以简化添加测试
function(do_test target arg result)
    add_test(NAME Comp${arg} COMMAND ${target} ${arg})
    set_tests_properties(Comp${arg}
            PROPERTIES PASS_REGULAR_EXPRESSION ${result}
            )
endfunction(do_test)

# do a bunch of result based tests
# 做一堆基于结果的测试
do_test(Tutorial 4 "4 is 2")
do_test(Tutorial 9 "9 is 3")
do_test(Tutorial 5 "5 is 2.236")
do_test(Tutorial 7 "7 is 2.645")
do_test(Tutorial 25 "25 is 5")
do_test(Tutorial -25 "-25 is [-nan|nan|0]")
do_test(Tutorial 0.0001 "0.0001 is 0.01")

# setup installer
# 设置安装程序
include(InstallRequiredSystemLibraries)
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/License.txt")
set(CPACK_PACKAGE_VERSION_MAJOR "${Tutorial_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${Tutorial_VERSION_MINOR}")
include(CPack)

# install the configuration targets
# 安装配置目标
install(EXPORT MathFunctionsTargets
        FILE MathFunctionsTargets.cmake
        DESTINATION lib/cmake/MathFunctions
        )

include(CMakePackageConfigHelpers)
# generate the config file that is includes the exports
# 生成包含导出的配置文件
configure_package_config_file(${CMAKE_CURRENT_SOURCE_DIR}/Config.cmake.in
        "${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfig.cmake"
        INSTALL_DESTINATION "lib/cmake/example"
        NO_SET_AND_CHECK_MACRO
        NO_CHECK_REQUIRED_COMPONENTS_MACRO
        )
# generate the version file for the config file
# 生成配置文件的版本文件
write_basic_package_version_file(
        "${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfigVersion.cmake"
        VERSION "${Tutorial_VERSION_MAJOR}.${Tutorial_VERSION_MINOR}"
        COMPATIBILITY AnyNewerVersion
)

# install the configuration file
# 安装配置文件
install(FILES
        ${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfig.cmake
        DESTINATION lib/cmake/MathFunctions
        )

# generate the export targets for the build tree
# needs to be after the install(TARGETS ) command
# 在install(TARGETS)命令之后生成生成树的导出目标
export(EXPORT MathFunctionsTargets
        FILE "${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsTargets.cmake"
        )


