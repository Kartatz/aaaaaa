cmake_minimum_required(VERSION 3.19)

project(
	something
	VERSION 0.1
	DESCRIPTION "..."
	HOMEPAGE_URL "..."
	LANGUAGES C CXX
)

set(CMAKE_POLICY_DEFAULT_CMP0069 NEW)
set(CMAKE_POLICY_DEFAULT_CMP0048 NEW)
set(CMAKE_POLICY_DEFAULT_CMP0077 NEW)

set(CMAKE_PLATFORM_NO_VERSIONED_SONAME ON)

set(SOVERSION_SEPARATOR ".")

set(LIBVPX_SOURCE_DIRECTORY "${CMAKE_SOURCE_DIR}/submodules/libvpx")
set(LIBVPX_BUILD_DIRECTORY "${CMAKE_BINARY_DIR}/libvpx-build")
set(LIBVPX_INSTALL_PREFIX "${LIBVPX_BUILD_DIRECTORY}/binaries")

# Extract libvpx version
file(READ "${LIBVPX_SOURCE_DIRECTORY}/libs.mk" FILE_CONTENTS)

string(REGEX MATCH "SO_VERSION_MAJOR := *([0-9]+)" _ ${FILE_CONTENTS})
set(LIBVPX_VERSION_MAJOR "${CMAKE_MATCH_1}")

string(REGEX MATCH "SO_VERSION_MINOR := *([0-9]+)" _ ${FILE_CONTENTS})
set(LIBVPX_VERSION_MINOR "${CMAKE_MATCH_1}")

string(REGEX MATCH "SO_VERSION_PATCH := *([0-9]+)" _ ${FILE_CONTENTS})
set(LIBVPX_VERSION_MICRO "${CMAKE_MATCH_1}")

set(LIBVPX_SOVERSION "${LIBVPX_VERSION_MAJOR}.${LIBVPX_VERSION_MINOR}.${LIBVPX_VERSION_MICRO}")
set(LIBVPX_LIBRARY_NAME "${CMAKE_SHARED_LIBRARY_PREFIX}vpx${CMAKE_SHARED_LIBRARY_SUFFIX}${SOVERSION_SEPARATOR}${LIBVPX_SOVERSION}")
message("${LIBVPX_LIBRARY_NAME}")
set(LIBVPX_TARGET)

if ("${ANDROID_ABI}" STREQUAL "arm64-v8a")
	set(LIBVPX_TARGET "arm64-android-gcc")
elseif ("${ANDROID_ABI}" STREQUAL "armeabi-v7a")
	set(LIBVPX_TARGET "armv7-android-gcc")
elseif ("${ANDROID_ABI}" STREQUAL "x86")
	set(LIBVPX_TARGET "x86-android-gcc")
elseif ("${ANDROID_ABI}" STREQUAL "x86_64")
	set(LIBVPX_TARGET "x86_64-android-gcc")
else()
	error("Unknown Android ABI: ${ANDROID_ABI}")
endif()

set(
	LIBVPX_EXTRA_C_FLAGS
	"${CMAKE_C_FLAGS} -Wno-error"
)

set(
	LIBVPX_EXTRA_CXX_FLAGS
	"${CMAKE_CXX_FLAGS} -Wno-error"
)

set(LIBVPX_CC "${ANDROID_TOOLCHAIN_ROOT}/bin/${TOOLCHAIN_PREFIX}-clang")
set(LIBVPX_CXX "${ANDROID_TOOLCHAIN_ROOT}/bin/${TOOLCHAIN_PREFIX}-clang++")
set(LIBVPX_LD "${LIBVPX_CC}")
set(LIBVPX_AR "${CMAKE_AR}")
set(LIBVPX_NM "${CMAKE_NM}")
set(LIBVPX_STRIP "${CMAKE_STRIP}")
set(LIBVPX_RANLIB "${CMAKE_RANLIB}")

set(
	LIBVPX_CONFIGURE_FLAGS
	--disable-static
	--enable-shared
	--enable-pic
	--enable-libyuv
	--enable-small
	--enable-optimizations
	--enable-better-hw-compatibility
	--disable-examples
	--disable-tools
	--disable-debug
	--disable-neon-asm
	--disable-neon-dotprod
	--disable-unit-tests
	--disable-install-docs
	--disable-docs
	--enable-realtime-only
	--enable-vp8
	--enable-vp9
	--disable-webm-io
	--disable-werror
	--target=${LIBVPX_TARGET}
	--prefix=${LIBVPX_INSTALL_PREFIX}
	--extra-cflags=${LIBVPX_EXTRA_C_FLAGS}
	--extra-cxxflags=${LIBVPX_EXTRA_CXX_FLAGS}
)

set(
	LIBVPX_ENV
	"CC=${LIBVPX_CC}"
	"CXX=${LIBVPX_CXX}"
	"LD=${LIBVPX_LD}"
	"AR=${LIBVPX_AR}"
	"NM=${LIBVPX_NM}"
	"STRIP=${LIBVPX_STRIP}"
	"RANLIB=${LIBVPX_RANLIB}"
)

if (NOT EXISTS "${LIBVPX_BUILD_DIRECTORY}")
	file(MAKE_DIRECTORY "${LIBVPX_BUILD_DIRECTORY}")
endif()

if (NOT EXISTS "${LIBVPX_BUILD_DIRECTORY}/Makefile")
	message("-- Configuring libvpx")
	
	execute_process(
		COMMAND ${CMAKE_COMMAND} -E env ${LIBVPX_ENV}
		bash "${LIBVPX_SOURCE_DIRECTORY}/configure"
		${LIBVPX_CONFIGURE_FLAGS}
		WORKING_DIRECTORY "${LIBVPX_BUILD_DIRECTORY}"
		COMMAND_ERROR_IS_FATAL ANY
	)
endif()

if (NOT EXISTS "${LIBVPX_INSTALL_PREFIX}")
	add_custom_command(
		OUTPUT "${LIBVPX_INSTALL_PREFIX}"
		COMMAND make
		COMMAND make install
		WORKING_DIRECTORY "${LIBVPX_BUILD_DIRECTORY}"
		COMMENT "-- Building libvpx"
	)
endif()

add_custom_target(libvpx ALL DEPENDS "${LIBVPX_INSTALL_PREFIX}")
