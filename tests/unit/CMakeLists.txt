project(mococrw)

if(BUILD_TESTING) #set by Ctest. Also set in the integration build environment.
    if(NOT GMOCK_BOTH_LIBRARIES)
        find_package(GMock REQUIRED CONFIG)
        include_directories(SYSTEM ${GMOCK_INCLUDE_DIRS})
    endif()

    #TODO: clean this up
    set(LIB_SOURCES "${SRC_DIR}/openssl_wrap.cpp" "${SRC_DIR}/bio.cpp" "${SRC_DIR}/distinguished_name.cpp")
    set(MOCK_SOURCES "openssl_lib_mock.cpp" ${LIB_SOURCES})
    set(REAL_SOURCES "${SRC_DIR}/openssl_lib.cpp" ${LIB_SOURCES})

    find_package(Threads)

    add_executable(openssltest test_opensslwrapper.cpp ${MOCK_SOURCES})
    add_executable(keytests test_key.cpp ${REAL_SOURCES})
    add_executable(csrtests test_csr.cpp "${SRC_DIR}/key.cpp" ${REAL_SOURCES})
    add_executable(biotests test_bio.cpp ${REAL_SOURCES})
    add_executable(x509tests test_x509.cpp "${SRC_DIR}/key.cpp" ${REAL_SOURCES})
    add_executable(dntests test_distinguished_name.cpp ${REAL_SOURCES})

    target_link_libraries(openssltest ${GMOCK_BOTH_LIBRARIES} ${CMAKE_THREAD_LIBS_INIT})
    target_link_libraries(keytests ${GMOCK_BOTH_LIBRARIES} ${CMAKE_THREAD_LIBS_INIT} ${OPENSSL_LIBRARIES})
    target_link_libraries(csrtests ${GMOCK_BOTH_LIBRARIES} ${CMAKE_THREAD_LIBS_INIT} ${OPENSSL_LIBRARIES})
    target_link_libraries(biotests ${GMOCK_BOTH_LIBRARIES} ${CMAKE_THREAD_LIBS_INIT} ${OPENSSL_LIBRARIES})
    target_link_libraries(x509tests ${GMOCK_BOTH_LIBRARIES} ${CMAKE_THREAD_LIBS_INIT} ${OPENSSL_LIBRARIES})
    target_link_libraries(dntests ${GMOCK_BOTH_LIBRARIES} ${CMAKE_THREAD_LIBS_INIT} ${OPENSSL_LIBRARIES})

    add_test(OpenSSLTest openssltest)
    add_test(KeyTest keytests)
    add_test(CsrTests csrtests)
    add_test(
        NAME BioTests
        COMMAND biotests
        WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/test-certs"
    )
    add_test(DistinguishedNameTests dntests)
    add_test(
        NAME X509Tests
        COMMAND x509tests
        WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/test-certs"
    )
    message("The directory would be ${CMAKE_CURRENT_SOURCE_DIR}/test-certs")

    # Configure valgrind
    find_program(MEMORYCHECK_COMMAND NAMES valgrind)
    set(_ARGS "--leak-check=full --track-origins=yes")
    set(_ARGS "${_ARGS} --xml=yes --xml-file=valgrind_%p.xml")
    set(MEMORYCHECK_COMMAND_OPTIONS ${_ARGS})
    include(CTest)
endif()