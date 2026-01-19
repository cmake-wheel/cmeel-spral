cmake_minimum_required(VERSION 3.30)

set(ORG ralna)
set(NAME spral)
set(VERSION 2025.09.18)
set(EXT tar.gz)
set(SHA256 1358168ac95297049e4fc810e54a16e0c765796cfbaa156e09979e2620b7dae7)
set(TAG_PREFIX v)

project("cmeel-${NAME}" VERSION ${VERSION})

set(CONFIGURE_COMMAND
    "${CMAKE_COMMAND}" "-E" "env" "PKG_CONFIG_PATH=$ENV{PKG_CONFIG_PATH}" "--"
    "meson" "setup" "builddir" "-Dexamples=true" "-Dtests=true"
    "-Dlibblas=openblas" "-Dliblapack=openblas"
    "-Dlibmetis_path=$ENV{CMEEL_AVAILABLE_PREFIX}/lib"
    "--prefix=${CMAKE_INSTALL_PREFIX}")

if(APPLE)
  set(CONFIGURE_COMMAND ${CONFIGURE_COMMAND} "-Dopenmp=false")
endif()

include(ExternalProject)
ExternalProject_Add(
  ${NAME}
  URL "https://github.com/${ORG}/${NAME}/archive/refs/tags/${TAG_PREFIX}${VERSION}.${EXT}"
  URL_HASH "SHA256=${SHA256}"
  BUILD_IN_SOURCE ON
  DOWNLOAD_EXTRACT_TIMESTAMP OFF
  CONFIGURE_COMMAND ${CONFIGURE_COMMAND}
  BUILD_COMMAND "meson" "compile" "-C" "builddir"
  INSTALL_COMMAND "meson" "install" "-C" "builddir")

# dummy file for install target
install(FILES "README.md" DESTINATION "share/cmeel-${NAME}/")
