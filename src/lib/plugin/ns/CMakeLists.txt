# synergy -- mouse and keyboard sharing utility
# Copyright (C) 2015 Synergy Si Ltd.
# 
# This package is free software; you can redistribute it and/or
# modify it under the terms of the GNU General Public License
# found in the file COPYING that should have accompanied this file.
# 
# This package is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program.  If not, see <http://www.gnu.org/licenses/>.

file(GLOB headers "*.h")
file(GLOB sources "*.cpp")

if (SYNERGY_ADD_HEADERS)
	list(APPEND sources ${headers})
endif()

if (WIN32)
	set(OPENSSL_INCLUDE ../../../../ext/openssl/inc32)
endif()

if (APPLE)
	set(OPENSSL_INCLUDE ../../../../ext/openssl/include)
endif()

include_directories(
	../../../lib/
	../../../..
	${OPENSSL_INCLUDE}
)

add_library(ns SHARED ${sources})

if (WIN32)
  set(OPENSSL_LIBS
    ${CMAKE_SOURCE_DIR}/ext/openssl/out32dll/libeay32.lib
    ${CMAKE_SOURCE_DIR}/ext/openssl/out32dll/ssleay32.lib
  )
endif()

if (UNIX)
  if (APPLE)
    set(OPENSSL_LIBS
	  ${CMAKE_SOURCE_DIR}/ext/openssl/libssl.a
	  ${CMAKE_SOURCE_DIR}/ext/openssl/libcrypto.a
    )
  else()
    set(OPENSSL_LIBS ssl crypto)
  endif()
endif()

target_link_libraries(ns
	arch base client common io mt net ipc platform server synergy ${libs} ${OPENSSL_LIBS})

if (WIN32)
	add_custom_command(
		TARGET ns 
		POST_BUILD
		COMMAND xcopy /Y /Q
			..\\..\\..\\..\\..\\lib\\${CMAKE_CFG_INTDIR}\\ns.dll
			..\\..\\..\\..\\..\\bin\\${CMAKE_CFG_INTDIR}\\plugins\\
	)
else()
	if (APPLE)
		add_custom_command(
			TARGET ns
			POST_BUILD
			COMMAND
				mkdir -p
				${CMAKE_SOURCE_DIR}/bin/plugins
				&&
				cp
				${CMAKE_SOURCE_DIR}/lib/${CMAKE_CFG_INTDIR}/libns.dylib
				${CMAKE_SOURCE_DIR}/bin/plugins/
		)
	else()
		if (CMAKE_BUILD_TYPE STREQUAL Debug)
			add_custom_command(
				TARGET ns
				POST_BUILD
				COMMAND mkdir -p
					${CMAKE_SOURCE_DIR}/bin/plugins
					&&
					cp
					${CMAKE_SOURCE_DIR}/lib/debug/libns.so
					${CMAKE_SOURCE_DIR}/bin/plugins/
			)
		else()
			add_custom_command(
				TARGET ns
				POST_BUILD
				COMMAND mkdir -p
					${CMAKE_SOURCE_DIR}/bin/plugins
					&&
					cp
					${CMAKE_SOURCE_DIR}/lib/libns.so
					${CMAKE_SOURCE_DIR}/bin/plugins/
			)
		endif()
	endif()
endif()
