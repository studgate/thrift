#
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements. See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership. The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License. You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied. See the License for the
# specific language governing permissions and limitations
# under the License.
#

set(Boost_USE_STATIC_LIBS ON)
find_package(Boost 1.53.0 REQUIRED COMPONENTS program_options system filesystem)
include_directories(SYSTEM "${Boost_INCLUDE_DIRS}")

#Make sure gen-cpp files can be included
include_directories("${CMAKE_CURRENT_BINARY_DIR}")
include_directories("${CMAKE_CURRENT_BINARY_DIR}/gen-cpp")
include_directories("${PROJECT_SOURCE_DIR}/lib/cpp/src")


set(crosstestgencpp_SOURCES
    gen-cpp/ThriftTest.cpp
    gen-cpp/ThriftTest_types.cpp
    gen-cpp/ThriftTest_constants.cpp
    src/ThriftTest_extras.cpp
)
add_library(crosstestgencpp STATIC ${crosstestgencpp_SOURCES})
target_link_libraries(crosstestgencpp thrift)

set(crossstressgencpp_SOURCES
    gen-cpp/Service.cpp
    gen-cpp/StressTest_types.cpp
    gen-cpp/StressTest_constants.cpp
)
add_library(crossstressgencpp STATIC ${crossstressgencpp_SOURCES})
target_link_libraries(crossstressgencpp thrift)

add_executable(TestServer src/TestServer.cpp)
target_link_libraries(TestServer thrift thriftnb crosstestgencpp ${Boost_LIBRARIES} ${LIBEVENT_LIB})

add_executable(TestClient src/TestClient.cpp)
target_link_libraries(TestClient thrift thriftnb crosstestgencpp ${Boost_LIBRARIES} ${LIBEVENT_LIB})

add_executable(StressTest src/StressTest.cpp)
target_link_libraries(StressTest thrift thriftnb crossstressgencpp ${Boost_LIBRARIES} ${LIBEVENT_LIB})
add_test(NAME StressTest COMMAND StressTest)

add_executable(StressTestNonBlocking src/StressTestNonBlocking.cpp)
target_link_libraries(StressTestNonBlocking thrift thriftz thriftnb crossstressgencpp ${Boost_LIBRARIES} ${LIBEVENT_LIB})
add_test(NAME StressTestNonBlocking COMMAND StressTestNonBlocking)

#
# Common thrift code generation rules
#

add_custom_command(OUTPUT gen-cpp/ThriftTest.cpp gen-cpp/ThriftTest_types.cpp gen-cpp/ThriftTest_constants.cpp
    COMMAND thrift-compiler --gen cpp:templates,cob_style -r ${PROJECT_SOURCE_DIR}/test/ThriftTest.thrift
)

add_custom_command(OUTPUT gen-cpp/StressTest_types.cpp gen-cpp/StressTest_constants.cpp gen-cpp/Service.cpp
    COMMAND thrift-compiler --gen cpp ${PROJECT_SOURCE_DIR}/test/StressTest.thrift
)
