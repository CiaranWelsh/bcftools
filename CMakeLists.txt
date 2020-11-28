########################################################
#   BCFTOOLSlib
#

cmake_minimum_required(VERSION 3.16)

set(BCFTOOLS_VERSION_MAJOR 1)
set(BCFTOOLS_VERSION_MINOR 11)
set(BCFTOOLS_VERSION_MICRO 21)
set(VERSION "${BCFTOOLS_VERSION_MAJOR}.${BCFTOOLS_VERSION_MINOR}.${BCFTOOLS_VERSION_MICRO}")

project(bcftools-cmake VERSION ${VERSION})


include(CheckSymbolExists)
include(CheckCXXSymbolExists)
include(CheckLibraryExists)
include(CheckIncludeFile)
include(CheckIncludeFileCXX)
include(CheckFunctionExists)
include(CheckTypeSize)
include(GNUInstallDirs)

set(CMAKE_POSITION_INDEPENDENT_CODE ON)
set(CMAKE_VISIBILITY_INLINES_HIDDEN ON)
set(CMAKE_VERBOSE_MAKEFILE ON)

# put all runtime targets (exe and dll) into bin
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/bin)

# put libraries into lib
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/lib)

# archives
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/lib)

if (MSVC)
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS}" "/NODEFAULTLIB")
endif ()

########################################################
#   dependencies
#

add_subdirectory(third_party)

find_package(ZLIB REQUIRED)

find_path(
        HTS_INSTALL_PREFIX
        NAMES
        lib/${CMAKE_SHARED_LIBRARY_PREFIX}hts${CMAKE_SHARED_LIBRARY_SUFFIX}
        lib/${CMAKE_SHARED_LIBRARY_PREFIX}htslib${CMAKE_SHARED_LIBRARY_SUFFIX}
        include/htslib/hts.h
)
if (NOT HTS_INSTALL_PREFIX)
    message(WARNING "Could not automatically locate\
path to htslib. Please provide the path to the install\
tree for htslib. If you add this directory to PATH or install\
htslib in a system directory (/usr/local or C:/Program Files) it will\
be found automatically. ")
endif ()

message(STATUS "${HTS_INSTALL_PREFIX}/lib/cmake/hts")
set(CMAKE_PREFIX_PATH "${CMAKE_PREFIX_PATH}" "${HTS_INSTALL_PREFIX}/lib/cmake/hts")

find_package(HTS CONFIG REQUIRED)

find_library(
        HTS_LIBRARY
        NAMES
        ${CMAKE_SHARED_LIBRARY_PREFIX}hts${CMAKE_SHARED_LIBRARY_SUFFIX}
        ${CMAKE_SHARED_LIBRARY_PREFIX}htslib${CMAKE_SHARED_LIBRARY_SUFFIX}
        HINTS
        "${HTS_INSTALL_PREFIX}"
        "${HTS_INSTALL_PREFIX}/lib"
        "${HTS_INSTALL_PREFIX}/bin"
)

find_library(
        HTS_STATIC_LIBRARY
        NAMES
        ${CMAKE_STATIC_LIBRARY_PREFIX}hts${CMAKE_STATIC_LIBRARY_SUFFIX}
        ${CMAKE_STATIC_LIBRARY_PREFIX}htslib${CMAKE_STATIC_LIBRARY_SUFFIX}
        HINTS
        "${HTS_INSTALL_PREFIX}"
        "${HTS_INSTALL_PREFIX}/lib"
        "${HTS_INSTALL_PREFIX}/bin"
)

find_path(
        HTS_INCLUDE_DIR
        NAMES
        htslib/hts.h
        HINTS
        "${HTS_INSTALL_PREFIX}"
        "${HTS_INSTALL_PREFIX}/include"
)

########################################################
#   make a config.h like the one produced by ./configure
#


#/* config.h.in.  Generated from configure.ac by autoheader.  */

#/* Define if BCFtools should enable plugins. */
option(ENABLE_BCF_PLUGINS "enable bcf plugins" OFF)

#/* Define if BCFtools should enable for support PERL scripts in -i/-e
#   filtering expressions. */
option(ENABLE_PERL_FILTERS "support for PERL scripts" OFF)

#/* Define to 1 if you have the `z' library (-lz). */
check_library_exists(z zlibVersion ${ZLIB_LIBRARIES} HAVE_LIBZ)

#/* Define to the address where bug reports for this package should be sent. */
set(PACKAGE_BUGREPORT)

#/* Define to the full name of this package. */
set(PACKAGE_NAME "${PROJECT_NAME}")

#/* Define to the full name and version of this package. */
set(PACKAGE_STRING "${PROJECT_NAME}-${PROJECT_VERSION}")

#/* Define to the one symbol short name of this package. */
set(PACKAGE_TARNAME)

#/* Define to the home page for this package. */
set(PACKAGE_URL)

#/* Define to the version of this package. */
set(PACKAGE_VERSION "${PROJECT_VERSION}")

#/* Platform-dependent plugin filename extension. */
set(PLUGIN_EXT "${CMAKE_SHARED_LIBRARY_SUFFIX}")

#/* Number of bits in a file offset, on hosts where this is settable. */
#undef _FILE_OFFSET_BITS

#/* Define for large files, on AIX-style hosts. */
#undef _LARGE_FILES


#/* Define to 1 if you have the ANSI C header files. */
check_include_file(math.h STDC_HEADERS)

#/* Define to 1 if you have the unistd. This way is preferable to #ifdef _WIN32 because
# _WIN32 is also defined on WSL. */
check_include_file(unistd.h HAVE_UNISTD_H)

# check to see if we have sys/time.h
check_include_file(sys/time.h HAVE_SYS_TIME_H)

# check to see if we have getopt
check_include_file(getopt.h HAVE_GETOPT_H)

# check to see if we have usleep
check_include_file(usleep.h HAVE_USLEEP_H)

# on linux we have include strings while windows its string (no 's')
check_include_file(string.h HAVE_STRING)
check_include_file(strings.h HAVE_STRINGS)

# Check whether we have dirent.h
check_include_file(dirent.h HAVE_DIRENT_H)

# Check whether we have dlfcn.h
check_include_file(dlfcn.h HAVE_DLFCN_H)


configure_file(
        "${CMAKE_CURRENT_SOURCE_DIR}/config.h.in"
        "${CMAKE_CURRENT_BINARY_DIR}/config.h"
)

set(BCFTOOLS_VERSION_TEXT ${PROJECT_VERSION})
configure_file(
        "${CMAKE_CURRENT_SOURCE_DIR}/version.h.in"
        "${CMAKE_CURRENT_BINARY_DIR}/version.h"
)


########################################################
#   Build
#

set(
        BCF_SOURCES
        ${CMAKE_CURRENT_SOURCE_DIR}/HMM.h
        ${CMAKE_CURRENT_SOURCE_DIR}/bam2bcf.h
        ${CMAKE_CURRENT_SOURCE_DIR}/bam_sample.h
        ${CMAKE_CURRENT_SOURCE_DIR}/bcftools.h
        ${CMAKE_CURRENT_SOURCE_DIR}/bin.h
        ${CMAKE_CURRENT_SOURCE_DIR}/call.h
        ${CMAKE_CURRENT_SOURCE_DIR}/cols.h
        ${CMAKE_CURRENT_BINARY_DIR}/config.h
        ${CMAKE_CURRENT_SOURCE_DIR}/convert.h
        ${CMAKE_CURRENT_SOURCE_DIR}/extsort.h
        ${CMAKE_CURRENT_SOURCE_DIR}/filter.h
        ${CMAKE_CURRENT_SOURCE_DIR}/gvcf.h
        ${CMAKE_CURRENT_SOURCE_DIR}/hclust.h
        ${CMAKE_CURRENT_SOURCE_DIR}/hex.h
        ${CMAKE_CURRENT_SOURCE_DIR}/khash_str2str.h
        ${CMAKE_CURRENT_SOURCE_DIR}/kheap.h
        ${CMAKE_CURRENT_SOURCE_DIR}/kmin.h
        ${CMAKE_CURRENT_SOURCE_DIR}/mw.h
        ${CMAKE_CURRENT_SOURCE_DIR}/peakfit.h
        ${CMAKE_CURRENT_SOURCE_DIR}/ploidy.h
        ${CMAKE_CURRENT_SOURCE_DIR}/prob1.h
        ${CMAKE_CURRENT_SOURCE_DIR}/rbuf.h
        ${CMAKE_CURRENT_SOURCE_DIR}/regidx.h
        ${CMAKE_CURRENT_SOURCE_DIR}/smpl_ilist.h
        ${CMAKE_CURRENT_SOURCE_DIR}/tsv2vcf.h
        ${CMAKE_CURRENT_SOURCE_DIR}/variantkey.h
        ${CMAKE_CURRENT_SOURCE_DIR}/vcfbuf.h
        ${CMAKE_CURRENT_SOURCE_DIR}/vcmp.h
        ${CMAKE_CURRENT_SOURCE_DIR}/version.h
        ${CMAKE_CURRENT_SOURCE_DIR}/HMM.c
        ${CMAKE_CURRENT_SOURCE_DIR}/bam2bcf.c
        ${CMAKE_CURRENT_SOURCE_DIR}/bam2bcf_indel.c
        ${CMAKE_CURRENT_SOURCE_DIR}/bam_sample.c
        ${CMAKE_CURRENT_SOURCE_DIR}/bin.c
        ${CMAKE_CURRENT_SOURCE_DIR}/ccall.c
        ${CMAKE_CURRENT_SOURCE_DIR}/cols.c
        ${CMAKE_CURRENT_SOURCE_DIR}/consensus.c
        ${CMAKE_CURRENT_SOURCE_DIR}/convert.c
        ${CMAKE_CURRENT_SOURCE_DIR}/csq.c
        ${CMAKE_CURRENT_SOURCE_DIR}/em.c
        ${CMAKE_CURRENT_SOURCE_DIR}/extsort.c
        ${CMAKE_CURRENT_SOURCE_DIR}/filter.c
        ${CMAKE_CURRENT_SOURCE_DIR}/gvcf.c
        ${CMAKE_CURRENT_SOURCE_DIR}/hclust.c
        ${CMAKE_CURRENT_SOURCE_DIR}/kmin.c
        ${CMAKE_CURRENT_SOURCE_DIR}/main.c
        ${CMAKE_CURRENT_SOURCE_DIR}/mcall.c
        ${CMAKE_CURRENT_SOURCE_DIR}/mpileup.c
        ${CMAKE_CURRENT_SOURCE_DIR}/peakfit.c
        ${CMAKE_CURRENT_SOURCE_DIR}/ploidy.c
        ${CMAKE_CURRENT_SOURCE_DIR}/polysomy.c
        ${CMAKE_CURRENT_SOURCE_DIR}/prob1.c
        ${CMAKE_CURRENT_SOURCE_DIR}/regidx.c
        ${CMAKE_CURRENT_SOURCE_DIR}/reheader.c
        ${CMAKE_CURRENT_SOURCE_DIR}/smpl_ilist.c
        ${CMAKE_CURRENT_SOURCE_DIR}/tabix.c
        ${CMAKE_CURRENT_SOURCE_DIR}/tsv2vcf.c
        ${CMAKE_CURRENT_SOURCE_DIR}/vcfannotate.c
        ${CMAKE_CURRENT_SOURCE_DIR}/vcfbuf.c
        ${CMAKE_CURRENT_SOURCE_DIR}/vcfcall.c
        ${CMAKE_CURRENT_SOURCE_DIR}/vcfcnv.c
        ${CMAKE_CURRENT_SOURCE_DIR}/vcfconcat.c
        ${CMAKE_CURRENT_SOURCE_DIR}/vcfconvert.c
        ${CMAKE_CURRENT_SOURCE_DIR}/vcffilter.c
        ${CMAKE_CURRENT_SOURCE_DIR}/vcfgtcheck.c
        ${CMAKE_CURRENT_SOURCE_DIR}/vcfindex.c
        ${CMAKE_CURRENT_SOURCE_DIR}/vcfisec.c
        ${CMAKE_CURRENT_SOURCE_DIR}/vcfmerge.c
        ${CMAKE_CURRENT_SOURCE_DIR}/vcfnorm.c
        ${CMAKE_CURRENT_SOURCE_DIR}/vcfplugin.c
        ${CMAKE_CURRENT_SOURCE_DIR}/vcfquery.c
        ${CMAKE_CURRENT_SOURCE_DIR}/vcfroh.c
        ${CMAKE_CURRENT_SOURCE_DIR}/vcfsom.c
        ${CMAKE_CURRENT_SOURCE_DIR}/vcfsort.c
        ${CMAKE_CURRENT_SOURCE_DIR}/vcfstats.c
        ${CMAKE_CURRENT_SOURCE_DIR}/vcfview.c
        ${CMAKE_CURRENT_SOURCE_DIR}/vcmp.c
        #        ${CMAKE_CURRENT_SOURCE_DIR}/version.c
)

set(
        INCLUDE_DIRECTORIES
        "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>"
        "$<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>"
        "$<BUILD_INTERFACE:${HTS_INCLUDE_DIR}>"
        "$<INSTALL_INTERFACE:include>"
)

set(LINK_TARGETS)
if ("${ZLIB_FOUND}")
    set(LINK_TARGETS ${LINK_TARGETS} ZLIB::ZLIB)
endif ()

# build static library
add_library(bcftools-static STATIC ${BCF_SOURCES})
target_include_directories(bcftools-static PUBLIC ${INCLUDE_DIRECTORIES})
target_link_libraries(bcftools-static PUBLIC ${LINK_TARGETS}
        linux2win # interface library defined by Linux2Windows
        )
target_compile_definitions(bcftools-static PUBLIC BCFTOOLS_STATIC_DEFINE)

# build static library
add_library(bcftools STATIC ${BCF_SOURCES})
target_include_directories(bcftools PUBLIC ${INCLUDE_DIRECTORIES})
target_link_libraries(bcftools PUBLIC ${LINK_TARGETS}
        linux2win # interface library defined by Linux2Windows
        )
target_compile_definitions(bcftools PUBLIC BCFTOOLS_EXPORTS)

include(GenerateExportHeader)
generate_export_header(bcftools)

option(BUILD_TESTS "build bcftools tests" OFF)
if (BUILD_TESTS)
    add_subdirectory(test)
endif ()


##########################################################################################
#       Executables
#
