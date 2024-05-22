include_directories(${CMAKE_SOURCE_DIR})

include(transform)

# please keep the list sorted
set(fast_rnnt_srcs
  fast_rnnt.cu
  mutual_information.cu
)

# this is not CUDA code so we need the regular C++ compiler in the HIP case
# thus, also transform if FT_WITH_HIP
if(NOT FT_WITH_CUDA)
  transform(OUTPUT_VARIABLE fast_rnnt_srcs SRCS ${fast_rnnt_srcs})
endif()

pybind11_add_module(_fast_rnnt ${fast_rnnt_srcs})
target_link_libraries(_fast_rnnt PRIVATE mutual_information_core)

if(APPLE)
  target_link_libraries(_fast_rnnt
    PRIVATE
      ${TORCH_DIR}/lib/libtorch_python.dylib
  )
elseif(UNIX)
  target_link_libraries(_fast_rnnt
    PRIVATE
      ${PYTHON_LIBRARY}
      ${TORCH_DIR}/lib/libtorch_python.so
  )
endif()

if (FT_WITH_HIP)
  set_target_properties(_fast_rnnt PROPERTIES LINKER_LANGUAGE CXX)
endif()
