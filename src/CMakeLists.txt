add_subdirectory(indexer)
if(TARGET Qt6::Gui)
    add_subdirectory(lib)
    add_subdirectory(cli)
endif()
if(TARGET Qt6::Quick)
    add_subdirectory(quick)
endif()

# Optional: Install logging categories file
install(FILES ksyntaxhighlighting.categories
    DESTINATION ${CMAKE_INSTALL_DATADIR}/qlogging-categories6
)
