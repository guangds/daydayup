cmake_minimum_required(VERSION 3.16...3.25)

legacy_check()

if(NOT ENABLE_DECKLINK)
  target_disable(decklink-captions)
  return()
endif()

find_qt(COMPONENTS Widgets)

add_library(decklink-captions MODULE)
add_library(OBS::decklink-captions ALIAS decklink-captions)

target_sources(decklink-captions PRIVATE decklink-captions.cpp decklink-captions.h forms/captions.ui)
target_link_libraries(decklink-captions PRIVATE OBS::frontend-api OBS::libobs Qt::Widgets)

if(OS_WINDOWS)
  configure_file(cmake/windows/obs-module.rc.in decklink-captions.rc)
  target_sources(decklink-captions PRIVATE decklink-captions.rc)

  set_property(
    TARGET decklink-captions
    APPEND
    PROPERTY AUTORCC_OPTIONS --format-version 1)
elseif(OS_MACOS)
  find_library(COCOA Cocoa)
  mark_as_advanced(COCOA)
  target_link_libraries(decklink-captions PRIVATE ${COCOA})
  target_compile_options(decklink-captions PRIVATE -Wno-quoted-include-in-framework-header -Wno-comma)
elseif(OS_LINUX OR OS_FREEBSD)
  find_package(X11 REQUIRED)
  mark_as_advanced(X11)
  target_link_libraries(decklink-captions PRIVATE X11::X11)
endif()

set_target_properties_obs(
  decklink-captions
  PROPERTIES FOLDER plugins/decklink
             PREFIX ""
             AUTOMOC ON
             AUTOUIC ON
             AUTORCC ON
             AUTOUIC_SEARCH_PATHS forms)
