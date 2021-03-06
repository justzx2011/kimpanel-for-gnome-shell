cmake_minimum_required(VERSION 2.6)

project(kimpanel-for-gnome-shell)

find_package(Gettext REQUIRED)
find_program(ZIP zip)
find_program(UNZIP unzip)

if (NOT ZIP)
    message(FATAL_ERROR "zip not found")
endif (NOT ZIP)
if (NOT UNZIP)
    message(FATAL_ERROR "unzip not found")
endif (NOT UNZIP)

add_subdirectory(po)

set(datadir "${CMAKE_INSTALL_PREFIX}/share")
set(topextensiondir "${datadir}/gnome-shell/extensions")
set(extensionname "kimpanel")
set(uuid "${extensionname}@kde.org")
set(extensiondir "${topextensiondir}/${uuid}")
set(localextensiondir "$ENV{HOME}/.local/share/gnome-shell/extensions/${uuid}")
set(localedir "${datadir}/locale")

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/metadata.json.in ${CMAKE_CURRENT_BINARY_DIR}/metadata.json @ONLY)

set(extension_FILES
    extension.js
    indicator.js
    panel.js
    lib.js
    ${CMAKE_CURRENT_BINARY_DIR}/metadata.json
    stylesheet.css
)

install(FILES ${extension_FILES} DESTINATION ${extensiondir})

add_custom_command(OUTPUT ${uuid}.zip
    COMMAND ${CMAKE_COMMAND} -E remove_directory "${CMAKE_CURRENT_BINARY_DIR}/_build"
    COMMAND ${CMAKE_COMMAND} -E make_directory "${CMAKE_CURRENT_BINARY_DIR}/_build"
    COMMAND ${CMAKE_BUILD_TOOL} install DESTDIR="${CMAKE_CURRENT_BINARY_DIR}/_build"
    COMMAND ${CMAKE_COMMAND} -E copy_directory "${CMAKE_CURRENT_BINARY_DIR}/_build/${extensiondir}" "${CMAKE_CURRENT_BINARY_DIR}/_build/${uuid}/";
    COMMAND ${CMAKE_COMMAND} -E copy_directory "${CMAKE_CURRENT_BINARY_DIR}/_build/${datadir}/locale" "${CMAKE_CURRENT_BINARY_DIR}/_build/${uuid}/locale";
    COMMAND cd "${CMAKE_CURRENT_BINARY_DIR}/_build/${uuid}" && ${ZIP} -qr "${CMAKE_CURRENT_BINARY_DIR}/${uuid}.zip" .
)

add_custom_target(install-zip
    DEPENDS ${uuid}.zip
    COMMAND ${CMAKE_COMMAND} -E make_directory "${localextensiondir}"
    COMMAND ${UNZIP} -o "${CMAKE_CURRENT_BINARY_DIR}/${uuid}.zip" -d "${localextensiondir}"
)

add_custom_target(uninstall-zip
    COMMAND ${CMAKE_COMMAND} -E remove_directory "${localextensiondir}"
)