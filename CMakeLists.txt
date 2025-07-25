cmake_minimum_required(VERSION 3.20)

if(NOT DEFINED PROJECT_NAME)
  set(ARGPARSE_IS_TOP_LEVEL ON)
else()
  set(ARGPARSE_IS_TOP_LEVEL OFF)
endif()

project(argparse
        VERSION 3.2.0 
        DESCRIPTION "A single header argument parser for C++17"
        HOMEPAGE_URL "https://github.com/p-ranav/argparse"
        LANGUAGES CXX
)

option(ARGPARSE_INSTALL "Include an install target" ${ARGPARSE_IS_TOP_LEVEL})
option(ARGPARSE_BUILD_TESTS "Build tests" ${ARGPARSE_IS_TOP_LEVEL})
option(ARGPARSE_BUILD_SAMPLES "Build samples" OFF)

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

add_library(argparse INTERFACE)
add_library(argparse::argparse ALIAS argparse)

target_compile_features(argparse INTERFACE cxx_std_17)
target_include_directories(argparse INTERFACE
  $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/include>)

if(ARGPARSE_BUILD_SAMPLES)
  add_subdirectory(samples)
endif()
  
if(ARGPARSE_BUILD_TESTS)
  add_subdirectory(test)
endif()
  
if(ARGPARSE_INSTALL)
  install(TARGETS argparse EXPORT argparseConfig)
  install(EXPORT argparseConfig
          NAMESPACE argparse::
          DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME})
  install(FILES ${CMAKE_CURRENT_LIST_DIR}/include/argparse/argparse.hpp
          DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/argparse)


  set(CONFIG_FILE_NAME_WITHOUT_EXT "${PROJECT_NAME}Config")
  set(CMAKE_CONFIG_FILE_BASENAME "${CMAKE_CURRENT_BINARY_DIR}/${CONFIG_FILE_NAME_WITHOUT_EXT}")
  set(CMAKE_CONFIG_VERSION_FILE_NAME "${CMAKE_CONFIG_FILE_BASENAME}-version.cmake")
  set(CMAKE_CONFIG_FILE_NAME "${CMAKE_CONFIG_FILE_BASENAME}.cmake")

  if(${CMAKE_VERSION} VERSION_GREATER "3.14")
  	set(OPTIONAL_ARCH_INDEPENDENT "ARCH_INDEPENDENT")
  endif()

  write_basic_package_version_file("${CMAKE_CONFIG_VERSION_FILE_NAME}"
      COMPATIBILITY SameMajorVersion
      ${OPTIONAL_ARCH_INDEPENDENT}
  )

  export(EXPORT argparseConfig
         NAMESPACE argparse::)

  install(FILES "${CMAKE_CONFIG_VERSION_FILE_NAME}"
         DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}")

  set(PackagingTemplatesDir "${CMAKE_CURRENT_SOURCE_DIR}/packaging")

  set(CPACK_PACKAGE_NAME "${PROJECT_NAME}")
  set(CPACK_PACKAGE_VENDOR "argparse (C++) developers")
  set(CPACK_PACKAGE_DESCRIPTION "${PROJECT_DESCRIPTION}")
  set(CPACK_DEBIAN_PACKAGE_NAME "${CPACK_PACKAGE_NAME}")
  set(CPACK_RPM_PACKAGE_NAME "${CPACK_PACKAGE_NAME}")
  set(CPACK_PACKAGE_HOMEPAGE_URL "${PROJECT_HOMEPAGE_URL}")
  set(CPACK_PACKAGE_MAINTAINER "Pranav Srinivas Kumar")
  set(CPACK_DEBIAN_PACKAGE_MAINTAINER "${CPACK_PACKAGE_MAINTAINER}")
  set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/LICENSE")
  set(CPACK_RESOURCE_FILE_README "${CMAKE_CURRENT_SOURCE_DIR}/README.md")

  set(CPACK_DEBIAN_PACKAGE_NAME "lib${PROJECT_NAME}-dev")
  set(CPACK_DEBIAN_PACKAGE_DEPENDS "libc6-dev")
  set(CPACK_DEBIAN_PACKAGE_SUGGESTS "cmake, pkg-config, pkg-conf")

  set(CPACK_RPM_PACKAGE_NAME "lib${PROJECT_NAME}-devel")
  set(CPACK_RPM_PACKAGE_SUGGESTS "${CPACK_DEBIAN_PACKAGE_SUGGESTS}")

  set(CPACK_DEB_COMPONENT_INSTALL ON)
  set(CPACK_RPM_COMPONENT_INSTALL ON)
  set(CPACK_NSIS_COMPONENT_INSTALL ON)
  set(CPACK_DEBIAN_COMPRESSION_TYPE "xz")

  set(PKG_CONFIG_FILE_NAME "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.pc")
  configure_file("${PackagingTemplatesDir}/pkgconfig.pc.in" "${PKG_CONFIG_FILE_NAME}" @ONLY)
  install(FILES "${PKG_CONFIG_FILE_NAME}"
        DESTINATION "${CMAKE_INSTALL_LIBDIR}/pkgconfig"
  )
endif()

include(CPack)
