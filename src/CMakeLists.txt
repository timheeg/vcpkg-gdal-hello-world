add_executable(hello-world)

target_sources(hello-world PRIVATE
  main.cpp
)

# Link library dependencies
target_link_libraries(hello-world
  PRIVATE
    compiler_flags
  PUBLIC
    GDAL::GDAL
)

install(TARGETS hello-world
  DESTINATION ${CMAKE_INSTALL_BINDIR}
)

# Install dependencies
install(CODE [[
  function(install_library_with_deps LIBRARY)
    file(INSTALL
      DESTINATION "${CMAKE_INSTALL_PREFIX}/lib"
      TYPE SHARED_LIBRARY
      FOLLOW_SYMLINK_CHAIN
      FILES "${LIBRARY}"
    )
    file(GET_RUNTIME_DEPENDENCIES
      LIBRARIES ${LIBRARY}
      RESOLVED_DEPENDENCIES_VAR RESOLVED_DEPS
      UNRESOLVED_DEPENDENCIES_VAR UNRESOLVED_DEPS
    )
    foreach(FILE ${RESOLVED_DEPS})
      if(NOT IS_SYMLINK ${FILE})
        install_library_with_deps(${FILE})
      endif()
    endforeach()
    foreach(FILE ${UNRESOLVED_DEPS})
      message(STATUS "Unresolved from ${LIBRARY}: ${FILE}")
    endforeach()
  endfunction()
  file(GET_RUNTIME_DEPENDENCIES
    EXECUTABLES  $<TARGET_FILE:hello-world>
    RESOLVED_DEPENDENCIES_VAR RESOLVED_DEPS
    UNRESOLVED_DEPENDENCIES_VAR UNRESOLVED_DEPS
  )
  foreach(FILE ${RESOLVED_DEPS})
    install_library_with_deps(${FILE})
  endforeach()
  foreach(FILE ${UNRESOLVED_DEPS})
    message(STATUS "Unresolved: ${FILE}")
  endforeach()
]])
