# See adjacent README.rst for documentation of this test infrastructure.

# Note that the _isMultiConfig variable is set in the parent directory's
# CMakeLists.txt (slightly complex logic to support CMake versions before 3.9)

macro(add_RunCMake_test test)
  set(TEST_ARGS ${ARGN})
  if ("${ARGV1}" STREQUAL "TEST_DIR")
    if ("${ARGV2}" STREQUAL "")
      message(FATAL_ERROR "Invalid args")
    endif()
    set(Test_Dir ${ARGV2})
    list(REMOVE_AT TEST_ARGS 0)
    list(REMOVE_AT TEST_ARGS 0)
  else()
    set(Test_Dir ${test})
  endif()
  add_test(NAME RunCMake.${test} COMMAND ${CMAKE_CMAKE_COMMAND}
    -DCMAKE_MODULE_PATH=${CMAKE_CURRENT_SOURCE_DIR}
    -DRunCMake_GENERATOR_IS_MULTI_CONFIG=${_isMultiConfig}
    -DRunCMake_GENERATOR=${CMAKE_GENERATOR}
    -DRunCMake_GENERATOR_INSTANCE=${CMAKE_GENERATOR_INSTANCE}
    -DRunCMake_GENERATOR_PLATFORM=${CMAKE_GENERATOR_PLATFORM}
    -DRunCMake_GENERATOR_TOOLSET=${CMAKE_GENERATOR_TOOLSET}
    -DRunCMake_MAKE_PROGRAM=${CMake_TEST_EXPLICIT_MAKE_PROGRAM}
    -DRunCMake_SOURCE_DIR=${CMAKE_CURRENT_SOURCE_DIR}/${Test_Dir}
    -DRunCMake_BINARY_DIR=${CMAKE_CURRENT_BINARY_DIR}/${test}
    ${${test}_ARGS}
    ${TEST_ARGS}
    -P "${CMAKE_CURRENT_SOURCE_DIR}/${Test_Dir}/RunCMakeTest.cmake"
    )
  set_tests_properties("RunCMake.${test}" PROPERTIES LABELS "CMake;run")
  if(${test} MATCHES ^CMP)
    set_property(TEST "RunCMake.${test}" APPEND PROPERTY LABELS "policy")
  endif()
endmacro()

function(add_RunCMake_test_group test types)
  # create directory for common content
  set(TEST_CONFIG_DIR "${CMAKE_CURRENT_BINARY_DIR}/${test}/conf")
  file(REMOVE_RECURSE "${TEST_CONFIG_DIR}")
  file(MAKE_DIRECTORY "${TEST_CONFIG_DIR}")

  foreach(type IN LISTS types)
    # generate prerequirements config file in cmake as ctest doesn't have as
    # much system information so it is easier to set programs and environment
    # values here
    unset(${test}_${type}_FOUND_PREREQUIREMENTS)
    if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/${test}/${type}/Prerequirements.cmake")
      include("${CMAKE_CURRENT_SOURCE_DIR}/${test}/${type}/Prerequirements.cmake")
    else()
      string(REGEX MATCH "^[^.]*" main_type "${type}")
      include("${CMAKE_CURRENT_SOURCE_DIR}/${test}/${main_type}/Prerequirements.cmake")
    endif()
    get_test_prerequirements("${test}_${type}_FOUND_PREREQUIREMENTS"
        "${TEST_CONFIG_DIR}/${type}_config.cmake")

    # only add the test if prerequirements are met
    if(${test}_${type}_FOUND_PREREQUIREMENTS)
      add_test(NAME RunCMake.${test}_${type} COMMAND ${CMAKE_CMAKE_COMMAND}
        -DTEST_TYPE=${type}
        -DCMAKE_MODULE_PATH=${CMAKE_CURRENT_SOURCE_DIR}
        -DRunCMake_GENERATOR_IS_MULTI_CONFIG=${_isMultiConfig}
        -DRunCMake_GENERATOR=${CMAKE_GENERATOR}
        -DRunCMake_GENERATOR_INSTANCE=${CMAKE_GENERATOR_INSTANCE}
        -DRunCMake_GENERATOR_PLATFORM=${CMAKE_GENERATOR_PLATFORM}
        -DRunCMake_GENERATOR_TOOLSET=${CMAKE_GENERATOR_TOOLSET}
        -DRunCMake_MAKE_PROGRAM=${CMake_TEST_EXPLICIT_MAKE_PROGRAM}
        -DRunCMake_SOURCE_DIR=${CMAKE_CURRENT_SOURCE_DIR}/${test}
        -DRunCMake_BINARY_DIR=${CMAKE_CURRENT_BINARY_DIR}/${type}/${test}
        -Dconfig_file=${TEST_CONFIG_DIR}/${type}_config.cmake
        -P "${CMAKE_CURRENT_SOURCE_DIR}/${test}/RunCMakeTest.cmake"
        )
    endif()
  endforeach()
endfunction()

# Some tests use python for extra checks.
find_package(PythonInterp QUIET)

if(XCODE_VERSION AND "${XCODE_VERSION}" VERSION_LESS 6.1)
  set(Swift_ARGS -DXCODE_BELOW_6_1=1)
endif()

# Test MSVC for older host CMake versions, and test
# WIN32/CMAKE_C_COMPILER_ID to fix check on Intel for Windows.
if(MSVC OR (WIN32 AND CMAKE_C_COMPILER_ID MATCHES "MSVC|Intel"))
  set(LINKER_SUPPORTS_PDB 1)
else()
  set(LINKER_SUPPORTS_PDB 0)
endif()

add_RunCMake_test(CMP0019)
add_RunCMake_test(CMP0022)
add_RunCMake_test(CMP0026)
add_RunCMake_test(CMP0027)
add_RunCMake_test(CMP0037)
add_RunCMake_test(CMP0038)
add_RunCMake_test(CMP0039)
add_RunCMake_test(CMP0040)
add_RunCMake_test(CMP0041)
if(CMAKE_SYSTEM_NAME MATCHES Darwin AND CMAKE_SHARED_LIBRARY_RUNTIME_C_FLAG)
  add_RunCMake_test(CMP0042)
endif()
add_RunCMake_test(CMP0043)
add_RunCMake_test(CMP0045)
add_RunCMake_test(CMP0046)
add_RunCMake_test(CMP0049)
add_RunCMake_test(CMP0050)
add_RunCMake_test(CMP0051)
add_RunCMake_test(CMP0053)
add_RunCMake_test(CMP0054)
add_RunCMake_test(CMP0055)
add_RunCMake_test(CMP0057)
add_RunCMake_test(CMP0059)
add_RunCMake_test(CMP0060)
add_RunCMake_test(CMP0064)
if(CMAKE_SYSTEM_NAME MATCHES Darwin AND CMAKE_SHARED_LIBRARY_RUNTIME_C_FLAG)
  add_RunCMake_test(CMP0068)
endif()
add_RunCMake_test(CMP0069)
add_RunCMake_test(CMP0081)
add_RunCMake_test(CMP0102)
if(CMake_TEST_CUDA)
  add_RunCMake_test(CMP0104)
  set_property(TEST RunCMake.CMP0104 APPEND
    PROPERTY LABELS "CUDA")
endif()
add_RunCMake_test(CMP0106)
add_RunCMake_test(CMP0111)
add_RunCMake_test(CMP0115)
if(CMAKE_GENERATOR MATCHES "Ninja")
  add_RunCMake_test(CMP0116)
endif()
add_RunCMake_test(CMP0118)
add_RunCMake_test(CMP0119 -DCMAKE_C_COMPILER_ID=${CMAKE_C_COMPILER_ID})
add_RunCMake_test(CMP0121)
if (CMAKE_SYSTEM_NAME MATCHES "(Linux|Darwin)")
  add_RunCMake_test(CMP0125 -DCMAKE_SHARED_LIBRARY_PREFIX=${CMAKE_SHARED_LIBRARY_PREFIX}
                            -DCMAKE_SHARED_LIBRARY_SUFFIX=${CMAKE_SHARED_LIBRARY_SUFFIX})
endif()
add_RunCMake_test(CMP0126)

# The test for Policy 65 requires the use of the
# CMAKE_SHARED_LIBRARY_LINK_CXX_FLAGS variable, which both the VS and Xcode
# generators ignore.  The policy will have no effect on those generators.
if(NOT CMAKE_GENERATOR MATCHES "Visual Studio|Xcode")
  add_RunCMake_test(CMP0065 -DCMAKE_SYSTEM_NAME=${CMAKE_SYSTEM_NAME})
endif()
if(CMAKE_GENERATOR MATCHES "Make")
  add_RunCMake_test(Make -DMAKE_IS_GNU=${MAKE_IS_GNU})
endif()
unset(ninja_test_with_qt_version)
unset(ninja_qt_args)
if(CMake_TEST_Qt6)
  find_package(Qt6Widgets QUIET NO_MODULE)
  if(Qt6Widgets_FOUND)
    set(ninja_test_with_qt_version 6)
    # Work around Qt6 not finding sibling dependencies without CMAKE_PREFIX_PATH
    cmake_path(GET Qt6_DIR  PARENT_PATH base_dir)  # <base>/lib/cmake
    cmake_path(GET base_dir PARENT_PATH base_dir)  # <base>/lib
    cmake_path(GET base_dir PARENT_PATH base_dir)  # <base>
    set(ninja_qt_args -DCMAKE_PREFIX_PATH=${base_dir})
  endif()
elseif(CMake_TEST_Qt5)
  find_package(Qt5Widgets QUIET NO_MODULE)
  if(Qt5Widgets_FOUND)
    set(ninja_test_with_qt_version 5)
  endif()
endif()
if(CMAKE_GENERATOR MATCHES "Ninja")
  set(Ninja_ARGS
    -DCMAKE_C_OUTPUT_EXTENSION=${CMAKE_C_OUTPUT_EXTENSION}
    -DCMAKE_SHARED_LIBRARY_PREFIX=${CMAKE_SHARED_LIBRARY_PREFIX}
    -DCMAKE_SHARED_LIBRARY_SUFFIX=${CMAKE_SHARED_LIBRARY_SUFFIX})
  if(CMAKE_Fortran_COMPILER)
    list(APPEND Ninja_ARGS -DTEST_Fortran=1)
  endif()
  if(ninja_test_with_qt_version)
    list(APPEND Ninja_ARGS
      -DCMake_TEST_Qt_version=${ninja_test_with_qt_version}
      -DQt${ninja_test_with_qt_version}_DIR=${Qt${ninja_test_with_qt_version}_DIR}
      -DQt${ninja_test_with_qt_version}Core_DIR=${Qt${ninja_test_with_qt_version}Core_DIR}
      -DCMake_TEST_Qt${ninja_test_with_qt_version}Core_Version=${Qt${ninja_test_with_qt_version}Core_VERSION}
      -DQt${ninja_test_with_qt_version}Widgets_DIR=${Qt${ninja_test_with_qt_version}Widgets_DIR}
      ${ninja_qt_args}
    )
  endif()
  add_RunCMake_test(Ninja)
  set(NinjaMultiConfig_ARGS
    -DCYGWIN=${CYGWIN} -DMSYS=${MSYS}
    )
  if(ninja_test_with_qt_version)
    list(APPEND NinjaMultiConfig_ARGS
      -DCMake_TEST_Qt_version=${ninja_test_with_qt_version}
      -DQt${ninja_test_with_qt_version}Core_DIR=${Qt${ninja_test_with_qt_version}Core_DIR}
      -DCMake_TEST_Qt${ninja_test_with_qt_version}Core_Version=${Qt${ninja_test_with_qt_version}Core_VERSION}
      ${ninja_qt_args}
    )
  endif()
  if(DEFINED CMake_TEST_CUDA)
    list(APPEND NinjaMultiConfig_ARGS -DCMake_TEST_CUDA=${CMake_TEST_CUDA})
  endif()
  add_RunCMake_test(NinjaMultiConfig)
  set_property(TEST RunCMake.NinjaMultiConfig APPEND
    PROPERTY LABELS "CUDA")
endif()
add_RunCMake_test(CTest)

if(NOT CMake_TEST_EXTERNAL_CMAKE)
  add_RunCMake_test(ctest_memcheck
    -DPSEUDO_BC=$<TARGET_FILE:pseudo_BC>
    -DPSEUDO_PURIFY=$<TARGET_FILE:pseudo_purify>
    -DPSEUDO_VALGRIND=$<TARGET_FILE:pseudo_valgrind>
    -DPSEUDO_CUDA_SANITIZER=$<TARGET_FILE:pseudo_cuda-memcheck>
    -DPSEUDO_BC_NOLOG=$<TARGET_FILE:pseudonl_BC>
    -DPSEUDO_PURIFY_NOLOG=$<TARGET_FILE:pseudonl_purify>
    -DPSEUDO_VALGRIND_NOLOG=$<TARGET_FILE:pseudonl_valgrind>
    -DMEMCHECK_FAIL=$<TARGET_FILE:memcheck_fail>
    )
endif()

add_RunCMake_test(ABI -DCMake_TEST_CUDA=${CMake_TEST_CUDA})
set_property(TEST RunCMake.ABI APPEND
  PROPERTY LABELS "CUDA")

add_RunCMake_test(AndroidTestUtilities)
if(CMake_TEST_APPLE_SILICON)
  add_RunCMake_test(AppleSilicon)
endif()
set(want_NoQt_test TRUE)
if(CMake_TEST_Qt6 AND Qt6Widgets_FOUND)
  # Work around Qt6 not finding sibling dependencies without CMAKE_PREFIX_PATH
  cmake_path(GET Qt6_DIR  PARENT_PATH base_dir)  # <base>/lib/cmake
  cmake_path(GET base_dir PARENT_PATH base_dir)  # <base>/lib
  cmake_path(GET base_dir PARENT_PATH base_dir)  # <base>
  add_RunCMake_test(AutogenQt6 TEST_DIR Autogen
    -Dwith_qt_version=6
    "-DQt6_DIR:PATH=${Qt6_DIR}"
    "-DCMAKE_PREFIX_PATH:STRING=${base_dir}"
  )
  set(want_NoQt_test FALSE)
endif ()
if(CMake_TEST_Qt5 AND Qt5Widgets_FOUND)
  add_RunCMake_test(AutogenQt5 TEST_DIR Autogen
    -Dwith_qt_version=5
    "-DQt5_DIR:PATH=${Qt5_DIR}"
  )
  set(want_NoQt_test FALSE)
endif ()
if(want_NoQt_test)
  add_RunCMake_test(AutogenNoQt TEST_DIR Autogen)
endif()

add_RunCMake_test(ArtifactOutputDirs)

if(NOT DEFINED CMake_TEST_BuildDepends_GNU_AS
    AND (CMAKE_C_COMPILER_ID STREQUAL "GNU" OR CMAKE_C_COMPILER_ID STREQUAL "LCC")
    AND CMAKE_GENERATOR MATCHES "^Ninja"
    )
  execute_process(COMMAND "${CMAKE_C_COMPILER}" -print-prog-name=as
    RESULT_VARIABLE _gnu_res
    OUTPUT_VARIABLE _gnu_as OUTPUT_STRIP_TRAILING_WHITESPACE ERROR_QUIET)
  if(_gnu_res EQUAL 0 AND _gnu_as)
    set(CMake_TEST_BuildDepends_GNU_AS "${_gnu_as}")
  endif()
endif()

add_RunCMake_test(BuildDepends
  -DMSVC_VERSION=${MSVC_VERSION}
  -DCMAKE_C_COMPILER_ID=${CMAKE_C_COMPILER_ID}
  -DCMake_TEST_BuildDepends_GNU_AS=${CMake_TEST_BuildDepends_GNU_AS}
  )
if(UNIX AND "${CMAKE_GENERATOR}" MATCHES "Unix Makefiles|Ninja")
  add_RunCMake_test(Byproducts)
endif()
add_RunCMake_test(CMakeDependentOption)
add_RunCMake_test(CMakeRoleGlobalProperty)
add_RunCMake_test(CMakeRelease -DCMake_TEST_JQ=${CMake_TEST_JQ})
if(UNIX AND "${CMAKE_GENERATOR}" MATCHES "Unix Makefiles|Ninja")
  add_RunCMake_test(CompilerChange)
endif()
add_RunCMake_test(CompilerNotFound)
add_RunCMake_test(Configure -DMSVC_IDE=${MSVC_IDE})
add_RunCMake_test(DisallowedCommands)
if("${CMAKE_GENERATOR}" MATCHES "Unix Makefiles|Ninja")
  add_RunCMake_test(ExportCompileCommands)
endif()
add_RunCMake_test(ExcludeFromAll)
add_RunCMake_test(ExternalData)
add_RunCMake_test(FeatureSummary)
add_RunCMake_test(FPHSA)
add_RunCMake_test(FileAPI -DPYTHON_EXECUTABLE=${PYTHON_EXECUTABLE}
                          -DCMAKE_CXX_COMPILER_ID=${CMAKE_CXX_COMPILER_ID})
add_RunCMake_test(FindBoost)
add_RunCMake_test(FindLua)
add_RunCMake_test(FindOpenGL)
add_RunCMake_test(InitialFlags)
if(CMake_TEST_FindOpenSSL)
  add_RunCMake_test(FindOpenSSL)
endif()
if(CMake_TEST_UseSWIG)
  add_RunCMake_test(FindSWIG)
  add_RunCMake_test(UseSWIG -DCMake_TEST_FindPython=${CMake_TEST_FindPython})
endif()
if(NOT CMAKE_C_COMPILER_ID MATCHES "Watcom")
  add_RunCMake_test(GenerateExportHeader)
endif()
add_RunCMake_test(GenEx-COMPILE_LANGUAGE)
add_RunCMake_test(GenEx-COMPILE_LANG_AND_ID)
add_RunCMake_test(GenEx-LINK_LANGUAGE)
add_RunCMake_test(GenEx-LINK_LANG_AND_ID)
add_RunCMake_test(GenEx-HOST_LINK)
add_RunCMake_test(GenEx-DEVICE_LINK)
add_RunCMake_test(GenEx-TARGET_FILE -DLINKER_SUPPORTS_PDB=${LINKER_SUPPORTS_PDB})
add_RunCMake_test(GenEx-GENEX_EVAL)
add_RunCMake_test(GenEx-TARGET_RUNTIME_DLLS)
add_RunCMake_test(GeneratorExpression)
add_RunCMake_test(GeneratorInstance)
add_RunCMake_test(GeneratorPlatform)
if(XCODE_VERSION)
  set(GeneratorToolset_ARGS -DXCODE_VERSION=${XCODE_VERSION})
endif()
add_RunCMake_test(GeneratorToolset)
add_RunCMake_test(GetPrerequisites)
add_RunCMake_test(GNUInstallDirs -DSYSTEM_NAME=${CMAKE_SYSTEM_NAME})
add_RunCMake_test(GoogleTest) # Note: does not actually depend on Google Test
add_RunCMake_test(Graphviz)
add_RunCMake_test(TargetPropertyGeneratorExpressions)
add_RunCMake_test(Languages)
add_RunCMake_test(LinkItemValidation)
add_RunCMake_test(LinkStatic)
if(CMAKE_CXX_COMPILER_ID MATCHES "^(Cray|PGI|NVHPC|XL|XLClang|IBMClang|Fujitsu|FujitsuClang)$")
  add_RunCMake_test(MetaCompileFeatures)
endif()
if(MSVC)
  add_RunCMake_test(MSVCRuntimeLibrary)
  add_RunCMake_test(MSVCRuntimeTypeInfo)
  add_RunCMake_test(MSVCWarningFlags)
endif()
add_RunCMake_test(ObjectLibrary)
add_RunCMake_test(ParseImplicitIncludeInfo)
add_RunCMake_test(ParseImplicitLinkInfo)
if(UNIX AND CMAKE_SHARED_LIBRARY_RUNTIME_C_FLAG AND CMAKE_EXECUTABLE_FORMAT STREQUAL "ELF")
  add_RunCMake_test(RuntimePath)
endif()
add_RunCMake_test(ScriptMode)
add_RunCMake_test(Swift -DCMAKE_Swift_COMPILER=${CMAKE_Swift_COMPILER} -DCMAKE_SYSTEM_NAME=${CMAKE_SYSTEM_NAME})
add_RunCMake_test(TargetObjects)
add_RunCMake_test(TargetProperties)
add_RunCMake_test(ToolchainFile)
add_RunCMake_test(find_dependency)
add_RunCMake_test(CompileDefinitions)
add_RunCMake_test(CompileFeatures -DCMake_NO_C_STANDARD=${CMake_NO_C_STANDARD} -DCMake_NO_CXX_STANDARD=${CMake_NO_CXX_STANDARD})
add_RunCMake_test(Policy)
add_RunCMake_test(PolicyScope)
add_RunCMake_test(WriteBasicConfigVersionFile)
add_RunCMake_test(WriteCompilerDetectionHeader)
add_RunCMake_test(SourceProperties)
if(NOT WIN32)
  add_RunCMake_test(PositionIndependentCode -DCMAKE_SYSTEM_NAME=${CMAKE_SYSTEM_NAME}
                                            -DCMAKE_CXX_COMPILER_ID=${CMAKE_CXX_COMPILER_ID})
endif()
if(NOT CMAKE_GENERATOR MATCHES "Visual Studio")
  add_RunCMake_test(VisibilityPreset)
endif()
if (QT4_FOUND)
  set(CompatibleInterface_ARGS -DQT_QMAKE_EXECUTABLE:FILEPATH=${QT_QMAKE_EXECUTABLE})
endif()
add_RunCMake_test(CompatibleInterface)
add_RunCMake_test(Syntax)
add_RunCMake_test(WorkingDirectory)
add_RunCMake_test(MaxRecursionDepth)

add_RunCMake_test(add_custom_command)
add_RunCMake_test(add_custom_target)
add_RunCMake_test(add_dependencies)
add_RunCMake_test(add_executable)
add_RunCMake_test(add_library)
add_RunCMake_test(add_subdirectory)
add_RunCMake_test(add_test)
add_RunCMake_test(build_command)
add_executable(exit_code exit_code.c)
set(execute_process_ARGS
  -DEXIT_CODE_EXE=$<TARGET_FILE:exit_code>
  -DPYTHON_EXECUTABLE=${PYTHON_EXECUTABLE}
  )
if(NOT CMake_TEST_EXTERNAL_CMAKE)
  list(APPEND execute_process_ARGS -DTEST_ENCODING_EXE=$<TARGET_FILE:testEncoding>)
endif()
add_RunCMake_test(execute_process)
add_RunCMake_test(export)
add_RunCMake_test(cmake_host_system_information)
add_RunCMake_test(cmake_language)
add_RunCMake_test(cmake_minimum_required)
add_RunCMake_test(cmake_parse_arguments)
add_RunCMake_test(cmake_path -DMSYS=${MSYS})
add_RunCMake_test(continue)
add_executable(color_warning color_warning.c)
add_executable(fake_build_command fake_build_command.c)
add_RunCMake_test(ctest_build
  -DCOLOR_WARNING=$<TARGET_FILE:color_warning>
  -DFAKE_BUILD_COMMAND_EXE=$<TARGET_FILE:fake_build_command>
)
add_RunCMake_test(ctest_cmake_error)
add_RunCMake_test(ctest_configure)
if(COVERAGE_COMMAND)
  add_RunCMake_test(ctest_coverage -DCOVERAGE_COMMAND=${COVERAGE_COMMAND})
endif()
add_RunCMake_test(ctest_start)
add_RunCMake_test(ctest_submit)
add_RunCMake_test(ctest_test
  -DIMAGE_DIR=${CMAKE_SOURCE_DIR}/Utilities/Sphinx/static
)
add_RunCMake_test(ctest_disabled_test)
add_RunCMake_test(ctest_skipped_test)
add_RunCMake_test(ctest_update)
add_RunCMake_test(ctest_upload)
add_RunCMake_test(ctest_environment)
add_RunCMake_test(ctest_fixtures)
add_RunCMake_test(define_property)
add_RunCMake_test(file -DCYGWIN=${CYGWIN} -DMSYS=${MSYS})
add_RunCMake_test(file-CHMOD -DMSYS=${MSYS})
add_RunCMake_test(file-RPATH -DCMAKE_SYSTEM_NAME=${CMAKE_SYSTEM_NAME})
add_RunCMake_test(find_file)
add_RunCMake_test(find_library -DCYGWIN=${CYGWIN} -DMSYS=${MSYS})
add_RunCMake_test(find_package -DMSYS=${MSYS})
add_RunCMake_test(find_path)
add_RunCMake_test(find_program -DCMAKE_SYSTEM_NAME=${CMAKE_SYSTEM_NAME})
add_RunCMake_test(foreach)
add_RunCMake_test(function)
add_RunCMake_test(get_filename_component)
add_RunCMake_test(get_property)
add_RunCMake_test(if)
add_RunCMake_test(include)
add_RunCMake_test(include_directories)
add_RunCMake_test(include_guard)
add_RunCMake_test(list)
add_RunCMake_test(load_cache)
add_RunCMake_test(math)
add_RunCMake_test(message)
add_RunCMake_test(option)
add_RunCMake_test(project -DCMake_TEST_RESOURCES=${CMake_TEST_RESOURCES})
add_RunCMake_test(project_injected)
add_RunCMake_test(return)
add_RunCMake_test(separate_arguments)
add_RunCMake_test(set_property)
add_RunCMake_test(string)
add_RunCMake_test(test_include_dirs)
add_RunCMake_test(BundleUtilities)
if(APPLE)
  add_RunCMake_test(INSTALL_NAME_DIR)
  add_RunCMake_test(MacOSVersions)
endif()

function(add_RunCMake_test_try_compile)
  if(CMAKE_VERSION VERSION_LESS 3.9.20170907 AND "x${CMAKE_CXX_COMPILER_ID}" STREQUAL "xMSVC")
    # Older CMake versions do not know about MSVC language standards.
    # Approximate our logic from MSVC-CXX.cmake.
    if ((NOT CMAKE_CXX_COMPILER_VERSION VERSION_LESS 19.0.24215.1 AND
         CMAKE_CXX_COMPILER_VERSION VERSION_LESS 19.10) OR
         NOT CMAKE_CXX_COMPILER_VERSION VERSION_LESS 19.10.25017)
      set(CMAKE_CXX_STANDARD_DEFAULT 14)
    elseif (NOT CMAKE_CXX_COMPILER_VERSION VERSION_LESS 16.0)
      set(CMAKE_CXX_STANDARD_DEFAULT "")
    else()
      unset(CMAKE_CXX_STANDARD_DEFAULT)
    endif()
  endif()
  if(CMAKE_VERSION VERSION_LESS 3.18.20200813 AND "x${CMAKE_C_COMPILER_ID}" STREQUAL "xMSVC")
    # Older CMake versions do not know about MSVC language standards.
    # Approximate our logic from MSVC-C.cmake.
    if(NOT CMAKE_C_COMPILER_VERSION VERSION_LESS 19.27)
      set(CMAKE_C_STANDARD_DEFAULT 99)
    else()
      set(CMAKE_C_STANDARD_DEFAULT "")
    endif()
  endif()
  if(CMAKE_VERSION VERSION_LESS 3.20.20210225 AND "x${CMAKE_C_COMPILER_ID}" STREQUAL "xClang")
    # Older CMake versions do not know about Clang MSVC compatibility mode
    # standards. Approximate the logic from Clang-C.cmake.
    if(CMAKE_C_COMPILER_VERSION VERSION_GREATER_EQUAL 11.0)
      set(CMAKE_C_STANDARD_DEFAULT 17)
    elseif(CMAKE_C_COMPILER_VERSION VERSION_GREATER_EQUAL 3.5.2)
      set(CMAKE_C_STANDARD_DEFAULT 11)
    endif()
  endif()
  if(CMAKE_VERSION VERSION_LESS 3.20.6 AND "x${CMAKE_C_COMPILER_ID}" STREQUAL "xIntelLLVM" AND "x${CMAKE_C_SIMULATE_ID}" STREQUAL "xMSVC")
    # Older CMake versions accidentally set the default standards to empty when
    # IntelLLVM targets the MSVC ABI, thus not activating standard selection.
    # Approximate the logic from IntelLLVM-{C,CXX}.cmake.
    if(DEFINED CMAKE_C_STANDARD_DEFAULT AND "${CMAKE_C_STANDARD_DEFAULT}" STREQUAL "")
      set(CMAKE_C_STANDARD_DEFAULT 17)
    endif()
    if(DEFINED CMAKE_CXX_STANDARD_DEFAULT AND "${CMAKE_CXX_STANDARD_DEFAULT}" STREQUAL "")
      set(CMAKE_CXX_STANDARD_DEFAULT 14)
    endif()
  endif()
  foreach(var
      CMAKE_SYSTEM_NAME
      CMAKE_C_COMPILER_ID
      CMAKE_C_COMPILER_VERSION
      CMAKE_C_STANDARD_DEFAULT
      CMAKE_CXX_COMPILER_ID
      CMAKE_CXX_COMPILER_VERSION
      CMAKE_CXX_STANDARD_DEFAULT
      CMake_TEST_CUDA
      CMake_TEST_ISPC
      CMake_TEST_HIP
      CMake_TEST_FILESYSTEM_1S
      CMAKE_OBJC_STANDARD_DEFAULT
      CMAKE_OBJCXX_STANDARD_DEFAULT
      )
    if(DEFINED ${var})
      list(APPEND try_compile_ARGS -D${var}=${${var}})
    endif()
  endforeach()
  add_RunCMake_test(try_compile)
  set_property(TEST RunCMake.try_compile APPEND
    PROPERTY LABELS "CUDA;ISPC")
endfunction()
add_RunCMake_test_try_compile()

add_RunCMake_test(try_run -DCMAKE_SYSTEM_NAME=${CMAKE_SYSTEM_NAME}
                          -DCMAKE_C_COMPILER_ID=${CMAKE_C_COMPILER_ID})
add_RunCMake_test(set)
add_RunCMake_test(variable_watch)
add_RunCMake_test(while)
add_RunCMake_test(CMP0004)
add_RunCMake_test(TargetPolicies)
add_RunCMake_test(alias_targets)
add_RunCMake_test(InterfaceLibrary)
add_RunCMake_test(no_install_prefix)
add_RunCMake_test(configure_file)
add_RunCMake_test(CTestTimeout -DTIMEOUT=${CTestTestTimeout_TIME})
add_RunCMake_test(CTestTimeoutAfterMatch)
if(CMake_TEST_CUDA)
  add_RunCMake_test(CUDA_architectures)
  set_property(TEST RunCMake.CUDA_architectures APPEND PROPERTY LABELS "CUDA")
endif()
add_RunCMake_test(DependencyGraph -DCMAKE_Fortran_COMPILER=${CMAKE_Fortran_COMPILER})

# ctresalloc links against CMakeLib and CTestLib, which means it can't be built
# if CMake_TEST_EXTERNAL_CMAKE is activated (the compiler might be different.)
# So, it has to be provided in the original build tree.
if(CMake_TEST_EXTERNAL_CMAKE)
  set(no_package_root_path)
  if(NOT CMAKE_VERSION VERSION_LESS 3.12)
    set(no_package_root_path NO_PACKAGE_ROOT_PATH)
  endif()
  find_program(ctresalloc ctresalloc PATHS ${CMake_TEST_EXTERNAL_CMAKE}
    NO_DEFAULT_PATH
    ${no_package_root_path}
    NO_CMAKE_PATH
    NO_CMAKE_ENVIRONMENT_PATH
    NO_SYSTEM_ENVIRONMENT_PATH
    NO_CMAKE_SYSTEM_PATH
    NO_CMAKE_FIND_ROOT_PATH
    )
  if(ctresalloc)
    add_executable(ctresalloc IMPORTED)
    set_property(TARGET ctresalloc PROPERTY IMPORTED_LOCATION ${ctresalloc})
  endif()
else()
  add_executable(ctresalloc CTestResourceAllocation/ctresalloc.cxx)
  target_link_libraries(ctresalloc CTestLib)
  target_include_directories(ctresalloc PRIVATE
    ${CMake_BINARY_DIR}/Source
    ${CMake_SOURCE_DIR}/Source
    ${CMake_SOURCE_DIR}/Source/CTest
    )
  set_property(TARGET ctresalloc PROPERTY RUNTIME_OUTPUT_DIRECTORY ${CMake_BIN_DIR})
endif()

if(TARGET ctresalloc)
  add_RunCMake_test(CTestResourceAllocation -DCTRESALLOC_COMMAND=$<TARGET_FILE:ctresalloc>)
else()
  message(STATUS "Could not find ctresalloc")
endif()

if(NOT WIN32
    AND NOT MSYS # FIXME: This works on CYGWIN but not on MSYS
    )
  add_RunCMake_test(SymlinkTrees)
endif ()

find_package(Qt4 QUIET)
find_package(Qt5Core QUIET)
if (QT4_FOUND AND Qt5Core_FOUND AND NOT Qt5Core_VERSION VERSION_LESS 5.1.0)
  add_RunCMake_test(IncompatibleQt)
endif()
if (QT4_FOUND)
  add_RunCMake_test(ObsoleteQtMacros -DQT_QMAKE_EXECUTABLE:FILEPATH=${QT_QMAKE_EXECUTABLE})
endif()

find_package(PkgConfig QUIET)
if(PKG_CONFIG_FOUND)
  add_RunCMake_test(FindPkgConfig)
endif()

if(CMake_TEST_FindGTK2)
  add_RunCMake_test(FindGTK2)
endif()

if("${CMAKE_GENERATOR}" MATCHES "Visual Studio")
  add_RunCMake_test(CMAKE_MSVCIDE_RUN_PATH)
  add_RunCMake_test(include_external_msproject -DVS_PLATFORM_NAME=${CMAKE_VS_PLATFORM_NAME})
  if("${CMAKE_GENERATOR}" MATCHES "Visual Studio (9|10)" AND NOT CMAKE_VS_DEVENV_COMMAND)
    set(NO_USE_FOLDERS 1)
  endif()
  add_RunCMake_test(VSSolution -DNO_USE_FOLDERS=${NO_USE_FOLDERS})
endif()

if("${CMAKE_GENERATOR}" MATCHES "Visual Studio ([^9]|9[0-9])")
  add_RunCMake_test(VS10Project
    -DCMAKE_C_COMPILER_ID=${CMAKE_C_COMPILER_ID}
    -DCMAKE_C_COMPILER_VERSION=${CMAKE_C_COMPILER_VERSION}
    )
  if( vs12 AND wince )
    add_RunCMake_test( VS10ProjectWinCE "-DRunCMake_GENERATOR_PLATFORM=${wince_sdk}")
  endif()
endif()

if(CMAKE_GENERATOR MATCHES "^Visual Studio (1[6-9]|[2-9][0-9])")
  add_RunCMake_test(VsDotnetSdk)
  add_RunCMake_test(VsNugetPackageRestore)
endif()

if(XCODE_VERSION)
  add_RunCMake_test(XcodeProject -DXCODE_VERSION=${XCODE_VERSION})
  add_RunCMake_test(XcodeProject-Embed -DXCODE_VERSION=${XCODE_VERSION})

  # This test can take a very long time due to lots of combinations.
  # Use a long default timeout and provide an option to customize it.
  if(NOT DEFINED CMake_TEST_XcodeProject_TIMEOUT)
    set(CMake_TEST_XcodeProject_TIMEOUT 2000)
  endif()
  set_property(TEST RunCMake.XcodeProject PROPERTY TIMEOUT ${CMake_TEST_XcodeProject_TIMEOUT})
endif()

if(CMAKE_C_COMPILER_ID STREQUAL "AppleClang"
    AND NOT CMAKE_C_COMPILER_VERSION VERSION_LESS 6.0)
  add_RunCMake_test(Framework)
endif()

add_RunCMake_test(File_Archive)
add_RunCMake_test(File_Configure)
add_RunCMake_test(File_Generate)
add_RunCMake_test(ExportWithoutLanguage)
add_RunCMake_test(target_link_directories)
add_RunCMake_test(target_link_libraries)
add_RunCMake_test(target_link_libraries-ALIAS)
add_RunCMake_test(target_link_libraries-LINK_LANGUAGE)
add_RunCMake_test(target_link_libraries-LINK_LANG_AND_ID)
add_RunCMake_test(add_link_options -DCMAKE_C_COMPILER_ID=${CMAKE_C_COMPILER_ID})
add_RunCMake_test(target_link_options -DCMAKE_C_COMPILER_ID=${CMAKE_C_COMPILER_ID}
                                      -DCMake_TEST_CUDA=${CMake_TEST_CUDA})
set_property(TEST RunCMake.target_link_options APPEND
  PROPERTY LABELS "CUDA")

add_RunCMake_test(target_compile_definitions)
add_RunCMake_test(target_compile_features)
add_RunCMake_test(target_compile_options
  -DCMAKE_C_COMPILER_ID=${CMAKE_C_COMPILER_ID}
  -DCMAKE_C_SIMULATE_ID=${CMAKE_C_SIMULATE_ID}
  )
add_RunCMake_test(target_include_directories)
add_RunCMake_test(target_sources)
add_RunCMake_test(CheckCompilerFlag   -DCMake_TEST_CUDA=${CMake_TEST_CUDA}
                                      -DCMake_TEST_ISPC=${CMake_TEST_ISPC}
                                      -DCMAKE_Fortran_COMPILER_ID=${CMAKE_Fortran_COMPILER_ID}
                                      -DCMake_TEST_HIP=${CMake_TEST_HIP})
add_RunCMake_test(CheckSourceCompiles -DCMake_TEST_CUDA=${CMake_TEST_CUDA}
                                      -DCMake_TEST_ISPC=${CMake_TEST_ISPC}
                                      -DCMAKE_Fortran_COMPILER_ID=${CMAKE_Fortran_COMPILER_ID}
                                      -DCMake_TEST_HIP=${CMake_TEST_HIP})
add_RunCMake_test(CheckSourceRuns     -DCMake_TEST_CUDA=${CMake_TEST_CUDA}
                                      -DCMAKE_Fortran_COMPILER_ID=${CMAKE_Fortran_COMPILER_ID}
                                      -DCMake_TEST_HIP=${CMake_TEST_HIP})
set_property(TEST RunCMake.CheckCompilerFlag
                  RunCMake.CheckSourceCompiles
                  RunCMake.CheckSourceRuns
    APPEND PROPERTY LABELS "CUDA")
set_property(TEST RunCMake.CheckSourceCompiles
                  RunCMake.CheckCompilerFlag
    APPEND PROPERTY LABELS "ISPC")
set_property(TEST RunCMake.CheckCompilerFlag
                  RunCMake.CheckSourceCompiles
                  RunCMake.CheckSourceRuns
    APPEND PROPERTY LABELS "HIP")
add_RunCMake_test(CheckModules)
add_RunCMake_test(CheckIPOSupported)
if (CMAKE_SYSTEM_NAME MATCHES "(Linux|Darwin)"
    AND (CMAKE_C_COMPILER_ID MATCHES "Clang|GNU|LCC" OR CMAKE_Fortran_COMPILER_ID MATCHES "GNU|LCC"))
  add_RunCMake_test(CheckLinkerFlag -DCMAKE_C_COMPILER_ID=${CMAKE_C_COMPILER_ID}
                                    -DCMAKE_Fortran_COMPILER_ID=${CMAKE_Fortran_COMPILER_ID}
                                    -DCMake_TEST_CUDA=${CMake_TEST_CUDA}
                                    -DCMake_TEST_HIP=${CMake_TEST_HIP})
  set_property(TEST RunCMake.CheckLinkerFlag APPEND PROPERTY LABELS "CUDA")
  set_property(TEST RunCMake.CheckLinkerFlag APPEND PROPERTY LABELS "HIP")
endif()


add_executable(pseudo_llvm-rc pseudo_llvm-rc.c)
add_RunCMake_test(CommandLine -DLLVM_RC=$<TARGET_FILE:pseudo_llvm-rc> -DCMAKE_SYSTEM_NAME=${CMAKE_SYSTEM_NAME}
                  -DCYGWIN=${CYGWIN} -DMSYS=${MSYS} -DPYTHON_EXECUTABLE=${PYTHON_EXECUTABLE})
add_RunCMake_test(CommandLineTar)

if(CMAKE_PLATFORM_NO_VERSIONED_SONAME OR (NOT CMAKE_SHARED_LIBRARY_SONAME_FLAG AND NOT CMAKE_SHARED_LIBRARY_SONAME_C_FLAG))
  set(NO_NAMELINK 1)
else()
  set(NO_NAMELINK 0)
endif()

add_RunCMake_test(install -DNO_NAMELINK=${NO_NAMELINK} -DCYGWIN=${CYGWIN} -DMSYS=${MSYS}
  -DCMAKE_SHARED_LIBRARY_RPATH_ORIGIN_TOKEN=${CMAKE_SHARED_LIBRARY_RPATH_ORIGIN_TOKEN}
  -DCMAKE_SYSTEM_NAME=${CMAKE_SYSTEM_NAME}
  -DCMAKE_SHARED_LIBRARY_RUNTIME_C_FLAG=${CMAKE_SHARED_LIBRARY_RUNTIME_C_FLAG}
  -DCMAKE_EXECUTABLE_FORMAT=${CMAKE_EXECUTABLE_FORMAT}
  -DCMake_TEST_ISPC=${CMake_TEST_ISPC}
  )
set_property(TEST RunCMake.install APPEND
  PROPERTY LABELS "ISPC")

add_RunCMake_test(file-GET_RUNTIME_DEPENDENCIES
  -DCMAKE_C_COMPILER_ID=${CMAKE_C_COMPILER_ID}
  -DCMake_INSTALL_NAME_TOOL_BUG=${CMake_INSTALL_NAME_TOOL_BUG}
  )

add_RunCMake_test(CPackCommandLine)
add_RunCMake_test(CPackConfig)
add_RunCMake_test(CPackInstallProperties)
if(XCODE_VERSION)
  set(ExternalProject_ARGS -DXCODE_VERSION=${XCODE_VERSION})
endif()
add_RunCMake_test(ExternalProject)
add_RunCMake_test(FetchContent)
set(CTestCommandLine_ARGS -DPYTHON_EXECUTABLE=${PYTHON_EXECUTABLE})
if(NOT CMake_TEST_EXTERNAL_CMAKE)
  list(APPEND CTestCommandLine_ARGS -DTEST_AFFINITY=$<TARGET_FILE:testAffinity>)
endif()
add_executable(print_stdin print_stdin.c)
add_RunCMake_test(CTestCommandLine -DTEST_PRINT_STDIN=$<TARGET_FILE:print_stdin>)
add_RunCMake_test(CacheNewline)
# Only run this test on unix platforms that support
# symbolic links
if(UNIX)
  add_RunCMake_test(CPackSymlinks)
endif()

set(IfacePaths_INCDIRS_ARGS -DTEST_PROP=INCLUDE_DIRECTORIES)
add_RunCMake_test(IfacePaths_INCDIRS TEST_DIR IfacePaths)

set(IfacePaths_SOURCES_ARGS -DTEST_PROP=SOURCES)
add_RunCMake_test(IfacePaths_SOURCES TEST_DIR IfacePaths)

# Matlab module related tests
if(CMake_TEST_FindMatlab OR CMake_TEST_FindMatlab_MCR OR (NOT "${CMake_TEST_FindMatlab_MCR_ROOT_DIR}" STREQUAL ""))
    set(FindMatlab_additional_test_options )
    if(CMake_TEST_FindMatlab_MCR OR NOT "${CMake_TEST_FindMatlab_MCR_ROOT_DIR}" STREQUAL "")
      set(FindMatlab_additional_test_options -DIS_MCR=TRUE)
    endif()
    if(NOT "${CMake_TEST_FindMatlab_MCR_ROOT_DIR}" STREQUAL "")
      set(FindMatlab_additional_test_options ${FindMatlab_additional_test_options} "-DMCR_ROOT:FILEPATH=${CMake_TEST_FindMatlab_MCR_ROOT_DIR}")
    endif()

  add_RunCMake_test(FindMatlab ${FindMatlab_additional_test_options})
endif()

add_executable(pseudo_emulator pseudo_emulator.c)
add_executable(pseudo_emulator_custom_command pseudo_emulator_custom_command.c)
add_executable(pseudo_emulator_custom_command_arg pseudo_emulator_custom_command_arg.c)
add_RunCMake_test(CrosscompilingEmulator
 -DPSEUDO_EMULATOR=$<TARGET_FILE:pseudo_emulator>
 -DPSEUDO_EMULATOR_CUSTOM_COMMAND=$<TARGET_FILE:pseudo_emulator_custom_command>
 -DPSEUDO_EMULATOR_CUSTOM_COMMAND_ARG=$<TARGET_FILE:pseudo_emulator_custom_command_arg>)
if("${CMAKE_GENERATOR}" MATCHES "Make|Ninja")
  if(UNIX AND NOT CYGWIN)
    execute_process(COMMAND ldd --help
      OUTPUT_VARIABLE LDD_HELP
      ERROR_VARIABLE LDD_ERR)
    if("${LDD_HELP}" MATCHES
        "(-r, --function-relocs.*process data and function relocations.*-u, --unused.*print unused direct dependencies)")
      add_RunCMake_test(LinkWhatYouUse)
    endif()
  endif()
  add_executable(pseudo_tidy pseudo_tidy.c)
  add_executable(pseudo_iwyu pseudo_iwyu.c)
  add_executable(pseudo_cpplint pseudo_cpplint.c)
  add_executable(pseudo_cppcheck pseudo_cppcheck.c)
  add_RunCMake_test(ClangTidy -DPSEUDO_TIDY=$<TARGET_FILE:pseudo_tidy>)
  add_RunCMake_test(IncludeWhatYouUse -DPSEUDO_IWYU=$<TARGET_FILE:pseudo_iwyu>)
  add_RunCMake_test(Cpplint -DPSEUDO_CPPLINT=$<TARGET_FILE:pseudo_cpplint>)
  add_RunCMake_test(Cppcheck -DPSEUDO_CPPCHECK=$<TARGET_FILE:pseudo_cppcheck>)
  add_RunCMake_test(MultiLint
    -DPSEUDO_TIDY=$<TARGET_FILE:pseudo_tidy>
    -DPSEUDO_IWYU=$<TARGET_FILE:pseudo_iwyu>
    -DPSEUDO_CPPLINT=$<TARGET_FILE:pseudo_cpplint>
    -DPSEUDO_CPPCHECK=$<TARGET_FILE:pseudo_cppcheck>
    )
  if(DEFINED CMake_TEST_CUDA)
    list(APPEND CompilerLauncher_ARGS -DCMake_TEST_CUDA=${CMake_TEST_CUDA})
  endif()
  if(DEFINED CMake_TEST_HIP)
    list(APPEND CompilerLauncher_ARGS -DCMake_TEST_HIP=${CMake_TEST_HIP})
  endif()
  if(DEFINED CMake_TEST_ISPC)
    list(APPEND CompilerLauncher_ARGS -DCMake_TEST_ISPC=${CMake_TEST_ISPC})
  endif()
  if(CMAKE_Fortran_COMPILER)
    list(APPEND CompilerLauncher_ARGS -DCMake_TEST_Fortran=1)
  endif()
  if (APPLE AND CMAKE_C_COMPILER_ID MATCHES "Clang|GNU")
    list(APPEND CompilerLauncher_ARGS -DCMake_TEST_OBJC=1)
    list(APPEND LinkerLauncher_ARGS -DCMake_TEST_OBJC=1)
  endif()
  add_RunCMake_test(CompilerLauncher)
  set_property(TEST RunCMake.CompilerLauncher APPEND
    PROPERTY LABELS "CUDA;HIP;ISPC")
  add_RunCMake_test(ctest_labels_for_subprojects)
  add_RunCMake_test(CompilerArgs)
  add_RunCMake_test(LinkerLauncher)
endif()

set(cpack_tests
  DEB.CUSTOM_NAMES
  DEB.DEBUGINFO
  DEB.DEFAULT_PERMISSIONS
  DEB.DEPENDENCIES
  DEB.EMPTY_DIR
  DEB.VERSION
  DEB.EXTRA
  DEB.GENERATE_SHLIBS
  DEB.GENERATE_SHLIBS_LDCONFIG
  DEB.LONG_FILENAMES
  DEB.MINIMAL
  DEB.PER_COMPONENT_FIELDS
  DEB.TIMESTAMPS
  DEB.MD5SUMS
  DEB.DEB_PACKAGE_VERSION_BACK_COMPATIBILITY
  DEB.DEB_DESCRIPTION
  DEB.PROJECT_META

  RPM.CUSTOM_BINARY_SPEC_FILE
  RPM.CUSTOM_NAMES
  RPM.DEBUGINFO
  RPM.DEFAULT_PERMISSIONS
  RPM.DEPENDENCIES
  RPM.DIST
  RPM.EMPTY_DIR
  RPM.VERSION
  RPM.INSTALL_SCRIPTS
  RPM.MAIN_COMPONENT
  RPM.MINIMAL
  RPM.PARTIALLY_RELOCATABLE_WARNING
  RPM.PER_COMPONENT_FIELDS
  RPM.SINGLE_DEBUGINFO
  RPM.EXTRA_SLASH_IN_PATH
  RPM.SOURCE_PACKAGE
  RPM.SUGGESTS
  RPM.SYMLINKS
  RPM.USER_FILELIST
  RPM.PROJECT_META

  7Z
  TBZ2
  TGZ
  TXZ
  TZ
  ZIP
  STGZ
  External
  )
if(APPLE)
  list(APPEND cpack_tests DragNDrop)
endif()
add_RunCMake_test_group(CPack "${cpack_tests}")
# add a test to make sure symbols are exported from a shared library
# for MSVC compilers CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS property is used
add_RunCMake_test(AutoExportDll
  -DCMAKE_SYSTEM_NAME=${CMAKE_SYSTEM_NAME}
  -DCMAKE_CXX_COMPILER_ID=${CMAKE_CXX_COMPILER_ID}
  )

add_RunCMake_test(AndroidMK)

if(CMake_TEST_ANDROID_NDK OR CMake_TEST_ANDROID_STANDALONE_TOOLCHAIN)
  if(NOT "${CMAKE_GENERATOR}" MATCHES "Make|Ninja|Visual Studio 1[456]")
    message(FATAL_ERROR "Android tests supported only by Makefile, Ninja, and Visual Studio >= 14 generators")
  endif()
  foreach(v TEST_ANDROID_NDK TEST_ANDROID_STANDALONE_TOOLCHAIN)
    if(CMake_${v})
      string(REPLACE ";" "|" ${v} "${CMake_${v}}")
      list(APPEND Android_ARGS "-D${v}=${${v}}")
    endif()
  endforeach()

  add_RunCMake_test(Android)

  # This test can take a very long time due to lots of combinations.
  # Use a long default timeout and provide an option to customize it.
  if(NOT DEFINED CMake_TEST_ANDROID_TIMEOUT)
    set(CMake_TEST_ANDROID_TIMEOUT 3000)
  endif()
  set_property(TEST RunCMake.Android PROPERTY TIMEOUT ${CMake_TEST_ANDROID_TIMEOUT})
endif()

if(${CMAKE_GENERATOR} MATCHES "Visual Studio ([^9]|9[0-9])")
  add_RunCMake_test(CSharpCustomCommand)
  add_RunCMake_test(CSharpReferenceImport)
endif()

add_RunCMake_test("CTestCommandExpandLists")

add_RunCMake_test(PrecompileHeaders -DCMAKE_C_COMPILER_ID=${CMAKE_C_COMPILER_ID}
  -DCMAKE_C_SIMULATE_ID=${CMAKE_C_SIMULATE_ID}
  -DCMAKE_C_COMPILER_VERSION=${CMAKE_C_COMPILER_VERSION})

add_RunCMake_test("UnityBuild")
add_RunCMake_test(CMakePresets
  -DPYTHON_EXECUTABLE=${PYTHON_EXECUTABLE}
  -DCMake_TEST_JSON_SCHEMA=${CMake_TEST_JSON_SCHEMA}
  )
add_RunCMake_test(CMakePresetsBuild
  -DPYTHON_EXECUTABLE=${PYTHON_EXECUTABLE}
  -DCMake_TEST_JSON_SCHEMA=${CMake_TEST_JSON_SCHEMA}
  -DCMAKE_SYSTEM_NAME=${CMAKE_SYSTEM_NAME}
  )
add_RunCMake_test(CMakePresetsTest
  -DPYTHON_EXECUTABLE=${PYTHON_EXECUTABLE}
  -DCMake_TEST_JSON_SCHEMA=${CMake_TEST_JSON_SCHEMA}
  )

if(${CMAKE_GENERATOR} MATCHES "Make|Ninja")
  add_RunCMake_test(TransformDepfile)
endif()

if(WIN32)
  add_RunCMake_test(Win32GenEx)
endif()

if("${CMAKE_C_COMPILER_ID}" STREQUAL "LCC" OR "${CMAKE_CXX_COMPILER_ID}" STREQUAL "LCC" OR "${CMAKE_Fortran_COMPILER_ID}" STREQUAL "LCC")
  add_RunCMake_test("CMP0129")
endif()
