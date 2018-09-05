cmake_minimum_required(VERSION 3.0)

project(pigpio)

set(CMAKE_C_FLAGS "-O3 -Wall -pthread")
set(PIGPIO_FLAGS "-L. -lrt")

# libpigpio.so
add_library(pigpio SHARED pigpio.c command.c custom.cext)
set_property(TARGET pigpio
	PROPERTY POSITION_INDEPENDENT_CODE ON
)

# libpigpiod_if.so
add_library(pigpiod_if SHARED pigpiod_if.c command.c)
set_property(TARGET pigpiod_if
	PROPERTY POSITION_INDEPENDENT_CODE ON
)

# libpigpiod_if2.so
add_library(pigpiod_if2 SHARED pigpiod_if2.c command.c)
set_property(TARGET pigpiod_if2
	PROPERTY POSITION_INDEPENDENT_CODE ON
)

# x_pigpio
add_executable(x_pigpio x_pigpio.c)
add_dependencies(x_pigpio pigpio)
target_link_libraries(x_pigpio
	${PIGPIO_FLAGS}
	-lpigpio
)

# x_pigpiod_if
add_executable(x_pigpiod_if x_pigpiod_if.c)
add_dependencies(x_pigpiod_if pigpiod_if)
target_link_libraries(x_pigpiod_if
	${PIGPIO_FLAGS}
	-lpigpiod_if
)

# x_pigpiod_if2
add_executable(x_pigpiod_if2 x_pigpiod_if2.c)
add_dependencies(x_pigpiod_if2 pigpiod_if2)
target_link_libraries(x_pigpiod_if2
	${PIGPIO_FLAGS}
	-lpigpiod_if2
)

# pigpiod
add_executable(pigpiod pigpiod.c)
add_dependencies(pigpiod pigpio)
target_link_libraries(pigpiod
	${PIGPIO_FLAGS}
	-lpigpio
)

# pigs
add_executable(pigs pigs.c command.c)

# pig2vcd
add_executable(pig2vcd pig2vcd.c command.c)

# install
install(DIRECTORY
	DESTINATION ${CMAKE_INSTALL_PREFIX}/cgi
	PATTERN ""
	PERMISSIONS OWNER_EXECUTE OWNER_WRITE OWNER_READ
		GROUP_READ GROUP_EXECUTE
		WORLD_READ WORLD_EXECUTE
)

install(TARGETS pigpio pigpiod_if pigpiod_if2 pig2vcd pigpiod pigs
	LIBRARY DESTINATION ${CMAKE_INSTALL_PREFIX}/lib
	RUNTIME DESTINATION ${CMAKE_INSTALL_PREFIX}/bin
)

install(FILES pigpio.h pigpiod_if.h pigpiod_if2.h
	DESTINATION ${CMAKE_INSTALL_PREFIX}/include
	PERMISSIONS OWNER_READ OWNER_WRITE
		GROUP_READ
		WORLD_READ
)

file(GLOB man_1_SRC "*.1")
install(FILES ${man_1_SRC}
	DESTINATION ${CMAKE_INSTALL_PREFIX}/man/man1
	PERMISSIONS OWNER_READ OWNER_WRITE
		GROUP_READ
		WORLD_READ
)

file(GLOB man_3_SRC "*.3")
install(FILES ${man_3_SRC}
	DESTINATION ${CMAKE_INSTALL_PREFIX}/man/man3
	PERMISSIONS OWNER_READ OWNER_WRITE
		GROUP_READ
		WORLD_READ
)

file(GLOB setup_SRC "setup.py")
find_program(PYTHON2_FOUND python2)
if(PYTHON2_FOUND)
	install(CODE "execute_process(COMMAND cd ${CMAKE_SOURCE_DIR} && python2 ${setup_SRC} install)")
endif()
find_program(PYTHON3_FOUND python3)
if(PYTHON3_FOUND)
	install(CODE "execute_process(COMMAND cd ${CMAKE_SOURCE_DIR} && python3 ${setup_SRC} install)")
endif()

#install(CODE "execute_process(COMMAND ldconfig)")

# configuration

# Change the include location for the case of an install location
set(pigpio_include_dirs ${CMAKE_INSTALL_PREFIX}/include) 
set(pigpio_libraries ${CMAKE_INSTALL_PREFIX}/lib) 

# We put the generated file for installation in a different repository (i.e., ./CMakeFiles/)
configure_file(pigpioConfig.cmake.in
  "${PROJECT_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/pigpioConfig.cmake" @ONLY)

install(FILES
  "${PROJECT_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/pigpioConfig.cmake"
  DESTINATION share/pigpio/cmake COMPONENT dev)

# uninstall
if(PYTHON2_FOUND)
	set(PY2_CMD python2 ${setup_SRC} install --record /tmp/pigpio > /dev/null)
	set(PY2_CMD ${PY2_CMD} && xargs rm -f < /tmp/pigpio > /dev/null)
endif()

if(PYTHON3_FOUND)
	set(PY3_CMD python3 ${setup_SRC} install --record /tmp/pigpio > /dev/null)
	set(PY3_CMD ${PY3_CMD} && xargs rm -f < /tmp/pigpio > /dev/null)
endif()

add_custom_target(uninstall
	COMMAND rm -f ${CMAKE_INSTALL_PREFIX}/include/pigpio.h
	COMMAND rm -f ${CMAKE_INSTALL_PREFIX}/include/pigpiod_if.h
	COMMAND rm -f ${CMAKE_INSTALL_PREFIX}/include/pigpiod_if2.h
	COMMAND rm -f ${CMAKE_INSTALL_PREFIX}/lib/libpigpio.so
	COMMAND rm -f ${CMAKE_INSTALL_PREFIX}/lib/libpigpiod_if.so
	COMMAND rm -f ${CMAKE_INSTALL_PREFIX}/lib/libpigpiod_if2.so
	COMMAND rm -f ${CMAKE_INSTALL_PREFIX}/bin/pig2vcd
	COMMAND rm -f ${CMAKE_INSTALL_PREFIX}/bin/pigpiod
	COMMAND rm -f ${CMAKE_INSTALL_PREFIX}/bin/pigs
	COMMAND cd ${CMAKE_SOURCE_DIR} && ${PY2_CMD}
	COMMAND cd ${CMAKE_SOURCE_DIR} && ${PY3_CMD}
	COMMAND rm -f ${CMAKE_INSTALL_PREFIX}/man/man1/pig*.1
	COMMAND rm -f ${CMAKE_INSTALL_PREFIX}/man/man3/pig*.3
	COMMAND ldconfig
)
