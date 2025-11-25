cmake_minimum_required(VERSION 3.4.1)

file(GLOB_RECURSE cppPaths "../ios/Stockfish/src/*.cpp")
add_library(
  stockfish
  SHARED
  ../ios/FlutterStockfish/ffi.cpp
  ${cppPaths}
)

# --- 16 KB PAGE SIZE FIX FOR ANDROID ARM64 ---
if(ANDROID_ABI STREQUAL "arm64-v8a")
  message(STATUS "Applying 16KB page size patch for arm64 build")
  set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -Wl,-z,max-page-size=16384")
  set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -Wl,-z,max-page-size=16384")
endif()
# ----------------------------------------------

if(ANDROID_ABI STREQUAL arm64-v8a)
  if (CMAKE_BUILD_TYPE STREQUAL "Debug")
    target_compile_options(stockfish PRIVATE -DUSE_PTHREADS -DIS_64BIT -DUSE_POPCNT -DUSE_NEON=8)
  else()
    target_compile_options(stockfish PRIVATE -fno-exceptions -DUSE_PTHREADS -DNDEBUG -O3 -DIS_64BIT -DUSE_POPCNT -DUSE_NEON=8)
  endif()
else()
  if (CMAKE_BUILD_TYPE STREQUAL "Debug")
    target_compile_options(stockfish PRIVATE -DUSE_PTHREADS)
  else()
    target_compile_options(stockfish PRIVATE -fno-exceptions -DUSE_PTHREADS -DNDEBUG -O3)
  endif()
endif()

file(DOWNLOAD https://tests.stockfishchess.org/api/nn/nn-1111cefa1111.nnue ${CMAKE_BINARY_DIR}/nn-1111cefa1111.nnue)
file(DOWNLOAD https://tests.stockfishchess.org/api/nn/nn-37f18f62d772.nnue ${CMAKE_BINARY_DIR}/nn-37f18f62d772.nnue)

