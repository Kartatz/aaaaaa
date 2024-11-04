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
	--disable-static \
	--enable-shared
	--enable-pic \
	--enable-libyuv \
	--enable-small \
	--enable-optimizations \
	--enable-better-hw-compatibility \
	--disable-examples \
	--disable-tools \
	--disable-debug \
	--disable-neon-asm \
	--disable-neon-dotprod \
	--disable-unit-tests \
	--disable-install-docs \
	--disable-docs \
	--enable-realtime-only \
	--enable-vp8 \
	--enable-vp9 \
	--disable-webm-io
	--prefix="${LIBVPX_INSTALL_PREFIX}"
	--extra-cflags="${CMAKE_C_FLAGS}"
	--extra-cxxflags="${CMAKE_CXX_FLAGS}"
)
