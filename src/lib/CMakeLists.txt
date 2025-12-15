add_library(KF6SyntaxHighlighting)

set_target_properties(KF6SyntaxHighlighting PROPERTIES
    VERSION     ${KSYNTAXHIGHLIGHTING_VERSION}
    SOVERSION   ${KSYNTAXHIGHLIGHTING_SOVERSION}
    EXPORT_NAME SyntaxHighlighting
)

target_sources(KF6SyntaxHighlighting PRIVATE
    abstracthighlighter.cpp
    context.cpp
    contextswitch.cpp
    definitiondownloader.cpp
    highlightingdata.cpp
    foldingregion.cpp
    format.cpp
    htmlhighlighter.cpp
    ansihighlighter.cpp
    keywordlist.cpp
    rule.cpp
    definition.cpp
    repository.cpp
    state.cpp
    syntaxhighlighter.cpp
    theme.cpp
    wildcardmatcher.cpp
    themedata.cpp
    worddelimiters.cpp
    $<TARGET_OBJECTS:SyntaxHighlightingData>
)

# Generate logging category header
configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/ksyntaxhighlighting_logging.h.in"
    "${CMAKE_CURRENT_BINARY_DIR}/ksyntaxhighlighting_logging.h"
    @ONLY
)

target_sources(KF6SyntaxHighlighting PRIVATE
    "${CMAKE_CURRENT_BINARY_DIR}/ksyntaxhighlighting_logging.cpp"
)

generate_export_header(KF6SyntaxHighlighting
    BASE_NAME KSyntaxHighlighting
    EXPORT_FILE_NAME ksyntaxhighlighting_export.h
)

qt_extract_metatypes(KF6SyntaxHighlighting)
target_link_libraries(KF6SyntaxHighlighting
    PUBLIC
        Qt6::Gui
    PRIVATE
        Qt6::Network
)

set(SyntaxHighlighting_HEADERS
    abstracthighlighter.h
    definition.h
    definitiondownloader.h
    foldingregion.h
    format.h
    repository.h
    state.h
    syntaxhighlighter.h
    theme.h
    wildcardmatcher.h
)

# Create CamelCase forwarding headers
set(CamelCase_HEADERS)
foreach(header AbstractHighlighter Definition DefinitionDownloader FoldingRegion Format Repository State SyntaxHighlighter Theme WildcardMatcher)
    string(TOLOWER ${header} header_lower)
    set(forwarding_header "${CMAKE_CURRENT_BINARY_DIR}/KSyntaxHighlighting/${header}")
    file(WRITE ${forwarding_header} "#include \"${header_lower}.h\"\n")
    list(APPEND CamelCase_HEADERS ${forwarding_header})
endforeach()

target_include_directories(KF6SyntaxHighlighting
    INTERFACE "$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/KSyntaxHighlighting>"
)

install(TARGETS KF6SyntaxHighlighting 
    EXPORT KF6SyntaxHighlightingTargets
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/KSyntaxHighlighting
)

install(FILES
    ${CamelCase_HEADERS}
    ${SyntaxHighlighting_HEADERS}
    ${CMAKE_CURRENT_BINARY_DIR}/ksyntaxhighlighting_export.h
    ${CMAKE_CURRENT_BINARY_DIR}/ksyntaxhighlighting_logging.h
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/KSyntaxHighlighting/KSyntaxHighlighting
)

# Optional: QDoc generation (requires qdoc tool)
# add_custom_target(qdoc COMMAND qdoc ksyntaxhighlighting.qdocconf)
