add_executable(ksyntaxhighlighter6 ksyntaxhighlighter.cpp)
if(WIN32)
    set_target_properties(ksyntaxhighlighter6 PROPERTIES WIN32_EXECUTABLE FALSE)
endif()
target_link_libraries(ksyntaxhighlighter6 KF6SyntaxHighlighting)

install(TARGETS ksyntaxhighlighter6
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    BUNDLE DESTINATION .
)
