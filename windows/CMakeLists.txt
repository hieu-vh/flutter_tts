cmake_minimum_required(VERSION 3.14)
set(PROJECT_NAME "flutter_tts")
project(${PROJECT_NAME} LANGUAGES CXX)

################ NuGet install begin ################
find_program(NUGET_EXE NAMES nuget)
if(NOT NUGET_EXE)
	message(FATAL_ERROR "NUGET.EXE not found. Please install this executable, and run CMake again.")
endif()

# Ensure NuGet package installation for Microsoft.Windows.CppWinRT
execute_process(
    COMMAND ${NUGET_EXE} install "Microsoft.Windows.CppWinRT" -Version 2.0.210503.1 -ExcludeVersion -OutputDirectory ${CMAKE_BINARY_DIR}/packages
    RESULT_VARIABLE nuget_result
    ERROR_VARIABLE nuget_error
)

if(NOT nuget_result EQUAL 0)
    message(FATAL_ERROR "Failed to install NuGet package: ${nuget_error}")
endif()
################ NuGet install end ################

# This value is used when generating builds using this plugin, so it must not be changed
set(PLUGIN_NAME "flutter_tts_plugin")

add_library(${PLUGIN_NAME} SHARED
  "flutter_tts_plugin.cpp"
)

apply_standard_settings(${PLUGIN_NAME})

set_target_properties(${PLUGIN_NAME} PROPERTIES
  CXX_VISIBILITY_PRESET hidden
  VS_GLOBAL_KEYWORD "Win32Proj"
)

target_compile_definitions(${PLUGIN_NAME} PRIVATE FLUTTER_PLUGIN_IMPL)

target_include_directories(${PLUGIN_NAME} INTERFACE
  "${CMAKE_CURRENT_SOURCE_DIR}/include"
)

target_link_libraries(${PLUGIN_NAME} PRIVATE flutter flutter_wrapper_plugin)

# List of absolute paths to libraries that should be bundled with the plugin
set(flutter_tts_bundled_libraries
  ""
  PARENT_SCOPE
)

################ NuGet import begin ################
# Link with CppWinRT props and targets
set_target_properties(${PLUGIN_NAME} PROPERTIES VS_PROJECT_IMPORT
  ${CMAKE_BINARY_DIR}/packages/Microsoft.Windows.CppWinRT/build/native/Microsoft.Windows.CppWinRT.props
)

target_link_libraries(${PLUGIN_NAME} PRIVATE
  ${CMAKE_BINARY_DIR}/packages/Microsoft.Windows.CppWinRT/build/native/Microsoft.Windows.CppWinRT.targets
)
################ NuGet import end ################
