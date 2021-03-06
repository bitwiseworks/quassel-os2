quassel_add_module(Client EXPORT)

target_sources(${TARGET} PRIVATE
    abstractmessageprocessor.cpp
    backlogrequester.cpp
    backlogsettings.cpp
    buffermodel.cpp
    buffersettings.cpp
    bufferviewoverlay.cpp
    client.cpp
    clientaliasmanager.cpp
    clientauthhandler.cpp
    clientbacklogmanager.cpp
    clientbufferviewconfig.cpp
    clientbufferviewmanager.cpp
    clientidentity.cpp
    clientignorelistmanager.cpp
    clientirclisthelper.cpp
    clientsettings.cpp
    clienttransfer.cpp
    clienttransfermanager.cpp
    clientuserinputhandler.cpp
    coreaccount.cpp
    coreaccountmodel.cpp
    coreconnection.cpp
    execwrapper.cpp
    irclistmodel.cpp
    messagefilter.cpp
    messagemodel.cpp
    networkmodel.cpp
    selectionmodelsynchronizer.cpp
    transfermodel.cpp
    treemodel.cpp

    # needed for automoc
    abstractui.h
)

target_link_libraries(${TARGET}
    PUBLIC
        Qt5::Core
        Qt5::Gui
        Qt5::Network
        Qt5::Widgets  # QAbstractItemView in BufferModel
        Quassel::Common
)

if(OS2)
    set_target_properties(${TARGET} PROPERTIES
        OS2_DEF_VENDOR "$ENV{VENDOR}"
        OS2_DEF_PATCH "${QUASSEL_PATCH}"
        OS2_DEF_VERSION "${QUASSEL_VERSION_STRING}"
        TARGET_SHORT quasclnt)
endif()

if (EMBED_DATA)
    set_property(SOURCE client.cpp APPEND PROPERTY COMPILE_DEFINITIONS EMBED_DATA)
endif()

target_link_if_exists(${TARGET} PUBLIC Quassel::Resource::Data)
