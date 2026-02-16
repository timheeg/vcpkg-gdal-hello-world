# vcpkg-gdal-hello-world

This project verifies the `vcpkg` port build changes required to build `GDAL`
**with** `Java` bindings as part of the `gdal` port.

The changes to `gdal` are made as a custom overlay in this project.

## Linux

```bash
cmake --preset x64-linux-gcc-shared-release
cmake --build --preset x64-linux-gcc-shared-release --target install
cpack --preset x64-linux-gcc-shared-release
```

However, the cpack output is missing the gdal jni which is in installed in the
`/vcpkg/package` folder.
