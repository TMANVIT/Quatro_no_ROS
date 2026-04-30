cmake_minimum_required(VERSION 3.14)
project(quatro VERSION 1.0.0 LANGUAGES CXX)

include(GNUInstallDirs)

find_package(Eigen3 REQUIRED)
find_package(PCL REQUIRED)
find_package(teaserpp REQUIRED)
find_package(OpenMP REQUIRED)

option(QUATRO_TBB "Enable TBB support" ON)
if(QUATRO_TBB)
    find_package(TBB REQUIRED)
    include(ProcessorCount)
    ProcessorCount(N)
    set(TBB_PROC_NUM ${N})
    if(N LESS_EQUAL 4)
        set(TBB_PROC_NUM 1)
    endif()
endif()

add_library(quatro SHARED
        src/feature_matcher.cc
        src/fpfh.cc
        src/quatro.cpp
)
add_library(quatro::quatro ALIAS quatro)

target_compile_features(quatro PUBLIC cxx_std_17)

target_include_directories(quatro
        PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)

if(QUATRO_TBB)
    target_compile_definitions(quatro PUBLIC TBB_EN TBB_PROC_NUM=${TBB_PROC_NUM})
endif()

target_link_libraries(quatro
        PUBLIC
        Eigen3::Eigen
        ${PCL_LIBRARIES}
        teaserpp::teaser_registration
        teaserpp::teaser_io
        OpenMP::OpenMP_CXX
)

if(QUATRO_TBB)
    target_link_libraries(quatro PUBLIC TBB::tbb)
endif()

if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU" AND CMAKE_CXX_COMPILER_VERSION VERSION_LESS 9.0)
    target_link_libraries(quatro PRIVATE stdc++fs)
endif()


# install(DIRECTORY include/quatro DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
# install(DIRECTORY include/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/quatro)

install(TARGETS quatro
        EXPORT quatroTargets
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)

install(EXPORT quatroTargets
        FILE quatroTargets.cmake
        NAMESPACE quatro::
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/quatro
)

include(CMakePackageConfigHelpers)
configure_package_config_file(
        "${CMAKE_CURRENT_SOURCE_DIR}/cmake/quatroConfig.cmake.in"
        "${CMAKE_CURRENT_BINARY_DIR}/quatroConfig.cmake"
        INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/quatro
)

write_basic_package_version_file(
        "${CMAKE_CURRENT_BINARY_DIR}/quatroConfigVersion.cmake"
        VERSION ${PROJECT_VERSION}
        COMPATIBILITY SameMajorVersion
)

install(FILES
        "${CMAKE_CURRENT_BINARY_DIR}/quatroConfig.cmake"
        "${CMAKE_CURRENT_BINARY_DIR}/quatroConfigVersion.cmake"
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/quatro
)

install(DIRECTORY include/quatro DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})