cmake_minimum_required(VERSION 3.5)

# Project
project(tensorflow-lite C CXX)


# From https://stackoverflow.com/questions/10113017/setting-the-msvc-runtime-in-cmake
macro(configure_msvc_runtime)
  if(MSVC)
    # Default to statically-linked runtime.
    if("${MSVC_RUNTIME}" STREQUAL "")
      set(MSVC_RUNTIME "static")
    endif()
    # Set compiler options.
    set(variables
      CMAKE_C_FLAGS_DEBUG
      CMAKE_C_FLAGS_MINSIZEREL
      CMAKE_C_FLAGS_RELEASE
      CMAKE_C_FLAGS_RELWITHDEBINFO
      CMAKE_CXX_FLAGS_DEBUG
      CMAKE_CXX_FLAGS_MINSIZEREL
      CMAKE_CXX_FLAGS_RELEASE
      CMAKE_CXX_FLAGS_RELWITHDEBINFO
    )
    if(${MSVC_RUNTIME} STREQUAL "static")
      message(STATUS
        "MSVC -> forcing use of statically-linked runtime."
      )
      foreach(variable ${variables})
        if(${variable} MATCHES "/MD")
          string(REGEX REPLACE "/MD" "/MT" ${variable} "${${variable}}")
        endif()
      endforeach()
    else()
      message(STATUS
        "MSVC -> forcing use of dynamically-linked runtime."
      )
      foreach(variable ${variables})
        if(${variable} MATCHES "/MT")
          string(REGEX REPLACE "/MT" "/MD" ${variable} "${${variable}}")
        endif()
      endforeach()
    endif()
  endif()
endmacro()


SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}" )

# set(GTEST_LIB_DIR "${SDK_DIR}/googletest/lib")

set(TENSORFLOW_ROOT_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../../..")
set(TENSORFLOW_LITE_ROOT_DIR "${TENSORFLOW_ROOT_DIR}/tensorflow/contrib/lite")
set(TENSORFLOW_LITE_KERNEL_DIR "${TENSORFLOW_LITE_ROOT_DIR}/kernels")
set(TENSORFLOW_LITE_DOWNLOADED_DEPENDENCIES_DIR "${TENSORFLOW_LITE_ROOT_DIR}/tools/make/downloads")
set(TENSORFLOW_LITE_GEMMLOWP_DIR "${TENSORFLOW_LITE_DOWNLOADED_DEPENDENCIES_DIR}/gemmlowp")
set(TENSORFLOW_LITE_EIGEN_DIR "${TENSORFLOW_LITE_DOWNLOADED_DEPENDENCIES_DIR}/eigen")
set(TENSORFLOW_LITE_FLATBUFFERS_DIR "${TENSORFLOW_LITE_DOWNLOADED_DEPENDENCIES_DIR}/flatbuffers/include")
set(TENSORFLOW_LITE_FARMHASH_DIR "${TENSORFLOW_LITE_DOWNLOADED_DEPENDENCIES_DIR}/farmhash/src")
set(TENSORFLOW_LITE_GMOCK_DIR "${TENSORFLOW_LITE_DOWNLOADED_DEPENDENCIES_DIR}/googletest/googlemock/include")
set(TENSORFLOW_LITE_GTEST_DIR "${TENSORFLOW_LITE_DOWNLOADED_DEPENDENCIES_DIR}/googletest/googletest/include")
set(TENSORFLOW_LITE_NEON2SSE_DIR "${TENSORFLOW_LITE_DOWNLOADED_DEPENDENCIES_DIR}/neon_2_sse")
set(TENSORFLOW_LITE_FFT2D_DIR "${TENSORFLOW_LITE_DOWNLOADED_DEPENDENCIES_DIR}/fft2d")

if (WIN32)
    # Set C++14 as standard for the whole project
    set(CMAKE_CXX_STANDARD 14)
    set(PLATFORM_ENV_TIME_SRC ${TENSORFLOW_ROOT_DIR}/tensorflow/core/platform/windows/env_time.cc)
    configure_msvc_runtime()
    set(variables
      CMAKE_C_FLAGS_DEBUG
      CMAKE_C_FLAGS_MINSIZEREL
      CMAKE_C_FLAGS_RELEASE
      CMAKE_C_FLAGS_RELWITHDEBINFO
      CMAKE_CXX_FLAGS_DEBUG
      CMAKE_CXX_FLAGS_MINSIZEREL
      CMAKE_CXX_FLAGS_RELEASE
      CMAKE_CXX_FLAGS_RELWITHDEBINFO
    )

    # TODO: SIMD Visual studio
    add_definitions(
        -DGEMMLOWP_ALLOW_SLOW_SCALAR_FALLBACK
        -DNOMINMAX
        -DFARMHASH_NO_BUILTIN_EXPECT)
else ()
    set(CMAKE_CXX_STANDARD 11)
endif (WIN32)

if (ANDROID)
    SET (NNAPI_SOURCE
        ${TENSORFLOW_LITE_ROOT_DIR}/nnapi_delegate.cc
        )
else()
    SET (NNAPI_SOURCE
        ${TENSORFLOW_LITE_ROOT_DIR}/nnapi_delegate_disabled.cc
        )
endif (ANDROID)

if (WIN32)
    SET (MMAP_SOURCE
        ${TENSORFLOW_LITE_ROOT_DIR}/mmap_allocation_disabled.cc
        )
else()
    SET (MMAP_SOURCE
        ${TENSORFLOW_LITE_ROOT_DIR}/mmap_allocation.cc
        )
endif (WIN32)

add_definitions(
  -DFARMHASH_NO_CXX_STRING
  -DNDEBUG)

if(ANDROID_ABI STREQUAL x86)
  add_definitions(
    -DGEMMLOWP_ALLOW_SLOW_SCALAR_FALLBACK)
endif()

include_directories(${TENSORFLOW_ROOT_DIR}
    ${TENSORFLOW_LITE_GEMMLOWP_DIR}
    ${TENSORFLOW_LITE_FLATBUFFERS_DIR}
    ${TENSORFLOW_LITE_EIGEN_DIR}
    ${TENSORFLOW_LITE_FARMHASH_DIR}
    ${TENSORFLOW_LITE_GMOCK_DIR}
    ${TENSORFLOW_LITE_GTEST_DIR}
    ${TENSORFLOW_LITE_NEON2SSE_DIR}
    )

SET (TENSORFLOW_LITE_KERNEL_SOURCES
    ${TENSORFLOW_LITE_KERNEL_DIR}/activations.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/add.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/arg_min_max.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/audio_spectrogram.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/basic_rnn.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/batch_to_space_nd.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/bidirectional_sequence_lstm.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/bidirectional_sequence_rnn.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/cast.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/comparisons.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/concatenation.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/conv.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/depthwise_conv.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/dequantize.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/detection_postprocess.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/div.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/elementwise.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/eigen_support.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/embedding_lookup.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/embedding_lookup_sparse.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/exp.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/expand_dims.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/fake_quant.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/floor.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/floor_div.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/fully_connected.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/gather.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/gemm_support.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/hashtable_lookup.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/kernel_util.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/l2norm.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/local_response_norm.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/logical.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/lsh_projection.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/lstm.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/maximum_minimum.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/mfcc.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/mul.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/neg.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/one_hot.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/pack.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/pad.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/pooling.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/pow.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/reduce.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/register.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/reshape.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/resize_bilinear.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/select.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/shape.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/skip_gram.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/slice.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/space_to_batch_nd.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/space_to_depth.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/sparse_to_dense.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/split.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/squeeze.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/strided_slice.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/sub.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/tile.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/svdf.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/topk_v2.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/transpose.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/transpose_conv.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/unidirectional_sequence_lstm.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/unidirectional_sequence_rnn.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/unpack.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/internal/kernel_utils.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/internal/mfcc.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/internal/mfcc_dct.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/internal/mfcc_mel_filterbank.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/internal/optimized/neon_tensor_utils.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/internal/quantization_util.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/internal/reference/portable_tensor_utils.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/internal/spectrogram.cc
    ${TENSORFLOW_LITE_KERNEL_DIR}/internal/tensor_utils.cc
    )

add_library(tensorflow-lite STATIC
    ${TENSORFLOW_LITE_ROOT_DIR}/allocation.cc
    ${TENSORFLOW_LITE_ROOT_DIR}/arena_planner.cc
    ${TENSORFLOW_LITE_ROOT_DIR}/context.c
    ${TENSORFLOW_LITE_ROOT_DIR}/error_reporter.cc
    ${TENSORFLOW_LITE_ROOT_DIR}/graph_info.cc
    ${TENSORFLOW_LITE_ROOT_DIR}/interpreter.cc
    ${MMAP_SOURCE}
    ${TENSORFLOW_LITE_ROOT_DIR}/model.cc
    ${NNAPI_SOURCE}
    ${TENSORFLOW_LITE_ROOT_DIR}/op_resolver.cc
    ${TENSORFLOW_LITE_ROOT_DIR}/optional_debug_tools.cc
    ${TENSORFLOW_LITE_ROOT_DIR}/simple_memory_arena.cc
    ${TENSORFLOW_LITE_ROOT_DIR}/string_util.cc
    ${TENSORFLOW_LITE_ROOT_DIR}/util.cc
    ${TENSORFLOW_LITE_KERNEL_SOURCES}
    ${TENSORFLOW_LITE_FARMHASH_DIR}/farmhash.cc
    ${TENSORFLOW_LITE_FFT2D_DIR}/fftsg.c
    )



