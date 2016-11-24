#*******************************************************************************
#  Copyright (c) 2015 logi.cals GmbH
# 
#  All rights reserved. This program and the accompanying materials
#  are made available under the terms of the Eclipse Public License v1.0
#  and Eclipse Distribution License v1.0 which accompany this distribution. 
# 
#  The Eclipse Public License is available at 
#     http://www.eclipse.org/legal/epl-v10.html
#  and the Eclipse Distribution License is available at 
#    http://www.eclipse.org/org/documents/edl-v10.php.
# 
#  Contributors:
#     Rainer Poisel - initial version
#*******************************************************************************/

# Note: on OS X you should install XCode and the associated command-line tools

## compilation/linkage settings
INCLUDE_DIRECTORIES(
    .
    ${CMAKE_BINARY_DIR}
    )

CONFIGURE_FILE(VersionInfo.h.in
    ${CMAKE_BINARY_DIR}/VersionInfo.h
    @ONLY
    )

SET(common_src
    MQTTProtocolClient.c
    Clients.c
    utf-8.c
    StackTrace.c
    MQTTPacket.c
    MQTTPacketOut.c
    Messages.c
    Tree.c
    Socket.c
    Log.c
    MQTTPersistence.c
    Thread.c
    MQTTProtocolOut.c
    MQTTPersistenceDefault.c
    SocketBuffer.c
    Heap.c
    LinkedList.c
    )

ADD_LIBRARY(paho-mqtt3a STATIC ${common_src} MQTTAsync.c)

IF (CMAKE_SYSTEM_NAME MATCHES "Windows")
    SET(LIBS_SYSTEM ws2_32)
	TARGET_LINK_LIBRARIES(paho-mqtt3a ${LIBS_SYSTEM})
	ADD_DEFINITIONS(/D _CRT_SECURE_NO_WARNINGS)
ELSEIF (CMAKE_SYSTEM_NAME MATCHES "Linux")
    SET(LIBS_SYSTEM dl)
    IF (CMAKE_SYSTEM_NAME MATCHES "ANDROID")
	   TARGET_LINK_LIBRARIES(paho-mqtt3a ${LIBS_SYSTEM})
    ELSE()        
        TARGET_LINK_LIBRARIES(paho-mqtt3a pthread ${LIBS_SYSTEM})
    ENDIF()
ENDIF()

IF (PAHO_WITH_SSL)
    SET(OPENSSL_LIB_SEARCH_PATH "" CACHE PATH "Directory containing OpenSSL libraries")
    SET(OPENSSL_INC_SEARCH_PATH "" CACHE PATH "Directory containing OpenSSL includes")
    SET(OPENSSL_LIBRARIES ssl crypto)
    LINK_DIRECTORIES(
        ${OPENSSL_LIB_SEARCH_PATH}
        )
    INCLUDE_DIRECTORIES(
        ${OPENSSL_INC_SEARCH_PATH}
        )
    ADD_LIBRARY(paho-mqtt3cs SHARED ${common_src} MQTTClient.c SSLSocket.c)
    ADD_LIBRARY(paho-mqtt3as STATIC ${common_src} MQTTAsync.c SSLSocket.c)
    ADD_DEFINITIONS(-DOPENSSL=1)
    TARGET_LINK_LIBRARIES(paho-mqtt3cs pthread ${OPENSSL_LIBRARIES} ${LIBS_SYSTEM})
    TARGET_LINK_LIBRARIES(paho-mqtt3as pthread ${OPENSSL_LIBRARIES} ${LIBS_SYSTEM})
#    SET_TARGET_PROPERTIES(
#        paho-mqtt3cs paho-mqtt3as PROPERTIES
#        VERSION ${CLIENT_VERSION}
#        SOVERSION ${PAHO_VERSION_MAJOR})
#    INSTALL(TARGETS paho-mqtt3cs
#        RUNTIME DESTINATION bin
#        LIBRARY DESTINATION lib)
#    INSTALL(TARGETS paho-mqtt3as
#        RUNTIME DESTINATION bin
#        LIBRARY DESTINATION lib)
ENDIF()

