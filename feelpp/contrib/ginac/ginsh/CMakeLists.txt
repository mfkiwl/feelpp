include_directories(
	${CMAKE_CURRENT_SOURCE_DIR}/../ginac
	${CMAKE_CURRENT_BINARY_DIR}/../ginac
	${CMAKE_CURRENT_SOURCE_DIR}
	${CMAKE_CURRENT_BINARY_DIR})
add_definitions(-DIN_GINAC)

bison_target(ginsh_parser
             ginsh_parser.ypp
	     ${CMAKE_CURRENT_BINARY_DIR}/ginsh_parser.cpp)
flex_target(ginsh_lexer
            ginsh_lexer.lpp
	    ${CMAKE_CURRENT_BINARY_DIR}/ginsh_lexer.cpp)
add_flex_bison_dependency(ginsh_lexer ginsh_parser)

set(ginsh_SOURCES
    ginsh_parser.cpp
    ginsh_lexer.cpp
)

set(ginsh_HEADERS
    ginsh_parser.hpp
    ginsh.h
    ginsh_fcn_help.h
    ginsh_op_help.h
)

set(ginsh_DISTRIB
    ${ginsh_SOURCES}
    ${ginsh_HEADERS}
    ginsh_parser.ypp
    ginsh_lexer.lpp
    ginsh.1.in
    ginsh_fcn_help.py
    ginsh_op_help.py
)
if (READLINE_FOUND)
	include_directories(${READLINE_INCLUDE_DIRS})
endif()

add_custom_command(
	OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/ginsh_fcn_help.h
	COMMAND ${Python3_EXECUTABLE} ${CMAKE_CURRENT_SOURCE_DIR}/ginsh_fcn_help.py -o ginsh_fcn_help.h ${CMAKE_CURRENT_SOURCE_DIR}/ginsh.1.in
	DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/ginsh.1.in ${CMAKE_CURRENT_SOURCE_DIR}/ginsh_fcn_help.py
	WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR})

add_custom_command(
	OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/ginsh_op_help.h
	COMMAND ${Python3_EXECUTABLE} ${CMAKE_CURRENT_SOURCE_DIR}/ginsh_op_help.py -o ginsh_op_help.h ${CMAKE_CURRENT_SOURCE_DIR}/ginsh.1.in
	DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/ginsh.1.in ${CMAKE_CURRENT_SOURCE_DIR}/ginsh_op_help.py
	WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR})

add_custom_command(
	OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/ginsh_parser.hpp
	DEPENDS ${BISON_ginsh_parser_OUTPUTS}
	WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR})

set(ginsh_extra_libs)
if (READLINE_FOUND)
	set(ginsh_extra_libs ${READLINE_LIBRARIES})
endif()

add_executable(ginsh ${ginsh_SOURCES} ${ginsh_HEADERS})
target_link_libraries(ginsh feelpp_ginac ${ginsh_extra_libs} ${LIBDL_LIBRARIES})
install(TARGETS ginsh RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}")
