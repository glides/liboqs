if(NOT WIN32)
    set(LIBM m)

    if(USE_OPENSSL)
        include_directories(${OPENSSL_INCLUDE_DIR})
        set(INTERNAL_TEST_DEPS ${OPENSSL_CRYPTO_LIBRARY} ${LIBM})
    else()
        set(INTERNAL_TEST_DEPS ${LIBM})
    endif()

    execute_process(COMMAND ${PROJECT_SOURCE_DIR}/scripts/git_commit.sh OUTPUT_VARIABLE GIT_COMMIT)
    add_definitions(-DOQS_COMPILE_GIT_COMMIT="${GIT_COMMIT}")

    # These functions tested here are
    # not part of the liboqs API.
    get_directory_property(AES_IMPL
                           DIRECTORY ${PROJECT_SOURCE_DIR}/src/crypto
                           DEFINITION AES_IMPL)
    add_executable(test_aes test_aes.c ${AES_IMPL} ${PROJECT_SOURCE_DIR}/src/common/common.c)
    target_link_libraries(test_aes ${INTERNAL_TEST_DEPS})

    get_directory_property(SHA2_IMPL
                           DIRECTORY ${PROJECT_SOURCE_DIR}/src/crypto
                           DEFINITION SHA2_IMPL)
    add_executable(test_hash test_hash.c ${SHA2_IMPL} ${PROJECT_SOURCE_DIR}/src/common/common.c)
    target_link_libraries(test_hash ${INTERNAL_TEST_DEPS})

    get_directory_property(SHA3_IMPL
                           DIRECTORY ${PROJECT_SOURCE_DIR}/src/crypto
                           DEFINITION SHA3_IMPL)
    add_executable(test_sha3 test_sha3.c ${SHA3_IMPL} ${PROJECT_SOURCE_DIR}/src/common/common.c)
    target_link_libraries(test_sha3 ${INTERNAL_TEST_DEPS})

    set(UNIX_TESTS test_aes test_hash test_sha3)
    set(PYTHON3_EXEC python3)
    if(BUILD_SHARED_LIBS)
        set(USE_LIBOQS_SO --use_liboqs_so)
    endif()
else()
    set(PYTHON3_EXEC python)
endif()

set(API_TEST_DEPS oqs ${LIBM})

# KEM API tests
add_executable(example_kem example_kem.c)
target_link_libraries(example_kem ${API_TEST_DEPS})

add_executable(kat_kem kat_kem.c)
target_link_libraries(kat_kem ${API_TEST_DEPS})

add_executable(test_kem test_kem.c)
target_link_libraries(test_kem ${API_TEST_DEPS})

add_executable(speed_kem speed_kem.c)
target_link_libraries(speed_kem ${API_TEST_DEPS})

# SIG API tests
add_executable(example_sig example_sig.c)
target_link_libraries(example_sig ${API_TEST_DEPS})

add_executable(kat_sig kat_sig.c)
target_link_libraries(kat_sig ${API_TEST_DEPS})

add_executable(test_sig test_sig.c)
target_link_libraries(test_sig ${API_TEST_DEPS})

add_executable(speed_sig speed_sig.c)
target_link_libraries(speed_sig ${API_TEST_DEPS})

# TODO: Get CMake to find python.
# and set PATH variable in Windows
# for DLL builds.
add_custom_target(
    run_tests
    COMMAND ${PYTHON3_EXEC} -m pytest --verbose --numprocesses=auto --build_tool=${CMAKE_MAKE_PROGRAM} ${USE_LIBOQS_SO} --junitxml=${CMAKE_BINARY_DIR}/test-results/pytest/test-results.xml
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
    DEPENDS oqs example_kem kat_kem test_kem example_sig kat_sig test_sig ${UNIX_TESTS}
    USES_TERMINAL)
