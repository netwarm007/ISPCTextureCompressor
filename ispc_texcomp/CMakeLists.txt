set(FUNCTIONS kernel_astc kernel)

foreach(FUNC IN LISTS FUNCTIONS)
set(ISPC_OBJECTS ${FUNC}_ispc.o)
if(NOT ISPC_SINGLE_TARGET)
    list(APPEND ISPC_OBJECTS ${FUNC}_ispc_avx.o ${FUNC}_ispc_sse2.o)
endif()
list(APPEND ALL_ISPC_OBJECTS ${ISPC_OBJECTS})
add_custom_command(OUTPUT ${ISPC_OBJECTS}
    COMMAND ${CMAKE_COMMAND} -E env "PATH=${ISPC_COMPILER_PATH}" ${ISPC_COMPILER} ${ISPC_OPTIONS} -o ${FUNC}_ispc.o -h ${FUNC}_ispc.h ${CMAKE_CURRENT_SOURCE_DIR}/${FUNC}.ispc
    MAIN_DEPENDENCY ${CMAKE_CURRENT_SOURCE_DIR}/${FUNC}.ispc
    COMMENT "Building ${FUNC}"
    )
endforeach(FUNC)

add_library(ispc_texcomp ispc_texcomp.cpp ispc_texcomp_astc.cpp ${ALL_ISPC_OBJECTS})

set_target_properties(ispc_texcomp
PROPERTIES PUBLIC_HEADER ispc_texcomp.h
)

target_include_directories(ispc_texcomp PRIVATE ${CMAKE_CURRENT_BINARY_DIR})

install(TARGETS ispc_texcomp
    ARCHIVE DESTINATION ${CMAKE_SYSTEM_NAME}/lib
    PUBLIC_HEADER DESTINATION ${CMAKE_SYSTEM_NAME}/include
)
