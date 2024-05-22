include_directories(${CMAKE_SOURCE_DIR})

# it is located in fast_rnnt/cmake/transform.cmake
include(transform)

set(srcs
  mutual_information_cpu.cu
)

if (FT_WITH_CUDA)
  list(APPEND srcs mutual_information_cuda.cu)
elseif(FT_WITH_HIP)
  list(APPEND srcs mutual_information_hip.hip)
else()
  transform(OUTPUT_VARIABLE srcs SRCS ${srcs})
endif()

add_library(mutual_information_core ${srcs})
target_link_libraries(mutual_information_core PUBLIC ${TORCH_LIBRARIES})
# for <torch/extension.h>
target_include_directories(mutual_information_core PUBLIC ${PYTHON_INCLUDE_DIRS})

if (FT_WITH_HIP)
  set_source_files_properties(${srcs} PROPERTIES LANGUAGE HIP)
  set_target_properties(mutual_information_core PROPERTIES LINKER_LANGUAGE HIP)
  target_link_libraries(mutual_information_core PUBLIC hip::device hip::host)
endif()
