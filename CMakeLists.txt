cmake_minimum_required(VERSION 3.20)

set(ORG ralna)
set(NAME spral)
set(VERSION 2023.03.29)

project(cmeel-${NAME} VERSION ${VERSION})

include(FindPkgConfig)
pkg_check_modules(BLAS REQUIRED openblas)
pkg_check_modules(HWLOC REQUIRED hwloc)

list(JOIN BLAS_LDFLAGS " " BLAS_LIBS)
list(JOIN HWLOC_LDFLAGS " " HWLOC_LIBS)
string(REPLACE "hwloc" "metis" METIS_LIBS ${HWLOC_LIBS}) # ugh

set(SET_FLAGS
    "${CMAKE_COMMAND}"
    "-E"
    "env"
    "CXXFLAGS=-I${HWLOC_PREFIX}/include"
    "FFLAGS=${HWLOC_LIBS}"
    "FCFLAGS=${HWLOC_LIBS}"
    "--") # ugh

include(ExternalProject)
ExternalProject_Add(
  ${NAME}
  URL "https://github.com/${ORG}/${NAME}/archive/refs/tags/v${VERSION}.tar.gz"
  BUILD_IN_SOURCE ON
  DOWNLOAD_EXTRACT_TIMESTAMP OFF
  CONFIGURE_COMMAND "./autogen.sh"
  COMMAND "patch" "-i" "${CMAKE_SOURCE_DIR}/configure.patch"
  COMMAND
    ${SET_FLAGS} "./configure" "--prefix=${CMAKE_INSTALL_PREFIX}"
    "--with-blas=${BLAS_LIBS}" "--with-lapack=${BLAS_LIBS}"
    "--with-metis=${METIS_LIBS}"
    "--with-metis-inc-dir=${HWLOC_PREFIX}/include" # ugh
  # "--disable-dependency-tracking" "--disable-openmp" "--disable-gpu"
  BUILD_COMMAND "make"
  INSTALL_COMMAND "make" "install")

# dummy file for install target
install(FILES README.md DESTINATION share/cmeel-${NAME}/)
