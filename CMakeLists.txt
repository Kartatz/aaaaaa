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
	--prefix=${LIBVPX_INSTALL_PREFIX}
	--extra-cflags=${CMAKE_C_FLAGS}
	--extra-cxxflags=${CMAKE_CXX_FLAGS}
)

list(APPEND LIBVPX_CONFIGURE_FLAGS "LD=${CMAKE_C_COMPILER}")

list(APPEND LIBVPX_CONFIGURE_FLAGS "AR=${CMAKE_AR}")
list(APPEND LIBVPX_CONFIGURE_FLAGS "CC=${ANDROID_TOOLCHAIN_ROOT}/bin/${TOOLCHAIN_PREFIX}-clang")

if (NOT EXISTS "${LIBVPX_BUILD_DIRECTORY}")
	file(MAKE_DIRECTORY "${LIBVPX_BUILD_DIRECTORY}")
endif()

if (NOT EXISTS "${LIBVPX_BUILD_DIRECTORY}/Makefile")
	message("-- Configuring libvpx")
	
	execute_process(
		COMMAND bash
		"${LIBVPX_SOURCE_DIRECTORY}/configure"
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
