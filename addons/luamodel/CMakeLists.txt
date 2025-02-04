CMAKE_MINIMUM_REQUIRED(VERSION 3.0)

LIST( APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/CMake )

SET_TARGET_PROPERTIES ( ${PROJECT_EXECUTABLES} PROPERTIES
  LINKER_LANGUAGE CXX
  )

FIND_PACKAGE (Lua 5.1 REQUIRED)

INCLUDE_DIRECTORIES (
  ${CMAKE_SOURCE_DIR}/include/
  ${CMAKE_BINARY_DIR}/include/
  ${LUA_INCLUDE_DIR}
  )

SET ( LUAMODEL_SOURCES 
  luamodel.cc
  luatables.cc
  )

IF(RBDL_BUILD_EXECUTABLES)
  ADD_EXECUTABLE (rbdl_luamodel_util rbdl_luamodel_util.cc)
  IF (RBDL_USE_CASADI_MATH)
    ADD_EXECUTABLE (test_casadi_luareader test_casadi_luareader.cc)
  ENDIF(RBDL_USE_CASADI_MATH)
  ADD_EXECUTABLE (test_eigen_luareader test_eigen_luareader.cc)
ENDIF(RBDL_BUILD_EXECUTABLES)


IF (RBDL_BUILD_STATIC)
  ADD_LIBRARY ( rbdl_luamodel-static STATIC ${LUAMODEL_SOURCES} )
  IF (NOT WIN32)
    SET_TARGET_PROPERTIES ( rbdl_luamodel-static PROPERTIES PREFIX "lib")
  ENDIF (NOT WIN32)
  SET_TARGET_PROPERTIES ( rbdl_luamodel-static PROPERTIES OUTPUT_NAME "rbdl_luamodel")

  
  IF(RBDL_BUILD_ADDON_MUSCLE)
    TARGET_LINK_LIBRARIES (rbdl_luamodel-static
      rbdl_muscle-static         
      rbdl-static
      ${LUA_LIBRARIES} 
    )
    TARGET_LINK_LIBRARIES (rbdl_luamodel_util
      rbdl_luamodel-static      
      rbdl_muscle-static               
      rbdl-static
      ${LUA_LIBRARIES} 
    )

  ELSE(RBDL_BUILD_ADDON_MUSCLE)
    TARGET_LINK_LIBRARIES (rbdl_luamodel-static
      rbdl-static
      ${LUA_LIBRARIES}
    )
    IF(RBDL_BUILD_EXECUTABLES)
    TARGET_LINK_LIBRARIES (rbdl_luamodel_util
      rbdl_luamodel-static      
      rbdl-static
      ${LUA_LIBRARIES}
	    ${LUA_LIBRARY}
	    )

    ENDIF(RBDL_BUILD_EXECUTABLES)
  ENDIF(RBDL_BUILD_ADDON_MUSCLE)

  IF(RBDL_BUILD_EXECUTABLES)
  INSTALL (TARGETS rbdl_luamodel_util
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  )
  ENDIF(RBDL_BUILD_EXECUTABLES)

  INSTALL (TARGETS rbdl_luamodel-static
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  )

ELSE (RBDL_BUILD_STATIC)
  ADD_LIBRARY ( rbdl_luamodel SHARED ${LUAMODEL_SOURCES} )
  SET_TARGET_PROPERTIES ( rbdl_luamodel PROPERTIES
    VERSION ${RBDL_VERSION}
    SOVERSION ${RBDL_SO_VERSION}
    )
  IF (RBDL_USE_CASADI_MATH) # Build seperate library for casadi
    ADD_LIBRARY ( rbdl-casadi_luamodel SHARED ${LUAMODEL_SOURCES} )
    SET_TARGET_PROPERTIES ( rbdl-casadi_luamodel PROPERTIES
      VERSION ${RBDL_VERSION}
      SOVERSION ${RBDL_SO_VERSION}
    )
    TARGET_COMPILE_DEFINITIONS(rbdl-casadi_luamodel PUBLIC RBDL_USE_CASADI_MATH)
    
  
  ENDIF (RBDL_USE_CASADI_MATH)

  IF(RBDL_BUILD_ADDON_MUSCLE)
    TARGET_LINK_LIBRARIES (rbdl_luamodel
      rbdl_muscle          
      rbdl
      ${LUA_LIBRARIES}
    )
    IF(RBDL_BUILD_EXECUTABLES)
    TARGET_LINK_LIBRARIES (rbdl_luamodel_util
      rbdl_luamodel
      rbdl_muscle            
      rbdl
      )
    TARGET_LINK_LIBRARIES (rbdl_luamodel_util
      rbdl_luamodel
      rbdl_muscle            
      rbdl
    )
    ENDIF(RBDL_BUILD_EXECUTABLES)
  ELSE(RBDL_BUILD_ADDON_MUSCLE)
    TARGET_LINK_LIBRARIES (rbdl_luamodel
      rbdl
      ${LUA_LIBRARIES}
      )
    IF (RBDL_USE_CASADI_MATH)
    TARGET_LINK_LIBRARIES (rbdl-casadi_luamodel
        rbdl-casadi
        ${LUA_LIBRARIES}
    )
    ENDIF (RBDL_USE_CASADI_MATH)

    IF(RBDL_BUILD_EXECUTABLES)
    TARGET_LINK_LIBRARIES (rbdl_luamodel_util
      rbdl_luamodel
      rbdl
      )
    IF (RBDL_USE_CASADI_MATH)
    TARGET_LINK_LIBRARIES (test_casadi_luareader
      rbdl-casadi_luamodel
      rbdl-casadi
      )
    ENDIF (RBDL_USE_CASADI_MATH)
    TARGET_LINK_LIBRARIES (test_eigen_luareader
      rbdl_luamodel
      rbdl
      )
    ENDIF(RBDL_BUILD_EXECUTABLES)
  ENDIF(RBDL_BUILD_ADDON_MUSCLE)

  IF(RBDL_BUILD_EXECUTABLES)
  INSTALL (TARGETS rbdl_luamodel_util
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  )
  ENDIF(RBDL_BUILD_EXECUTABLES)

  INSTALL (TARGETS rbdl_luamodel
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    )

  IF (RBDL_USE_CASADI_MATH) # Install casadi luamodel
    INSTALL(TARGETS rbdl-casadi_luamodel
     RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
     LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
     ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
     )
  ENDIF (RBDL_USE_CASADI_MATH)
ENDIF (RBDL_BUILD_STATIC)

FILE ( GLOB headers 
  "${CMAKE_CURRENT_SOURCE_DIR}/*.h"
  )

INSTALL ( FILES ${headers} 
  DESTINATION 
  ${CMAKE_INSTALL_INCLUDEDIR}/rbdl/addons/luamodel
  )
