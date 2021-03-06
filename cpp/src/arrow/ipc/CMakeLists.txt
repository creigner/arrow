# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

#
# Messaging and interprocess communication

add_arrow_test(feather_test)
add_arrow_test(read_write_test PREFIX "arrow-ipc")
add_arrow_test(json_simple_test PREFIX "arrow-ipc")
add_arrow_test(json_test PREFIX "arrow-ipc")

# json_integration_test is two things at the same time:
# - an executable that can be called to answer integration test requests
# - a self-(unit)test for the C++ side of integration testing
if(ARROW_BUILD_TESTS)
  add_arrow_test(json_integration_test EXTRA_LINK_LIBS ${GFLAGS_LIBRARIES})
  add_dependencies(arrow-integration arrow-json-integration-test)
elseif(ARROW_BUILD_INTEGRATION)
  add_executable(arrow-json-integration-test json_integration_test.cc)
  target_link_libraries(arrow-json-integration-test ${ARROW_TEST_LINK_LIBS}
                        ${GFLAGS_LIBRARIES} GTest::GTest)

  add_dependencies(arrow-json-integration-test arrow arrow_testing)
  add_dependencies(arrow-integration arrow-json-integration-test)
endif()

# Headers: top level
arrow_install_all_headers("arrow/ipc")

if(ARROW_BUILD_STATIC)
  set(ARROW_UTIL_LIB arrow_static)
else()
  set(ARROW_UTIL_LIB arrow_shared)
endif()

if(ARROW_BUILD_UTILITIES OR ARROW_BUILD_INTEGRATION)
  add_executable(arrow-file-to-stream file_to_stream.cc)
  target_link_libraries(arrow-file-to-stream ${ARROW_UTIL_LIB})
  add_executable(arrow-stream-to-file stream_to_file.cc)
  target_link_libraries(arrow-stream-to-file ${ARROW_UTIL_LIB})

  if(ARROW_BUILD_INTEGRATION)
    add_dependencies(arrow-integration arrow-file-to-stream)
    add_dependencies(arrow-integration arrow-stream-to-file)
  endif()
endif()

add_arrow_benchmark(read_write_benchmark PREFIX "arrow-ipc")

if(ARROW_FUZZING)
  add_executable(arrow-ipc-generate-fuzz-corpus generate_fuzz_corpus.cc)
  target_link_libraries(arrow-ipc-generate-fuzz-corpus ${ARROW_UTIL_LIB}
                        ${ARROW_TEST_LINK_LIBS})
endif()

add_arrow_fuzz_target(file_fuzz PREFIX "arrow-ipc")
add_arrow_fuzz_target(stream_fuzz PREFIX "arrow-ipc")
