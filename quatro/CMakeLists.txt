project(quatro)

if (NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE "Release")
endif ()

if(CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
    if(DEFINED ENV{QUATRO_INSTALL_DIR})
        set(CMAKE_INSTALL_PREFIX "$ENV{QUATRO_INSTALL_DIR}"
                CACHE PATH "Install prefix (from env)" FORCE)
    else()
        set(CMAKE_INSTALL_PREFIX "${CMAKE_SOURCE_DIR}/install"
                CACHE PATH "Install prefix (default)" FORCE)
    endif()
endif()

include_directories(include)

if (OPENMP_FOUND)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
endif ()

include(3rdparty/find_dependencies.cmake)

find_package(Eigen3 REQUIRED)
find_package(PCL REQUIRED)
find_package(teaserpp REQUIRED)
if(PCL_FOUND)
    message(WARNING "PCL_VER: ${PCL_VERSION}, and PCL_DIR: ${PCL_INCLUDE_DIRS}")
endif()
if(Eigen3_FOUND)
    message(WARNING "EIGEN_VER: ${EIGEN3_VERSION_STRING}, and Eigen DIR: ${EIGEN3_INCLUDE_DIR}")
endif()
if(teaserpp_FOUND)
    message(WARNING "teaserpp found")
endif()

option(QUATRO_TBB "Enable TBB support" ON)
if(QUATRO_TBB)
    message(WARNING "Quatro with TBB")
    list(APPEND CMAKE_MODULE_PATH /usr/local/include /usr/include)
    find_library(TBB_LIBRARY tbb HINTS /usr/lib /usr/local/lib)
    message(WARNING "TBB FOUND, ${TBB_LIBRARY}")
    add_definitions(-DTBB_EN)
    include(ProcessorCount)
    ProcessorCount(N)
    message(WARNING "Processer number: ${N}")
    if(N GREATER 4)
        add_definitions(-DTBB_PROC_NUM=${N})
    else()
        add_definitions(-DTBB_PROC_NUM=1)
    endif()
endif()

set(quatro_srcs
        src/graph.cc
        src/teaser_utils/feature_matcher.cc
        src/teaser_utils/fpfh.cc
)

set(quatro_libs
        stdc++fs
        pmc::pmc
        Eigen3::Eigen
        ${PCL_LIBRARIES}
)

add_library(quatro SHARED ${quatro_srcs})
add_library(quatro::quatro ALIAS quatro)

# Link libraries
target_include_directories(quatro
        PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>
        PRIVATE
        ${PCL_INCLUDE_DIRS}
)

if (QUATRO_TBB)
    target_link_libraries(quatro ${OpenMP_LIBS} ${EIGEN3_LIBS} ${PCL_LIBRARIES} teaserpp::teaser_registration teaserpp::teaser_io pmc::pmc ${TBB_LIBRARY})
else()
    target_link_libraries(quatro ${OpenMP_LIBS} ${EIGEN3_LIBS} ${PCL_LIBRARIES} teaserpp::teaser_registration teaserpp::teaser_io pmc::pmc)
endif()

# Install library target with export
install(TARGETS quatro
        EXPORT quatroTargets
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)