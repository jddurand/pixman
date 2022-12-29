include(GenerateExportHeader)

set(SOURCES
    pixman.c
    pixman-access.c
    pixman-access-accessors.c
    pixman-bits-image.c
    pixman-combine32.c
    pixman-combine-float.c
    pixman-conical-gradient.c
    pixman-filter.c
    pixman-x86.c
    pixman-mips.c
    pixman-arm.c
    pixman-ppc.c
    pixman-edge.c
    pixman-edge-accessors.c
    pixman-fast-path.c
    pixman-glyph.c
    pixman-general.c
    pixman-gradient-walker.c
    pixman-image.c
    pixman-implementation.c
    pixman-linear-gradient.c
    pixman-matrix.c
    pixman-noop.c
    pixman-radial-gradient.c
    pixman-region16.c
    pixman-region32.c
    pixman-solid-fill.c
    pixman-timer.c
    pixman-trap.c
    pixman-utils.c
	dither/make-blue-noise.c
)

set(HEADERS
    pixman.h
    pixman-accessor.h
    pixman-combine32.h
    pixman-compiler.h
    pixman-edge-imp.h
    pixman-inlines.h
    pixman-private.h
	dither/blue-noise-64x64.h
)

#proccesor_optimizations
if(ARM)
    if (ARM_NEON)
        add_definitions(-DUSE_ARM_NEON)
        list(APPEND SOURCES pixman-arm-neon.c
                            pixman-arm-neon-asm.S
                            pixman-arm-neon-asm-bilinear.S
        )
    endif (ARM_NEON)
    if (ARM_SIMD)
        add_definitions(-DUSE_ARM_SIMD)
        list(APPEND SOURCES pixman-arm-simd.c
                            pixman-arm-simd-asm.S
                            pixman-arm-simd-asm-scaled.S
        )
    endif (ARM_SIMD)
    if (ARM_IWMMXT)
        add_definitions(-DUSE_ARM_IWMMXT)
        list(APPEND SOURCES "pixman-mmx.c")
    endif()
endif(ARM)   

if(MIPS)
    if (MIPS_DSPR2)
        add_definitions(-DUSE_MIPS_DSPR2)
        list(APPEND SOURCES pixman-mips-dspr2.c
                            pixman-mips-dspr2-asm.S
                            pixman-mips-memcpy-asm.S
        )
    endif (MIPS_DSPR2)
    if (MIPS_LOONGSON_MMI)
        add_definitions(-DUSE_LOONGSON_MMI)
        list(APPEND SOURCES "pixman-mmx.c")
    endif()
endif(MIPS)  

if(PPC)
    if (PPC_VMX)
        add_definitions(-DUSE_VMX)
        list(APPEND SOURCES "pixman-vmx.c")
    endif (PPC_VMX)
endif(PPC)  

if(X86)
    if (X86_MMX)
        add_definitions(-DUSE_X86_MMX)
        list(APPEND SOURCES "pixman-mmx.c")
    endif (X86_MMX)
    if (X86_SSE2)
        add_definitions(-DUSE_SSE2)
        list(APPEND SOURCES "pixman-sse2.c")
    endif (X86_SSE2)
    if (X86_SSSE3)
        add_definitions(-DUSE_SSSE3)
        list(APPEND SOURCES "pixman-ssse3.c")
    endif(X86_SSSE3)
endif(X86)

#
# Note that position independant flags is ON by default for SHARED, MODULE and EXECUTABLE
#

if(BUILD_SHARED)
    add_library(pixman-1 SHARED ${SOURCES} ${HEADERS})
    install(TARGETS pixman-1 EXPORT PixmanTargets RUNTIME DESTINATION bin ARCHIVE DESTINATION lib LIBRARY DESTINATION lib)
endif()

if(BUILD_STATIC)
    add_library(pixman-1_static STATIC ${SOURCES} ${HEADERS})
    install(TARGETS pixman-1_static EXPORT PixmanTargets RUNTIME DESTINATION bin ARCHIVE DESTINATION lib LIBRARY DESTINATION lib)
endif(BUILD_STATIC)

#
# We generate the same export header for both
#
if(BUILD_SHARED)
	set(export_target pixman-1)
else()
	set(export_target pixman-1_static)
endif()
set(CMAKE_CXX_VISIBILITY_PRESET hidden)
set(CMAKE_VISIBILITY_INLINES_HIDDEN 1)
generate_export_header(${export_target}
	BASE_NAME PIXMAN_1
	EXPORT_FILE_NAME pixman-export.h
	EXPORT_MACRO_NAME PIXMAN_API
	NO_EXPORT_MACRO_NAME PIXMAN_API_NO_EXPORT)
if(BUILD_SHARED)
	target_compile_definitions(pixman-1 PRIVATE -Dpixman_1_EXPORTS)
endif()
if(BUILD_STATIC)
	target_compile_definitions(pixman-1_static PUBLIC -DPIXMAN_1_STATIC_DEFINE)
endif()
install(FILES pixman.h ${CMAKE_CURRENT_BINARY_DIR}/pixman-version.h ${CMAKE_CURRENT_BINARY_DIR}/pixman-export.h DESTINATION include/pixman-1)
