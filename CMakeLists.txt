cmake_minimum_required(VERSION 3.0)

project(pigpio)

set(CMAKE_C_FLAGS "-O3 -Wall -pthread")
set(PIGPIO_FLAGS "-L. -lrt")
#set(DESTDIR ${CMAKE_CURRENT_SOURCE_DIR}/build/dest)

if(NOT DEFINED BUILD_SHARED_LIBS)
set(BUILD_SHARED_LIBS "ON")
endif(NOT DEFINED BUILD_SHARED_LIBS)

# libpigpio.(so|a)
add_library(pigpio pigpio.c command.c custom.cext)

# libpigpiod_if.(so|a)
add_library(pigpiod_if pigpiod_if.c command.c)

# libpigpiod_if2.(so|a)
add_library(pigpiod_if2 pigpiod_if2.c command.c)


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

# construct install strings
# otherwise built makefiles contain $ENV{DESTDIR} instead of the literal build dir
# and that chokes skbuild:
# https://github.com/scikit-build/scikit-build/blob/0c33f15b0c0452e8bd70ab6057b4d283d6025b51/skbuild/cmaker.py#L427

string(CONCAT DESTDIR_CGI ${DESTDIR} "/opt/pigpio/cgi")


# install
install(DIRECTORY
	DESTINATION autopilot/external/pigpio/cgi
	PATTERN ""
	PERMISSIONS OWNER_EXECUTE OWNER_WRITE OWNER_READ
		GROUP_READ GROUP_EXECUTE
		WORLD_READ WORLD_EXECUTE
)

install(TARGETS pigpio pigpiod_if pigpiod_if2 pig2vcd pigpiod pigs
	LIBRARY DESTINATION autopilot/external/pigpio/lib
	RUNTIME DESTINATION autopilot/external/pigpio
	ARCHIVE DESTINATION autopilot/external/pigpio/lib
)

install(FILES pigpio.h pigpiod_if.h pigpiod_if2.h
	DESTINATION autopilot/external/pigpio/include
	PERMISSIONS OWNER_READ OWNER_WRITE
		GROUP_READ
		WORLD_READ
)

file(GLOB man_1_SRC "*.1")
install(FILES ${man_1_SRC}
	DESTINATION autopilot/external/pigpio/man/man1
	PERMISSIONS OWNER_READ OWNER_WRITE
		GROUP_READ
		WORLD_READ
)

file(GLOB man_3_SRC "*.3")
install(FILES ${man_3_SRC}
	DESTINATION autopilot/external/pigpio/man/man3
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

install(CODE "execute_process(COMMAND ldconfig)")

# uninstall
if(PYTHON2_FOUND)
	set(PY2_CMD python2 ${setup_SRC} install --record ../../autopilot/external/pigpio/tmp/pigpio > /dev/null)
	set(PY2_CMD ${PY2_CMD} && xargs rm -f < ../../autopilot/external/pigpio/tmp/pigpio > /dev/null)
endif()

if(PYTHON3_FOUND)
	set(PY3_CMD python3 ${setup_SRC} install --record ../../autopilot/external/pigpio/tmp/pigpio > /dev/null)
	set(PY3_CMD ${PY3_CMD} && xargs rm -f < ../../autopilot/external/pigpio/tmp/pigpio > /dev/null)
endif()

add_custom_target(uninstall
	COMMAND rm -f autopilot/external/pigpio/include/pigpio.h
	COMMAND rm -f autopilot/external/pigpio/include/pigpiod_if.h
	COMMAND rm -f autopilot/external/pigpio/include/pigpiod_if2.h
	COMMAND rm -f autopilot/external/pigpio/lib/libpigpio.so
	COMMAND rm -f autopilot/external/pigpio/lib/libpigpiod_if.so
	COMMAND rm -f autopilot/external/pigpio/libpigpiod_if2.so
	COMMAND rm -f autopilot/external/pigpio/pig2vcd
	COMMAND rm -f autopilot/external/pigpio/pigpiod
	COMMAND rm -f autopilot/external/pigpio/pigs
	COMMAND cd ${CMAKE_SOURCE_DIR} && ${PY2_CMD}
	COMMAND cd ${CMAKE_SOURCE_DIR} && ${PY3_CMD}
	COMMAND rm -f autopilot/external/pigpio/man/man1/pig*.1
	COMMAND rm -f autopilot/external/pigpio/man/man3/pig*.3
	COMMAND ldconfig
)
