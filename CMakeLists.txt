cmake_minimum_required(VERSION 2.8)
PROJECT(qtservice)

OPTION(${PROJECT_NAME}_BUILD_SHARED "Build ${PROJECT_NAME} as shared libraries" true)

SET(${PROJECT_NAME}-qt-components QtCore)

ADD_DEFINITIONS(-DUNICODE)

SET(QT_DONT_USE_QTGUI TRUE)
IF(NOT WIN32)
  SET(QT_USE_QTNETWORK TRUE)
  SET(${PROJECT_NAME}-qt-components ${${PROJECT_NAME}-qt-components} QtNetwork)
ENDIF()

IF(NOT ${${PROJECT_NAME}_BUILD_SHARED})
  SET(${PROJECT_NAME}-libtype "STATIC")
ELSE()
  SET(${PROJECT_NAME}-libtype "SHARED")
  ADD_DEFINITIONS(-DQT_QTSERVICE_EXPORT)
ENDIF()
find_package(Qt4 COMPONENTS ${${PROJECT_NAME}-qt-components})
include(${QT_USE_FILE})

# generate moc files
QT4_GENERATE_MOC(qtservice.cpp qtservice.moc)
IF (WIN32)
  QT4_GENERATE_MOC(qtservice_win.cpp qtservice_win.moc)
ELSEIF(UNIX)
  QT4_GENERATE_MOC(qtservice_unix.cpp qtservice_unix.moc)
ENDIF()

SET(${PROJECT_NAME}-sources qtservice.cpp qtservice.moc)

IF (WIN32)
  SET(${PROJECT_NAME}-sources ${${PROJECT_NAME}-sources} qtservice_win.cpp qtservice_win.moc)
ELSEIF(UNIX)
  SET(${PROJECT_NAME}-sources ${${PROJECT_NAME}-sources} qtservice_unix.cpp qtunixsocket.cpp qtunixserversocket.cpp qtservice_unix.moc)
ENDIF()

# for #include of generated moc files
INCLUDE_DIRECTORIES(${CMAKE_CURRENT_BINARY_DIR})
    
# for #include of moc headers from binary tree
INCLUDE_DIRECTORIES(${CMAKE_CURRENT_SOURCE_DIR})

ADD_LIBRARY(${PROJECT_NAME} ${${PROJECT_NAME}-libtype} ${${PROJECT_NAME}-sources} ${${PROJECT_NAME}-mocs})
TARGET_LINK_LIBRARIES(${PROJECT_NAME} ${QT_LIBRARIES})

INSTALL(TARGETS ${PROJECT_NAME}
		EXPORT ${PROJECT_NAME}-libraries
		RUNTIME DESTINATION bin
		LIBRARY DESTINATION lib
		ARCHIVE DESTINATION lib
		)

INSTALL(FILES qtservice.h QtServiceBase QtServiceController DESTINATION ${QT_INCLUDE_DIR}/${PROJECT_NAME})
IF(UNIX)
  INSTALL(FILES qtunixserversocket.h qtunixsocket.h DESTINATION ${QT_INCLUDE_DIR}/${PROJECT_NAME})
ENDIF()

CONFIGURE_FILE( ${CMAKE_CURRENT_SOURCE_DIR}/ProjectConfig.cmake.in
	${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}-config.cmake @ONLY )
INSTALL(FILES ${PROJECT_BINARY_DIR}/${PROJECT_NAME}-config.cmake
	DESTINATION share/cmake/${PROJECT_NAME} COMPONENT Development )

INSTALL(EXPORT ${PROJECT_NAME}-libraries DESTINATION share/cmake/${PROJECT_NAME} COMPONENT Development)