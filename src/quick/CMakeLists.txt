# SPDX-FileCopyrightText: 2018 Eike Hein <hein@kde.org>
# SPDX-FileCopyrightText: 2021 Volker Krause <vkrause@kde.org>
# SPDX-License-Identifier: BSD-3-Clause

qt_add_qml_module(kquicksyntaxhighlightingplugin
    URI "org.kde.syntaxhighlighting"
    VERSION 1.0
    PLUGIN_TARGET kquicksyntaxhighlightingplugin
    OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/qml/org/kde/syntaxhighlighting
)

target_sources(kquicksyntaxhighlightingplugin PRIVATE
    kquicksyntaxhighlighter.cpp
    types.cpp
)
target_link_libraries(kquicksyntaxhighlightingplugin PRIVATE
    KF6SyntaxHighlighting
    Qt6::Quick
)

install(TARGETS kquicksyntaxhighlightingplugin
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}/qml/org/kde/syntaxhighlighting
    RUNTIME DESTINATION ${CMAKE_INSTALL_LIBDIR}/qml/org/kde/syntaxhighlighting
)

# Optional: QDoc generation (requires qdoc tool)
# add_custom_target(qdoc_qml COMMAND qdoc ksyntaxhighlightingqml.qdocconf)
