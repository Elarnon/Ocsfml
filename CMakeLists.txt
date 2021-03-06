cmake_minimum_required(VERSION 2.8)

set(SFML_STATIC_LIBRARIES FALSE CACHE BOOL "Use SFML static library version")
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/Configuration/Modules/")

find_package( 
	SFML 
	COMPONENTS system window graphics audio network
)

find_package(
	Boost
	REQUIRED
)

execute_process(
	COMMAND ocamlc -where
	OUTPUT_VARIABLE OCAML_LIB_DIR
)
string(STRIP ${OCAML_LIB_DIR} OCAML_LIB_DIR)

if (CMAKE_SIZEOF_VOID_P MATCHES "8")
    set(CAMLPP_ARCHITECTURE_SIXTYFOUR TRUE)
else()
    set(CAMLPP_ARCHITECTURE_SIXTYFOUR FALSE)
endif()


set( SFML_DIR ${SFML_INCLUDE_DIR} CACHE PATH "SFML 'include' directory path" )
set( SFML_SYSTEM ${SFML_SYSTEM_LIBRARY_RELEASE} CACHE FILEPATH "SFML system library path" )
set( SFML_WINDOW ${SFML_WINDOW_LIBRARY_RELEASE} CACHE FILEPATH "SFML window library path" )
set( SFML_GRAPHICS ${SFML_GRAPHICS_LIBRARY_RELEASE} CACHE FILEPATH "SFML graphics library path" )
set( SFML_AUDIO ${SFML_AUDIO_LIBRARY_RELEASE} CACHE FILEPATH "SFML audio library path" )
set( SFML_NETWORK ${SFML_NETWORK_LIBRARY_RELEASE} CACHE FILEPATH "SFML network library path" )
set( BOOST_DIR ${Boost_INCLUDE_DIR} CACHE PATH "Boost 'include' directory path" )
set( OCAML_DIR ${OCAML_LIB_DIR} CACHE PATH "Ocaml 'include' directory path (where 'caml' directory is)" )
set( OCSFML_DISTRIB_DIR "" CACHE PATH "What should be the ocsfml distrib directory")
set( CAMLPP_ARCHITECTURE_SIXTYFOUR ${CAMLPP_ARCHITECTURE_SIXTYFOUR}
CACHE BOOLEAN "Is the target 64-bits?" )

if(${CMAKE_SYSTEM_NAME} MATCHES "Darwin" OR ${CMAKE_SYSTEM_NAME} MATCHES "Linux")
set( OCSFML_LINK_BYTECODE_DYNAMIC TRUE CACHE BOOL "Link dynamically c++ primitives against byte-code executable" )
set( OCSFML_LINK_NATIVE_DYNAMIC TRUE CACHE BOOL "Link dynamically c++ primitives against native executable" 	 )
else()
set( OCSFML_LINK_BYTECODE_DYNAMIC FALSE CACHE BOOL "Link dynamically c++ primitives against byte-code executable" )
set( OCSFML_LINK_NATIVE_DYNAMIC FALSE CACHE BOOL "Link dynamically c++ primitives against native executable" )
endif()

if(${SFML_STATIC_LIBRARIES})
set(SFML_STATIC_LIBRARIES true)
else()
set(SFML_STATIC_LIBRARIES false)
endif()


if(${SFML_STATIC_LIBRARIES} OR ${CMAKE_SYSTEM_NAME} MATCHES "Windows")
set(LINK_SFML_TO_STATIC_LIB true)
else()
set(LINK_SFML_TO_STATIC_LIB false)
endif()


if(${OCSFML_LINK_BYTECODE_DYNAMIC})
set(OCSFML_LINK_BYTECODE_DYNAMIC true)
set(OCSFML_LINK_BYTECODE "-dllib" )
else()
set(OCSFML_LINK_BYTECODE_DYNAMIC false)
set(OCSFML_LINK_BYTECODE "-cclib" )
endif()

if(${OCSFML_LINK_NATIVE_DYNAMIC})
set(OCSFML_LINK_NATIVE_DYNAMIC true)
else()
set(OCSFML_LINK_NATIVE_DYNAMIC false)
endif()
set(OCSFML_LINK_NATIVE "-cclib" )

##
set( DEV_MODE FALSE)
set( CPP_DEV_MODE FALSE)
set( CAML_DEV_MODE FALSE)

set(CPP_COMPILER ${CMAKE_CXX_COMPILER})
set(EXTERNAL_CPP_INSTALL_PREFIX "${CMAKE_SOURCE_DIR}/camlpp")


if(MSVC_VERSION EQUAL 1600) #VC++ 2010
  include(Configuration/VC++2010.cmake)
elseif(MSVC_VERSION EQUAL 1700) #VC++ 2012
  include(Configuration/VC++2010.cmake)
elseif(CMAKE_COMPILER_IS_GNUCXX)
  include(Configuration/G++.cmake)
elseif("${CMAKE_CXX_COMPILER_ID}" MATCHES "Clang")
  include(Configuration/Clang.cmake)
endif()
      
macro(correct_osx_framework module_name)
  if("${SFML_${module_name}}" MATCHES ".*/([^/]+)\\.framework$")
    string( REGEX REPLACE ".*/([^/]+)\\.framework$" "${SFML_${module_name}}/\\1" SFML_${module_name} "${SFML_${module_name}}" )
  endif()
endmacro()



if(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
  set(DLL_EXTENSION "so")
  set(SCRIPT_EXTENSION "sh")
 # set( COMPILATION_FLAGS "${COMPILATION_FLAGS}; A \"-arch\" ; A \"${CMAKE_OSX_ARCHITECTURES}\"")
 # set( LINKING_DLL_FLAGS "${LINKING_DLL_FLAGS}; A \"-arch\"; A \"${CMAKE_OSX_ARCHITECTURES}\"")
 correct_osx_framework( SYSTEM )
 correct_osx_framework( WINDOW )
 correct_osx_framework( GRAPHICS )
 correct_osx_framework( NETWORK )
 correct_osx_framework( AUDIO )
elseif(${CMAKE_SYSTEM_NAME} MATCHES "Linux")
  set(DLL_EXTENSION "so")
  set(SCRIPT_EXTENSION "sh")
elseif(${CMAKE_SYSTEM_NAME} MATCHES "Windows")
  set(DLL_EXTENSION "dll")
  set(SCRIPT_EXTENSION "bat")
endif()


configure_file( ${CMAKE_SOURCE_DIR}/myocamlbuild.ml.in ${CMAKE_SOURCE_DIR}/myocamlbuild.ml)
configure_file( ${CMAKE_SOURCE_DIR}/install.in ${CMAKE_SOURCE_DIR}/install)

# configure_file(${CMAKE_SOURCE_DIR}/OcsfmlSystem/ocsfml_system_stub.hpp ${CMAKE_SOURCE_DIR}/OcsfmlWindow/ocsfml_system_stub.hpp COPYONLY)
# configure_file(${CMAKE_SOURCE_DIR}/OcsfmlSystem/ocsfml_system_stub.hpp ${CMAKE_SOURCE_DIR}/OcsfmlGraphics/ocsfml_system_stub.hpp COPYONLY)
# configure_file(${CMAKE_SOURCE_DIR}/OcsfmlSystem/ocsfml_system_stub.hpp ${CMAKE_SOURCE_DIR}/OcsfmlNetwork/ocsfml_system_stub.hpp COPYONLY)
# configure_file(${CMAKE_SOURCE_DIR}/OcsfmlSystem/ocsfml_system_stub.hpp ${CMAKE_SOURCE_DIR}/OcsfmlAudio/ocsfml_system_stub.hpp COPYONLY)
# configure_file(${CMAKE_SOURCE_DIR}/OcsfmlWindow/ocsfml_window_stub.hpp ${CMAKE_SOURCE_DIR}/OcsfmlGraphics/ocsfml_window_stub.hpp COPYONLY)

configure_file(${CMAKE_SOURCE_DIR}/_tags.in ${CMAKE_SOURCE_DIR}/_tags)
configure_file(${CMAKE_SOURCE_DIR}/OcsfmlSystem/ocsfmlsystem.cpplib.in ${CMAKE_SOURCE_DIR}/OcsfmlSystem/ocsfmlsystem.cpplib)
configure_file(${CMAKE_SOURCE_DIR}/OcsfmlWindow/ocsfmlwindow.cpplib.in ${CMAKE_SOURCE_DIR}/OcsfmlWindow/ocsfmlwindow.cpplib)
configure_file(${CMAKE_SOURCE_DIR}/OcsfmlGraphics/ocsfmlgraphics.cpplib.in ${CMAKE_SOURCE_DIR}/OcsfmlGraphics/ocsfmlgraphics.cpplib)
configure_file(${CMAKE_SOURCE_DIR}/OcsfmlAudio/ocsfmlaudio.cpplib.in ${CMAKE_SOURCE_DIR}/OcsfmlAudio/ocsfmlaudio.cpplib)
configure_file(${CMAKE_SOURCE_DIR}/OcsfmlNetwork/ocsfmlnetwork.cpplib.in ${CMAKE_SOURCE_DIR}/OcsfmlNetwork/ocsfmlnetwork.cpplib)

set(OCAMLBUILD_COMMAND "ocamlbuild")

include_directories(
  ${EXTERNAL_CPP_INSTALL_PREFIX}
  ${OCAML_DIR}
  ${SFML_DIR}
  ${BOOST_DIR}
  )


add_subdirectory(OcsfmlSystem)
add_subdirectory(OcsfmlWindow)
add_subdirectory(OcsfmlGraphics)
add_subdirectory(OcsfmlAudio)
add_subdirectory(OcsfmlNetwork)
add_subdirectory(Test)

add_custom_target( 
	clear
	${OCAMLBUILD_COMMAND} -clean
	WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
)

add_custom_target(
	ocsfml
	ALL	${OCAMLBUILD_COMMAND} -use-ocamlfind ocsfmlsystem.cma
	COMMAND ${OCAMLBUILD_COMMAND} -use-ocamlfind ocsfmlsystem.cmxa
	COMMAND	${OCAMLBUILD_COMMAND} -use-ocamlfind ocsfmlwindow.cma
	COMMAND	${OCAMLBUILD_COMMAND} -use-ocamlfind ocsfmlwindow.cmxa
	COMMAND	${OCAMLBUILD_COMMAND} -use-ocamlfind ocsfmlgraphics.cma 
	COMMAND	${OCAMLBUILD_COMMAND} -use-ocamlfind ocsfmlgraphics.cmxa
	COMMAND	${OCAMLBUILD_COMMAND} -use-ocamlfind ocsfmlnetwork.cma
	COMMAND	${OCAMLBUILD_COMMAND} -use-ocamlfind ocsfmlnetwork.cmxa
	COMMAND	${OCAMLBUILD_COMMAND} -use-ocamlfind ocsfmlaudio.cma
	COMMAND	${OCAMLBUILD_COMMAND} -use-ocamlfind ocsfmlaudio.cmxa
	WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
)


  

macro(ocsfml_library dir name)
  set(
    ${name} 
    _build/Ocsfml${dir}/ocsfml${dir}.cmi 
    _build/Ocsfml${dir}/dllocsfml${name}.${DLL_EXTENSION}  
    _build/Ocsfml${dir}/ocsfml${name}.cma 
    _build/Ocsfml${dir}/libocsfml${name}.${LIB_EXTENSION} 
    _build/Ocsfml${dir}/ocsfml${name}.cmxa 
    _build/Ocsfml${dir}/ocsfml${name}.${LIB_EXTENSION}
    )
endmacro()

ocsfml_library(System system) 
ocsfml_library(Window window) 
ocsfml_library(Graphics graphics) 
ocsfml_library(Audio audio) 
ocsfml_library(Network network)

add_custom_target(
  install
  ocamlfind install ocsfml META ${system} ${window} ${graphics} ${audio} ${network}
  WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
  )

add_custom_target(
  uninstall
  ocamlfind remove "ocsfml"
  )

add_custom_target(
  uninstall_ocsfml
  ${CMAKE_COMMAND} -E remove_directory "${TESTS_INSTALL_PREFIX}"
  WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
  )

add_dependencies(
  uninstall_ocsfml
  uninstall_external_cpp
  uninstall
  uninstall_tests
  )

add_custom_target(
  doc
  ocamlbuild -use-ocamlfind ocsfml.docdir/index.html
  WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
  )

add_custom_target(
  distribuable
  ${CMAKE_COMMAND} -E remove_directory ${OCSFML_DISTRIB_DIR}/
  COMMAND ${CMAKE_COMMAND} -E make_directory ${OCSFML_DISTRIB_DIR}/
  COMMAND ${CMAKE_COMMAND} -E make_directory ${OCSFML_DISTRIB_DIR}/System/
  COMMAND ${CMAKE_COMMAND} -E make_directory ${OCSFML_DISTRIB_DIR}/Window/
  COMMAND ${CMAKE_COMMAND} -E make_directory ${OCSFML_DISTRIB_DIR}/Graphics/
  COMMAND ${CMAKE_COMMAND} -E make_directory ${OCSFML_DISTRIB_DIR}/Audio/
  COMMAND ${CMAKE_COMMAND} -E make_directory ${OCSFML_DISTRIB_DIR}/Network/
  COMMAND ${CMAKE_COMMAND} -E copy_directory _build/OcsfmlSystem/ ${OCSFML_DISTRIB_DIR}/System/
  COMMAND ${CMAKE_COMMAND} -E copy_directory _build/OcsfmlWindow/ ${OCSFML_DISTRIB_DIR}/Window/
  COMMAND ${CMAKE_COMMAND} -E copy_directory _build/OcsfmlGraphics/ ${OCSFML_DISTRIB_DIR}/Graphics/
  COMMAND ${CMAKE_COMMAND} -E copy_directory _build/OcsfmlAudio/ ${OCSFML_DISTRIB_DIR}/Audio/
  COMMAND ${CMAKE_COMMAND} -E copy_directory _build/OcsfmlNetwork/ ${OCSFML_DISTRIB_DIR}/Network/
  COMMAND ${CMAKE_COMMAND} -E copy META ${OCSFML_DISTRIB_DIR}/
  COMMAND ${CMAKE_COMMAND} -E copy install ${OCSFML_DISTRIB_DIR}/install.@SCRIPT_EXTENSION@
  COMMAND rm ${OCSFML_DISTRIB_DIR}/*/*.${OBJ_EXTENSION}
  COMMAND rm ${OCSFML_DISTRIB_DIR}/*/*.cmo
  COMMAND rm ${OCSFML_DISTRIB_DIR}/*/*.cmx
  COMMAND rm ${OCSFML_DISTRIB_DIR}/*/*.ml*
  COMMAND rm ${OCSFML_DISTRIB_DIR}/*/*.cpplib
  COMMAND rm -R ${OCSFML_DISTRIB_DIR}/*/ocsfml_*_stub/
  WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
  )
