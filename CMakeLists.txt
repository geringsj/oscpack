IF(BIICODE)
  include(${CMAKE_HOME_DIRECTORY}/biicode.cmake)

  IF(WIN32)
  list(REMOVE_ITEM  BII_LIB_SRC ip/posix/NetworkingUtils.cpp ip/posix/UdpSocket.cpp)
  
  ELSE()
  list(REMOVE_ITEM  BII_LIB_SRC ip/win32/NetworkingUtils.cpp ip/win32/UdpSocket.cpp)

  ENDIF()

  # Include base block dirU
  ADD_BIICODE_TARGETS()

  IF(WIN32)
  TARGET_LINK_LIBRARIES(${BII_LIB_TARGET} PUBLIC winmm)

  ENDIF()
  TARGET_INCLUDE_DIRECTORIES(${BII_LIB_TARGET} PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})

 
ELSE()
	cmake_minimum_required(VERSION 2.6)
	PROJECT(TestOscpack)

	INCLUDE_DIRECTORIES(${CMAKE_SOURCE_DIR})

	# separate versions of NetworkingUtils.cpp and UdpSocket.cpp are provided for Win32 and POSIX
	# the IpSystemTypePath selects the correct ones based on the current platform

	IF(WIN32)
	 set(IpSystemTypePath ip/win32)
	ELSE(WIN32)
	 set(IpSystemTypePath ip/posix)
	ENDIF(WIN32)

	ADD_LIBRARY(oscpack STATIC

	ip/IpEndpointName.h
	ip/IpEndpointName.cpp

	ip/NetworkingUtils.h
	${IpSystemTypePath}/NetworkingUtils.cpp

	ip/UdpSocket.h
	${IpSystemTypePath}/UdpSocket.cpp

	ip/PacketListener.h
	ip/TimerListener.h

	osc/OscTypes.h
	osc/OscTypes.cpp 
	osc/OscHostEndianness.h
	osc/OscException.h
	osc/OscPacketListener.h
	osc/MessageMappingOscPacketListener.h
	osc/OscReceivedElements.h
	osc/OscReceivedElements.cpp
	osc/OscPrintReceivedElements.h
	osc/OscPrintReceivedElements.cpp
	osc/OscOutboundPacketStream.h
	osc/OscOutboundPacketStream.cpp

	)
	
	target_include_directories(oscpack PRIVATE ${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/ip ${CMAKE_CURRENT_SOURCE_DIR}/osc)
	IF(WIN32)
	 #set(LIBS ${LIBS} Ws2_32 winmm)
	 target_link_libraries(oscpack PRIVATE Ws2_32 winmm)
	ENDIF(WIN32)

	ADD_EXECUTABLE(OscUnitTests tests/OscUnitTests.cpp)
	TARGET_LINK_LIBRARIES(OscUnitTests oscpack ${LIBS})

	ADD_EXECUTABLE(OscSendTests tests/OscSendTests.cpp)
	TARGET_LINK_LIBRARIES(OscSendTests oscpack ${LIBS})

	ADD_EXECUTABLE(OscReceiveTest tests/OscReceiveTest.cpp)
	TARGET_LINK_LIBRARIES(OscReceiveTest oscpack ${LIBS})


	ADD_EXECUTABLE(OscDump examples/OscDump.cpp)
	TARGET_LINK_LIBRARIES(OscDump oscpack ${LIBS})

	ADD_EXECUTABLE(SimpleReceive examples/SimpleReceive.cpp)
	TARGET_LINK_LIBRARIES(SimpleReceive oscpack ${LIBS})

	ADD_EXECUTABLE(SimpleSend examples/SimpleSend.cpp)
	TARGET_LINK_LIBRARIES(SimpleSend oscpack ${LIBS})
	
	
	target_include_directories(OscUnitTests PRIVATE ${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/ip ${CMAKE_CURRENT_SOURCE_DIR}/osc)
	target_include_directories(OscSendTests PRIVATE ${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/ip ${CMAKE_CURRENT_SOURCE_DIR}/osc)
	target_include_directories(OscReceiveTest PRIVATE ${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/ip ${CMAKE_CURRENT_SOURCE_DIR}/osc)
	target_include_directories(OscDump PRIVATE ${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/ip ${CMAKE_CURRENT_SOURCE_DIR}/osc)
	target_include_directories(SimpleReceive PRIVATE ${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/ip ${CMAKE_CURRENT_SOURCE_DIR}/osc)
	target_include_directories(SimpleSend PRIVATE ${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/ip ${CMAKE_CURRENT_SOURCE_DIR}/osc)


	if(MSVC)
	  # Force to always compile with W4
	  if(CMAKE_CXX_FLAGS MATCHES "/W[0-4]")
	    string(REGEX REPLACE "/W[0-4]" "/W4" CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
	  else()
	    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W4")
	  endif()
	elseif(CMAKE_COMPILER_IS_GNUCC OR CMAKE_COMPILER_IS_GNUCXX)
	  # Update if necessary
	  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -Wno-long-long -pedantic")
	endif()
ENDIF()
