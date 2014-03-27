cmake_minimum_required(VERSION 2.8)
PROJECT(qtservice)

OPTION(${PROJECT_NAME}_BUILD_SHARED "Build ${PROJECT_NAME} as shared libraries" true)
OPTION(USE_QT_5 "Use Qt5" NO)

SET(${PROJECT_NAME}-qt-components QtCore)

ADD_DEFINITIONS(-DUNICODE)

IF(USE_QT_5)
	FIND_PACKAGE(Qt5Core)

	IF(NOT WIN32)
		FIND_PACKAGE(Qt5Network)
	ENDIF()
ELSE()
	SET(QT_DONT_USE_QTGUI TRUE)
	IF(NOT WIN32)
	  SET(QT_USE_QTNETWORK TRUE)
	  SET(${PROJECT_NAME}-qt-components ${${PROJECT_NAME}-qt-components} QtNetwork)
	ENDIF()

	include(${QT_USE_FILE})

	find_package(Qt4 COMPONENTS ${${PROJECT_NAME}-qt-components})

ENDIF()

IF(MSVC AND BUILD_SHARED_LIBS)
  ADD_DEFINITIONS("/DQT_QTSERVICE_EXPORT=__declspec(dllexport)")
ENDIF()

# generate moc files
set(CMAKE_AUTOMOC TRUE)

SET(${PROJECT_NAME}-sources qtservice.cpp)

IF (WIN32)
  SET(${PROJECT_NAME}-sources ${${PROJECT_NAME}-sources} qtservice_win.cpp)
ELSEIF(UNIX)
  SET(${PROJECT_NAME}-sources ${${PROJECT_NAME}-sources} qtservice_unix.cpp qtunixsocket.cpp qtunixserversocket.cpp)
ENDIF()

# for #include of generated moc files
INCLUDE_DIRECTORIES(${CMAKE_CURRENT_BINARY_DIR})
    
# for #include of moc headers from binary tree
INCLUDE_DIRECTORIES(${CMAKE_CURRENT_SOURCE_DIR})

ADD_LIBRARY(${PROJECT_NAME} ${${PROJECT_NAME}-sources} ${${PROJECT_NAME}-mocs})

IF(NOT USE_QT_5)
	TARGET_LINK_LIBRARIES(${PROJECT_NAME} ${QT_LIBRARIES})
ELSE()
	qt5_use_modules(${PROJECT_NAME} Core Network)
ENDIF()

INSTALL(TARGETS ${PROJECT_NAME}
    EXPORT flow-LibraryDependencies
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

INSTALL(EXPORT flow-LibraryDependencies DESTINATION share/cmake/${PROJECT_NAME} COMPONENT Development)

set(QTSERVICE_INCLUDE_DIRS
  ${CMAKE_CURRENT_SOURCE_DIR}
  CACHE INTERNAL "${PROJECT_NAME} include directories"
)

set(QTSERVICE_LIBRARIES
  ${PROJECT_NAME}
  CACHE INTERNAL "${PROJECT_NAME} libraries"
)
