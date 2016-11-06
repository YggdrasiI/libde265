set (arm_sources
	arm.cc arm.h
)

set(neon_asm_files
	asm.S
	cpudetect.S
	hevcdsp_qpel_neon.S
	neon.S
	)

add_library(arm OBJECT ${arm_sources})

if(ENABLE_ARM_THUMB)
	if(NOT MSVC)
		set(CCAS_FLAGS "-DCONFIG_THUMB ${CCAS_FLAGS}")
		set(CMAKE_CXX_FLAGS "-mthumb ${CMAKE_CXX_FLAGS}")
	endif()
endif()

if(HAVE_VISIBILITY)
	set(CCAS_FLAGS "-DHAVE_VISIBILITY ${CCAS_FLAGS}")
endif()

set(ARM_OBJECTS $<TARGET_OBJECTS:arm> PARENT_SCOPE)

if(SUPPORTS_NEON)
	add_definitions(-DHAVE_NEON)
	set(CMAKE_CXX_FLAGS "-mfpu=neon ${CMAKE_CXX_FLAGS}")
	add_library(arm_neon OBJECT ${neon_asm_files})

	# Flags copied from Makefile.am
	set(CCAS_FLAGS "-DHAVE_NEON -DEXTERN_ASM= -DHAVE_AS_FUNC -DHAVE_SECTION_DATA_REL_RO ${CCAS_FLAGS}")

	# Use gcc for ASM compiling
	set_property(SOURCE ${neon_asm_files} PROPERTY LANGUAGE C)
	set_source_files_properties(${neon_asm_files} PROPERTIES COMPILE_FLAGS ${CCAS_FLAGS})

	set(ARM_OBJECTS $<TARGET_OBJECTS:arm> $<TARGET_OBJECTS:arm_neon> PARENT_SCOPE)
endif()
