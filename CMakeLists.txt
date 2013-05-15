cmake_minimum_required (VERSION 2.8)

MESSAGE("Source Dir: " ${CMAKE_SOURCE_DIR})
MESSAGE("Build Dir: " ${CMAKE_BINARY_DIR})

include(ExternalProject)

#Use the google protocol buffers library
ExternalProject_Add(
	"gpb"
	PREFIX "${CMAKE_BINARY_DIR}/ext/protobuf"
	URL "https://protobuf.googlecode.com/files/protobuf-2.5.0.tar.gz"
	CONFIGURE_COMMAND <SOURCE_DIR>/configure --prefix=<INSTALL_DIR>
)

include_directories(${CMAKE_BINARY_DIR}/ext/protobuf/include)
link_directories(${CMAKE_BINARY_DIR}/ext/protobuf/lib)

#Directory to store protocol parsing files
file(MAKE_DIRECTORY "${CMAKE_BINARY_DIR}/interim")

add_custom_command(
	OUTPUT "${CMAKE_BINARY_DIR}/interim/fcgi.pb.h" "${CMAKE_BINARY_DIR}/interim/fcgi.pb.cc"
	COMMAND "${CMAKE_BINARY_DIR}/ext/protobuf/bin/protoc" "${CMAKE_SOURCE_DIR}/ifs/fcgi.proto" -I "${CMAKE_SOURCE_DIR}/ifs" --cpp_out "${CMAKE_BINARY_DIR}/interim"
	DEPENDS gpb "${CMAKE_SOURCE_DIR}/ifs/fcgi.proto"
	COMMENT "Compiling Protocol file"
)

add_executable(fcgi src/cxx/main "${CMAKE_SOURCE_DIR}/ifs/fcgi.proto" "${CMAKE_BINARY_DIR}/interim/fcgi.pb.cc" "${CMAKE_BINARY_DIR}/interim/fcgi.pb.h")
target_link_libraries(fcgi protobuf)
