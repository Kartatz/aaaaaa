cmake_minimum_required(VERSION 3.19)

project(
	something
	VERSION 0.1
	DESCRIPTION "..."
	HOMEPAGE_URL "..."
	LANGUAGES C
)

set(CMAKE_POLICY_DEFAULT_CMP0069 NEW)
set(CMAKE_POLICY_DEFAULT_CMP0048 NEW)
set(CMAKE_POLICY_DEFAULT_CMP0077 NEW)

set(CMAKE_PLATFORM_NO_VERSIONED_SONAME ON)


set(LIBVPX_SOURCE_DIRECTORY "${CMAKE_SOURCE_DIR}/submodules/libvpx")
set(LIBVPX_BUILD_DIRECTORY "${CMAKE_BINARY_DIR}/libvpx-build")
set(LIBVPX_INSTALL_PREFIX "${LIBVPX_BUILD_DIRECTORY}/binaries")

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
	"${CMAKE_C_FLAGS} -Wno-macro-redefined"
)

set(
	LIBVPX_EXTRA_CXX_FLAGS
	"${CMAKE_CXX_FLAGS} -Wno-macro-redefined"
)

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

message("CC=${ANDROID_TOOLCHAIN_ROOT}/bin/${TOOLCHAIN_PREFIX}-clang")

set(
	LIBVPX_ENV
	"LD=${CMAKE_C_COMPILER}"
	"AR=${CMAKE_AR}"
	"CC=${ANDROID_TOOLCHAIN_ROOT}/bin/${TOOLCHAIN_PREFIX}-clang"
	"CXX=${ANDROID_TOOLCHAIN_ROOT}/bin/${TOOLCHAIN_PREFIX}-clang++"
	"NM=${CMAKE_NM}"
	"STRIP=${CMAKE_STRIP}"
	"RANLIB=${CMAKE_RANLIB}"
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
