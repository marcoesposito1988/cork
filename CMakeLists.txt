cmake_minimum_required(VERSION 3.5)

project(cork VERSION 1.0.0 LANGUAGES C CXX)

set(HEADERS
        src/cork.h

        src/accel/aabvh.h

        src/file_formats/files.h

        src/isct/absext4.h
        src/isct/empty3d.h
        src/isct/ext4.h
        src/isct/fixext4.h
        src/isct/fixint.h
        src/isct/gmpext4.h
        src/isct/quantization.h
        src/isct/triangle.h
        src/isct/unsafeRayTriIsct.h

        src/math/bbox.h
        src/math/ray.h
        src/math/vec.h

        src/mesh/mesh.h
        src/mesh/mesh.decl.h
        src/mesh/mesh.tpp
        src/mesh/mesh.bool.tpp
        src/mesh/mesh.isct.tpp
        src/mesh/mesh.remesh.tpp
        src/mesh/mesh.topoCache.tpp

        src/rawmesh/rawMesh.h
        src/rawmesh/rawMesh.tpp

        src/util/iterPool.h
        src/util/memPool.h
        src/util/prelude.h
        src/util/shortVec.h
        src/util/unionFind.h
        )

set(SOURCES
        src/cork.cpp
        src/off2obj.cpp

        src/file_formats/files.cpp
        src/file_formats/ifs.cpp
        src/file_formats/off.cpp

        src/isct/empty3d.cpp
        src/isct/quantization.cpp

        src/util/log.cpp
        src/util/timer.cpp
        )


add_library(triangle OBJECT src/isct/triangle.c)
target_compile_definitions(triangle PRIVATE NO_TIMER REDUCED TRILIBRARY CDT_ONLY ANSI_DECLARATORS)

add_library(cork ${SOURCES} $<TARGET_OBJECTS:triangle>)

if (UNIX)
    target_compile_options(triangle PRIVATE -fPIC)
    target_compile_options(cork PRIVATE -fPIC)
endif (UNIX)


target_include_directories(${PROJECT_NAME} PUBLIC
        # for headers when building
        $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/src>
        $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/src/accel>
        $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/src/file_formats>
        $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/src/isct>
        $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/src/math>
        $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/src/mesh>
        $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/src/rawmesh>
        $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/src/util>

        # for client in install mode
        $<INSTALL_INTERFACE:include>
        )

# install
include(GNUInstallDirs)
set(INSTALL_CONFIGDIR ${CMAKE_INSTALL_LIBDIR}/cmake/cork)

install(TARGETS cork EXPORT cork-targets
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        )

#This is required so that the exported target has the name Cork and not cork
set_target_properties(cork PROPERTIES EXPORT_NAME Cork)

# install headers
INSTALL(FILES ${HEADERS}
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
        COMPONENT CPP_INCLUDES
        )

#Export the targets to a script
install(EXPORT cork-targets
        FILE corkTargets.cmake
        NAMESPACE Cork::
        DESTINATION ${INSTALL_CONFIGDIR}
        )


#Create a ConfigVersion.cmake file
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
        ${CMAKE_CURRENT_BINARY_DIR}/corkConfigVersion.cmake
        VERSION ${PROJECT_VERSION}
        COMPATIBILITY AnyNewerVersion
)


configure_package_config_file(${CMAKE_CURRENT_LIST_DIR}/corkConfig.cmake.in
        ${CMAKE_CURRENT_BINARY_DIR}/corkConfig.cmake
        INSTALL_DESTINATION ${INSTALL_CONFIGDIR}
        )

#Install the config, ConfigVersion
install(FILES
        ${CMAKE_CURRENT_BINARY_DIR}/corkConfig.cmake
        ${CMAKE_CURRENT_BINARY_DIR}/corkConfigVersion.cmake
        DESTINATION ${INSTALL_CONFIGDIR}
        )