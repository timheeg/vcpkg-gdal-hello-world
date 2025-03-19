add_executable(hello-world)

target_sources(hello-world PRIVATE
  main.cpp
)

# Link library dependencies
target_link_libraries(hello-world PRIVATE
  compiler_flags
  GDAL::GDAL
)
