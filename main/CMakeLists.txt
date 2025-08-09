set(SOURCES
    "audio/audio_codec.cc"
    "audio/audio_service.cc"
    "audio/codecs/es8311_audio_codec.cc"
    "audio/processors/no_audio_processor.cc"
    "display/display.cc"
    "display/lcd_display.cc"
    "protocols/protocol.cc"
    "protocols/mqtt_protocol.cc"
    "protocols/websocket_protocol.cc"
    "mcp_server.cc"
    "system_info.cc"
    "application.cc"
    "ota.cc"
    "settings.cc"
    "device_state_event.cc"
    "main.cc"
)

set(INCLUDE_DIRS "." "display" "audio" "protocols")

# Add common board files
file(GLOB BOARD_COMMON_SOURCES ${CMAKE_CURRENT_SOURCE_DIR}/boards/common/*.cc)
list(APPEND SOURCES ${BOARD_COMMON_SOURCES})
list(APPEND INCLUDE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/boards/common)

# Fixed board type
set(BOARD_TYPE "waveshare-c6-lcd-1.69")

# Add board-specific files
file(GLOB BOARD_SOURCES
    ${CMAKE_CURRENT_SOURCE_DIR}/boards/${BOARD_TYPE}/*.cc
    ${CMAKE_CURRENT_SOURCE_DIR}/boards/${BOARD_TYPE}/*.c
)
list(APPEND SOURCES ${BOARD_SOURCES})
list(APPEND INCLUDE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/boards/${BOARD_TYPE})

# Wake word sources
if(CONFIG_USE_AFE_WAKE_WORD)
    list(APPEND SOURCES "audio/wake_words/afe_wake_word.cc")
elseif(CONFIG_USE_ESP_WAKE_WORD)
    list(APPEND SOURCES "audio/wake_words/esp_wake_word.cc")
elseif(CONFIG_USE_CUSTOM_WAKE_WORD)
    list(APPEND SOURCES "audio/wake_words/custom_wake_word.cc")
endif()

# Language assets
set(LANG_DIR "en-US")
set(LANG_JSON "${CMAKE_CURRENT_SOURCE_DIR}/assets/${LANG_DIR}/language.json")
set(LANG_HEADER "${CMAKE_CURRENT_SOURCE_DIR}/assets/lang_config.h")
file(GLOB LANG_SOUNDS ${CMAKE_CURRENT_SOURCE_DIR}/assets/${LANG_DIR}/*.p3)
file(GLOB COMMON_SOUNDS ${CMAKE_CURRENT_SOURCE_DIR}/assets/common/*.p3)

idf_component_register(SRCS ${SOURCES}
                    EMBED_FILES ${LANG_SOUNDS} ${COMMON_SOUNDS}
                    INCLUDE_DIRS ${INCLUDE_DIRS}
                    WHOLE_ARCHIVE)

if(NOT BOARD_NAME)
    set(BOARD_NAME ${BOARD_TYPE})
endif()
target_compile_definitions(${COMPONENT_LIB}
                    PRIVATE BOARD_TYPE="${BOARD_TYPE}" BOARD_NAME="${BOARD_NAME}")

add_custom_command(
    OUTPUT ${LANG_HEADER}
    COMMAND python ${PROJECT_DIR}/scripts/gen_lang.py
            --input "${LANG_JSON}"
            --output "${LANG_HEADER}"
    DEPENDS
        ${LANG_JSON}
        ${PROJECT_DIR}/scripts/gen_lang.py
    COMMENT "Generating ${LANG_DIR} language config"
)

add_custom_target(lang_header ALL
    DEPENDS ${LANG_HEADER}
)
