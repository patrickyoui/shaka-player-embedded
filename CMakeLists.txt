cmake_minimum_required(VERSION 3.2)

include(cmake/HunterGate.cmake)
HunterGate(
    URL "https://github.com/ruslo/hunter/archive/v0.23.165.tar.gz"
    SHA1 "5a73f91df5f6109c0bb1104d0c0ee423f7bece79"
    LOCAL
)

project(shaka_player_embedded VERSION 0.1.0)

option(SHAKA_PLAYER_EMBEDDED_BUILD_DEMO "Build demo" OFF)

# See 'js_engine', BUILD.gn
# Values: JSC, V8
if(IOS)
  set(js_default "JSC")
else()
  set(js_default "V8")
endif()

set(SHAKA_PLAYER_EMBEDDED_JS_ENGINE ${js_default} CACHE STRING "js engine")

find_package(Python 2 REQUIRED)

set(gen_dir ${CMAKE_CURRENT_BINARY_DIR}/generated)
file(MAKE_DIRECTORY ${gen_dir}/shaka)

set(shaka_version_h ${gen_dir}/shaka/version.h)
set(shaka_version_py ${CMAKE_CURRENT_LIST_DIR}/shaka/tools/version.py)

add_custom_command(
    OUTPUT
    ${shaka_version_h}
    COMMAND
    ${Python_EXECUTABLE}
    ${shaka_version_py}
    --output
    ${shaka_version_h}
    DEPENDS
    ${shaka_version_py}
)

set(shaka_config_h ${gen_dir}/shaka_config.h)
set(shaka_config_py ${CMAKE_CURRENT_LIST_DIR}/shaka/tools/gen_config_h.py)

add_custom_command(
    OUTPUT
    ${shaka_config_h}
    COMMAND
    ${Python_EXECUTABLE}
    ${shaka_config_py}
    --output
    ${shaka_config_h}
    --sdl-utils
    DEPENDS
    ${shaka_config_py}
)

set(eme_plugins_cc ${gen_dir}/eme_plugins.cc)
set(eme_implementations ${CMAKE_CURRENT_LIST_DIR}/shaka/src/eme/clearkey.json)
set(eme_plugins_py ${CMAKE_CURRENT_LIST_DIR}/shaka/tools/gen_eme_plugins.py)

add_custom_command(
    OUTPUT
    ${eme_plugins_cc}
    COMMAND
    ${Python_EXECUTABLE}
    ${eme_plugins_py}
    --output
    ${eme_plugins_cc}
    ${eme_implementations}
    DEPENDS
    ${eme_plugins_py}
    ${eme_implementations}
)

# Script 'shaka/tools/eme_plugin_build_info.py' outputs
# nothing in current configuration

set(
    exposed_type_generator_py
    ${CMAKE_CURRENT_LIST_DIR}/shaka/tools/idl/exposed_type_generator.py
)

set(
    idl_list
    manifest
    player_externs
    stats
    track
)

set(public_dir "${gen_dir}/shaka")
file(MAKE_DIRECTORY ${public_dir})

set(internal_dir "${gen_dir}/shaka/src/js")
file(MAKE_DIRECTORY ${internal_dir})

set(idl_shaka_sources "")

foreach(idl_source ${idl_list})
  set(input ${CMAKE_CURRENT_LIST_DIR}/shaka/src/js/${idl_source}.idl)

  set(
      outputs
      ${public_dir}/${idl_source}.h
      ${public_dir}/${idl_source}.cc
      ${internal_dir}/${idl_source}.h
      ${internal_dir}/${idl_source}_js.cc
      ${public_dir}/${idl_source}_objc.h
      ${public_dir}/${idl_source}_objc.mm
      ${internal_dir}/${idl_source}+Internal.h
  )

  list(
      APPEND
      idl_shaka_sources
      ${public_dir}/${idl_source}.cc
      ${internal_dir}/${idl_source}_js.cc
  )
  if(APPLE)
    list(
        APPEND
        idl_shaka_sources
        ${public_dir}/${idl_source}_objc.mm
    )
  endif()

  add_custom_command(
      OUTPUT
      ${outputs}
      COMMAND
      ${Python_EXECUTABLE}
      ${exposed_type_generator_py}
      --internal-dir ${internal_dir}
      --public-dir ${public_dir}
      --internal-rel-dir "../src/js" # Generated files are same as with native build
      ${input}
      DEPENDS
      ${exposed_type_generator_py}
      ${CMAKE_CURRENT_LIST_DIR}/shaka/tools/webidl/webidl/idl_parser.py
      ${CMAKE_CURRENT_LIST_DIR}/shaka/tools/webidl/webidl/idl_tokenizer.py
      ${input}
  )
endforeach()

# BUILD.gn
# source_set("internal_sources")
set(
    sources
    shaka/src/core/environment.cc
    shaka/src/core/environment.h
    shaka/src/core/js_manager_impl.cc
    shaka/src/core/js_manager_impl.h
    shaka/src/core/member.h
    shaka/src/core/network_thread.cc
    shaka/src/core/network_thread.h
    shaka/src/core/ref_ptr.h
    shaka/src/core/rejected_promise_handler.cc
    shaka/src/core/rejected_promise_handler.h
    shaka/src/core/task_runner.cc
    shaka/src/core/task_runner.h
    shaka/src/debug/mutex.h
    shaka/src/debug/thread.cc
    shaka/src/debug/thread.h
    shaka/src/debug/thread_event.cc
    shaka/src/debug/thread_event.h
    shaka/src/debug/waitable.cc
    shaka/src/debug/waitable.h
    shaka/src/debug/waiting_tracker.cc
    shaka/src/debug/waiting_tracker.h
    shaka/src/eme/clearkey_implementation.cc
    shaka/src/eme/clearkey_implementation.h
    shaka/src/eme/clearkey_implementation_factory.cc
    shaka/src/eme/clearkey_implementation_factory.h
    shaka/src/eme/implementation.cc
    shaka/src/js/base_64.cc
    shaka/src/js/base_64.h
    shaka/src/js/console.cc
    shaka/src/js/console.h
    shaka/src/js/debug.cc
    shaka/src/js/debug.h
    shaka/src/js/dom/attr.cc
    shaka/src/js/dom/attr.h
    shaka/src/js/dom/character_data.cc
    shaka/src/js/dom/character_data.h
    shaka/src/js/dom/comment.cc
    shaka/src/js/dom/comment.h
    shaka/src/js/dom/container_node.cc
    shaka/src/js/dom/container_node.h
    shaka/src/js/dom/document.cc
    shaka/src/js/dom/document.h
    shaka/src/js/dom/dom_exception.cc
    shaka/src/js/dom/dom_exception.h
    shaka/src/js/dom/dom_parser.cc
    shaka/src/js/dom/dom_parser.h
    shaka/src/js/dom/element.cc
    shaka/src/js/dom/element.h
    shaka/src/js/dom/exception_code.h
    shaka/src/js/dom/node.cc
    shaka/src/js/dom/node.h
    shaka/src/js/dom/text.cc
    shaka/src/js/dom/text.h
    shaka/src/js/dom/xml_document_parser.cc
    shaka/src/js/dom/xml_document_parser.h
    shaka/src/js/eme/implementation_helper_impl.cc
    shaka/src/js/eme/implementation_helper_impl.h
    shaka/src/js/eme/implementation_registry.cc
    shaka/src/js/eme/media_key_session.cc
    shaka/src/js/eme/media_key_session.h
    shaka/src/js/eme/media_key_system_access.cc
    shaka/src/js/eme/media_key_system_access.h
    shaka/src/js/eme/media_key_system_configuration.h
    shaka/src/js/eme/media_keys.cc
    shaka/src/js/eme/media_keys.h
    shaka/src/js/eme/search_registry.cc
    shaka/src/js/eme/search_registry.h
    shaka/src/js/events/event.cc
    shaka/src/js/events/event.h
    shaka/src/js/events/event_names.h
    shaka/src/js/events/event_target.cc
    shaka/src/js/events/event_target.h
    shaka/src/js/events/media_encrypted_event.cc
    shaka/src/js/events/media_encrypted_event.h
    shaka/src/js/events/media_key_message_event.cc
    shaka/src/js/events/media_key_message_event.h
    shaka/src/js/events/progress_event.cc
    shaka/src/js/events/progress_event.h
    shaka/src/js/js_error.cc
    shaka/src/js/js_error.h
    shaka/src/js/location.cc
    shaka/src/js/location.h
    shaka/src/js/mse/media_error.cc
    shaka/src/js/mse/media_error.h
    shaka/src/js/mse/media_source.cc
    shaka/src/js/mse/media_source.h
    shaka/src/js/mse/source_buffer.cc
    shaka/src/js/mse/source_buffer.h
    shaka/src/js/mse/text_track.cc
    shaka/src/js/mse/text_track.h
    shaka/src/js/mse/time_ranges.cc
    shaka/src/js/mse/time_ranges.h
    shaka/src/js/mse/video_element.cc
    shaka/src/js/mse/video_element.h
    shaka/src/js/navigator.cc
    shaka/src/js/navigator.h
    shaka/src/js/test_type.cc
    shaka/src/js/test_type.h
    shaka/src/js/timeouts.cc
    shaka/src/js/timeouts.h
    shaka/src/js/url.cc
    shaka/src/js/url.h
    shaka/src/js/vtt_cue.cc
    shaka/src/js/vtt_cue.h
    shaka/src/js/xml_http_request.cc
    shaka/src/js/xml_http_request.h
    shaka/src/mapping/any.cc
    shaka/src/mapping/any.h
    shaka/src/mapping/backing_object.cc
    shaka/src/mapping/backing_object.h
    shaka/src/mapping/backing_object_factory.cc
    shaka/src/mapping/backing_object_factory.h
    shaka/src/mapping/byte_buffer.cc
    shaka/src/mapping/byte_buffer.h
    shaka/src/mapping/byte_string.cc
    shaka/src/mapping/byte_string.h
    shaka/src/mapping/callback.cc
    shaka/src/mapping/callback.h
    shaka/src/mapping/convert_js.h
    shaka/src/mapping/enum.h
    shaka/src/mapping/exception_or.h
    shaka/src/mapping/generic_converter.h
    shaka/src/mapping/js_engine.h
    shaka/src/mapping/js_wrappers.h
    shaka/src/mapping/js_utils.cc
    shaka/src/mapping/js_utils.h
    shaka/src/mapping/names.h
    shaka/src/mapping/promise.cc
    shaka/src/mapping/promise.h
    shaka/src/mapping/register_member.cc
    shaka/src/mapping/register_member.h
    shaka/src/mapping/struct.cc
    shaka/src/mapping/struct.h
    shaka/src/mapping/weak_js_ptr.h
    shaka/src/media/audio_renderer.cc
    shaka/src/media/audio_renderer.h
    shaka/src/media/base_frame.cc
    shaka/src/media/base_frame.h
    shaka/src/media/decoder_thread.cc
    shaka/src/media/decoder_thread.h
    shaka/src/media/demuxer_thread.cc
    shaka/src/media/demuxer_thread.h
    shaka/src/media/ffmpeg_decoded_frame.cc
    shaka/src/media/ffmpeg_decoded_frame.h
    shaka/src/media/ffmpeg_encoded_frame.cc
    shaka/src/media/ffmpeg_encoded_frame.h
    shaka/src/media/frame_buffer.cc
    shaka/src/media/frame_buffer.h
    shaka/src/media/frame_converter.cc
    shaka/src/media/frame_converter.h
    shaka/src/media/frame_drawer.cc
    shaka/src/media/frame_drawer.h
    shaka/src/media/locked_frame_list.cc
    shaka/src/media/locked_frame_list.h
    shaka/src/media/media_processor.cc
    shaka/src/media/media_processor.h
    shaka/src/media/media_utils.cc
    shaka/src/media/media_utils.h
    shaka/src/media/pipeline_manager.cc
    shaka/src/media/pipeline_manager.h
    shaka/src/media/pipeline_monitor.cc
    shaka/src/media/pipeline_monitor.h
    shaka/src/media/renderer.cc
    shaka/src/media/renderer.h
    shaka/src/media/stream.cc
    shaka/src/media/stream.h
    shaka/src/media/types.cc
    shaka/src/media/types.h
    shaka/src/media/video_controller.cc
    shaka/src/media/video_controller.h
    shaka/src/media/video_renderer.cc
    shaka/src/media/video_renderer.h
    shaka/src/memory/heap_tracer.cc
    shaka/src/memory/heap_tracer.h
    shaka/src/memory/object_tracker.cc
    shaka/src/memory/object_tracker.h
    shaka/src/public/data.cc
    shaka/src/public/eme_promise.cc
    shaka/src/public/eme_promise_impl.h
    shaka/src/public/error.cc
    shaka/src/public/frame.cc
    shaka/src/public/js_manager.cc
    shaka/src/public/optional.cc
    shaka/src/public/player.cc
    shaka/src/public/shaka_utils.cc
    shaka/src/public/text_track_public.cc
    shaka/src/public/video.cc
    shaka/src/public/vtt_cue_public.cc
    shaka/src/util/buffer_reader.cc
    shaka/src/util/buffer_reader.h
    shaka/src/util/clock.cc
    shaka/src/util/clock.h
    shaka/src/util/crypto.h
    shaka/src/util/decryptor.h
    shaka/src/util/dynamic_buffer.cc
    shaka/src/util/dynamic_buffer.h
    shaka/src/util/file_system.cc
    shaka/src/util/file_system.h
    shaka/src/util/js_wrapper.h
    shaka/src/util/macros.h
    shaka/src/util/objc_utils.h
    shaka/src/util/shared_lock.cc
    shaka/src/util/shared_lock.h
    shaka/src/util/templates.h
    shaka/src/util/utils.cc
    shaka/src/util/utils.h

    shaka/src/util/file_system_posix.cc

    shaka/src/public/sdl_frame_drawer.cc

    shaka/src/util/crypto_openssl.cc
    shaka/src/util/decryptor_openssl.cc

    ${shaka_version_h}
    ${shaka_config_h}
    ${eme_plugins_cc}
    ${idl_shaka_sources}
)

if("${SHAKA_PLAYER_EMBEDDED_JS_ENGINE}" STREQUAL "JSC")
  list(
      APPEND
      sources

      # } else if (js_engine == "jsc") {

      shaka/src/mapping/jsc/js_engine.cc
      shaka/src/mapping/jsc/js_wrappers.cc
      shaka/src/mapping/jsc/jsc_utils.cc
      shaka/src/mapping/jsc/jsc_utils.h
  )
elseif("${SHAKA_PLAYER_EMBEDDED_JS_ENGINE}" STREQUAL "V8")
  list(
      APPEND
      sources

      # if (js_engine == "v8") {
      #   sources += [

      shaka/src/mapping/v8/js_engine.cc
      shaka/src/mapping/v8/js_wrappers.cc
      shaka/src/mapping/v8/v8_utils.cc
      shaka/src/mapping/v8/v8_utils.h
      shaka/src/memory/v8_heap_tracer.cc
      shaka/src/memory/v8_heap_tracer.h
  )
endif()

if(IOS)
  list(
      APPEND
      sources
      shaka/src/public/ShakaPlayerView.mm
  )
endif()

# shared_library("player_shared")
add_library(shaka_player_embedded ${sources})
add_library(
    shaka_player_embedded::shaka_player_embedded
    ALIAS
    shaka_player_embedded
)

if(IOS)
  set_target_properties(
     shaka_player_embedded
     PROPERTIES
     XCODE_ATTRIBUTE_CLANG_ENABLE_OBJC_ARC YES
  )
endif()

if(APPLE)
  target_compile_options(
      shaka_player_embedded PRIVATE "-Wno-c++11-narrowing"
  )
endif()

target_compile_definitions(
    shaka_player_embedded
    PUBLIC
    $<BUILD_INTERFACE:OS_POSIX>
    $<BUILD_INTERFACE:BUILDING_SHAKA>
)

if("${SHAKA_PLAYER_EMBEDDED_JS_ENGINE}" STREQUAL "JSC")
  target_compile_definitions(
      shaka_player_embedded
      PUBLIC
      $<BUILD_INTERFACE:USING_JSC>
  )
elseif("${SHAKA_PLAYER_EMBEDDED_JS_ENGINE}" STREQUAL "V8")
  hunter_add_package(v8)
  find_package(v8 CONFIG REQUIRED)
  target_link_libraries(
      shaka_player_embedded
      PUBLIC
      v8::v8_base
      v8::v8_init
      v8::v8_initializers
      v8::v8_libbase
      v8::v8_libplatform
      v8::v8_libsampler
      v8::v8_nosnapshot
  )

  target_compile_definitions(
      shaka_player_embedded
      PUBLIC
      $<BUILD_INTERFACE:USING_V8>
  )
else()
  message(FATAL_ERROR "Unknow value")
endif()

if(IOS)
  target_compile_definitions(
      shaka_player_embedded
      PUBLIC
      $<BUILD_INTERFACE:OS_IOS>
      $<BUILD_INTERFACE:PLATFORM="iOS">
  )
  find_package(javascriptcore REQUIRED)
  target_link_libraries(
      shaka_player_embedded
      PUBLIC
      javascriptcore::javascriptcore
  )
elseif(APPLE)
  target_compile_definitions(
      shaka_player_embedded
      PUBLIC
      $<BUILD_INTERFACE:OS_MAC>
      $<BUILD_INTERFACE:PLATFORM="macOS x64">
  )
elseif("${CMAKE_SYSTEM_NAME}" STREQUAL "Linux")
  target_compile_definitions(
      shaka_player_embedded
      PUBLIC
      $<BUILD_INTERFACE:PLATFORM="Linux x64">
  )
endif()

target_include_directories(
    shaka_player_embedded
    PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/shaka>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/shaka/include>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/shaka/include/shaka>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/shaka/src/eme> # for eme plugins
    $<BUILD_INTERFACE:${gen_dir}>
)

hunter_add_package(glog)
find_package(glog CONFIG REQUIRED)
target_link_libraries(shaka_player_embedded PUBLIC glog::glog)

hunter_add_package(CURL)
find_package(CURL CONFIG REQUIRED)
target_link_libraries(shaka_player_embedded PUBLIC CURL::libcurl)

hunter_add_package(libxml2)
find_package(libxml-2.0 CONFIG REQUIRED)
target_link_libraries(shaka_player_embedded PUBLIC PkgConfig::libxml-2.0)

hunter_add_package(SDL2)
find_package(SDL2 CONFIG REQUIRED)
target_link_libraries(shaka_player_embedded PUBLIC SDL2::SDL2)

hunter_add_package(ffmpeg)
find_package(ffmpeg CONFIG REQUIRED)
target_link_libraries(
    shaka_player_embedded
    PUBLIC
    ffmpeg::avcodec
    ffmpeg::avformat
    ffmpeg::avutil
    ffmpeg::swresample
    ffmpeg::swscale
)

target_include_directories(
    shaka_player_embedded
    PUBLIC
    $<BUILD_INTERFACE:${public_dir}>
)

### Installation (https://github.com/forexample/package-example) {

# Introduce variables:
# * CMAKE_INSTALL_LIBDIR
# * CMAKE_INSTALL_BINDIR
# * CMAKE_INSTALL_INCLUDEDIR
include(GNUInstallDirs)

set(config_install_dir "${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}")
set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")
set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(TARGETS_EXPORT_NAME "${PROJECT_NAME}Targets")
set(NAMESPACE "${PROJECT_NAME}::")

include(CMakePackageConfigHelpers)

# Use:
# * PROJECT_VERSION
write_basic_package_version_file(
    "${version_config}" COMPATIBILITY SameMajorVersion
)

# Use:
# * TARGETS_EXPORT_NAME
# * PROJECT_NAME
configure_package_config_file(
    "cmake/template/Config.cmake.in"
    "${project_config}"
    INSTALL_DESTINATION "${config_install_dir}"
)

install(
    FILES "${project_config}" "${version_config}"
    DESTINATION "${config_install_dir}"
)

install(
    EXPORT "${TARGETS_EXPORT_NAME}"
    NAMESPACE "${NAMESPACE}"
    DESTINATION "${config_install_dir}"
)

install(
    DIRECTORY shaka/include/
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

install(
    FILES
    ${public_dir}/manifest.h
    ${public_dir}/player_externs.h
    ${public_dir}/stats.h
    ${public_dir}/track.h
    ${gen_dir}/shaka_config.h
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/shaka
)

### }

install(
    TARGETS shaka_player_embedded
    EXPORT "${TARGETS_EXPORT_NAME}"
    LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}"
    ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}"
    RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}"
    INCLUDES DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}"
)

if(SHAKA_PLAYER_EMBEDDED_BUILD_DEMO)
  add_executable(demo shaka/demo/main.cc)

  target_link_libraries(demo PRIVATE shaka_player_embedded)

  hunter_add_package(gflags)
  find_package(gflags CONFIG REQUIRED)
  target_link_libraries(demo PUBLIC gflags)
endif()

if(IOS)
  add_subdirectory(sample_xcode_project/sample_xcode_project)
endif()
