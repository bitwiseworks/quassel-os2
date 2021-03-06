# Convenience function to avoid boilerplate
function(setup_executable _target _define)
    set_target_properties(${_target} PROPERTIES
        COMPILE_FLAGS ${_define}
        RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}
    )
    target_link_libraries(${_target} PRIVATE ${ARGN})
    install(TARGETS ${_target} RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})
endfunction()

if (HAVE_UMASK)
    set_property(SOURCE main.cpp APPEND PROPERTY COMPILE_DEFINITIONS HAVE_UMASK)
endif()

if (WITH_BUNDLED_ICONS)
    set_property(SOURCE main.cpp APPEND PROPERTY COMPILE_DEFINITIONS WITH_BUNDLED_ICONS)
endif()

# Windows icon resource
if(WIN32)
    if(MINGW)
        find_program(WINDRES_EXECUTABLE NAMES windres)
        if(WINDRES_EXECUTABLE)
            exec_program(windres
                ARGS "-i ${CMAKE_SOURCE_DIR}/pics/win32.rc"
                     "-o ${CMAKE_CURRENT_BINARY_DIR}/win32.o"
                     "--include-dir=${CMAKE_SOURCE_DIR}/pics"
                )
            set(WIN_RC ${CMAKE_CURRENT_BINARY_DIR}/win32.o)
        endif()
    else()
        set(WIN_RC ${CMAKE_SOURCE_DIR}/pics/win32.rc)
    endif()
endif()

if(OS2)
    enable_language(RC)
    set(OS2_RC ${CMAKE_SOURCE_DIR}/pics/os2.rc)
    set_source_files_properties(${CMAKE_SOURCE_DIR}/pics/os2.rc PROPERTIES LANGUAGE RC)
endif()

# Build the executables
if (WANT_CORE)
    add_executable(quasselcore main.cpp ${WIN_RC})
    setup_executable(quasselcore -DBUILD_CORE Qt5::Core Quassel::Core)
  if(OS2)
    set_target_properties(quasselcore PROPERTIES
      OS2_DEF_VENDOR "$ENV{VENDOR}"
      OS2_DEF_PATCH "${QUASSEL_PATCH}"
      OS2_DEF_VERSION "${QUASSEL_VERSION_STRING}")
  endif(OS2)
endif()

if (WANT_QTCLIENT)
  if(OS2)
    add_executable(quasselclient WIN32 main.cpp ${OS2_RC})
  else()
    add_executable(quasselclient WIN32 main.cpp ${WIN_RC})
  endif()
    setup_executable(quasselclient -DBUILD_QTUI Qt5::Core Qt5::Gui Quassel::QtUi)
    if (WITH_KDE)
        target_link_libraries(quasselclient PRIVATE KF5::CoreAddons)
    endif()
  if(OS2)
    set_target_properties(quasselclient PROPERTIES
      OS2_DEF_VENDOR "$ENV{VENDOR}"
      OS2_DEF_PATCH "${QUASSEL_PATCH}"
      OS2_DEF_VERSION "${QUASSEL_VERSION_STRING}"
      OS2_DEF_EXEType "WINDOWAPI")
  endif(OS2)
endif()

if (WANT_MONO)
  if(OS2)
    add_executable(quassel WIN32 main.cpp monoapplication.cpp ${OS2_RC})
  else()
    add_executable(quassel WIN32 main.cpp monoapplication.cpp ${WIN_RC})
  endif()
    setup_executable(quassel -DBUILD_MONO Qt5::Core Qt5::Gui Quassel::Core Quassel::QtUi)
    if (WITH_KDE)
        target_link_libraries(quassel PRIVATE KF5::CoreAddons)
    endif()
  if(OS2)
    set_target_properties(quassel PROPERTIES
      OS2_DEF_VENDOR "$ENV{VENDOR}"
      OS2_DEF_PATCH "${QUASSEL_PATCH}"
      OS2_DEF_VERSION "${QUASSEL_VERSION_STRING}"
      OS2_DEF_EXEType "WINDOWAPI")
  endif(OS2)
endif()

# Build bundles for MacOSX
if (APPLE)
    if (WANT_QTCLIENT)
        add_custom_command(TARGET quasselclient POST_BUILD
                           COMMAND ${CMAKE_SOURCE_DIR}/scripts/build/macosx_makebundle.py
                                   ${CMAKE_SOURCE_DIR} "Quassel Client" ${CMAKE_BINARY_DIR}/quasselclient)
    endif()
    if (WANT_MONO)
        add_custom_command(TARGET quassel POST_BUILD
                           COMMAND ${CMAKE_SOURCE_DIR}/scripts/build/macosx_makebundle.py
                                   ${CMAKE_SOURCE_DIR} "Quassel" ${CMAKE_BINARY_DIR}/quassel)
    endif()

    if (DEPLOY)
        if (WANT_QTCLIENT)
            add_custom_command(TARGET quasselclient POST_BUILD WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
                               COMMAND ${CMAKE_SOURCE_DIR}/scripts/build/macosx_makePackage.sh Client ${CMAKE_BINARY_DIR} qsvgicon)
        endif()
        if (WANT_CORE)
            add_custom_command(TARGET quasselcore POST_BUILD WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
                               COMMAND ${CMAKE_SOURCE_DIR}/scripts/build/macosx_makePackage.sh Core ${CMAKE_BINARY_DIR})
        endif()
        if (WANT_MONO)
            add_custom_command(TARGET quassel POST_BUILD WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
                               COMMAND ${CMAKE_SOURCE_DIR}/scripts/build/macosx_makePackage.sh Mono ${CMAKE_BINARY_DIR} qsvgicon)
        endif()
    endif()
endif()
