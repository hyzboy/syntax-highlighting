add_subdirectory(indexer)
if(TARGET Qt6::Gui)
    add_subdirectory(lib)
    add_subdirectory(cli)
endif()
if(TARGET Qt6::Quick)
    add_subdirectory(quick)
endif()

# Note: ECM would generate and install ksyntaxhighlighting.categories file
# This is optional for logging category configuration
if(EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/ksyntaxhighlighting.categories)
    install(FILES ksyntaxhighlighting.categories
        DESTINATION ${CMAKE_INSTALL_DATADIR}/qlogging-categories6
    )
endif()
