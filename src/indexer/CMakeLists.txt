# when cross compiling, use either the executable offered or try to cross-compile it in place

if(CMAKE_CROSSCOMPILING AND KATEHIGHLIGHTINGINDEXER_EXECUTABLE)
    add_executable(katehighlightingindexer IMPORTED GLOBAL)
    set_target_properties(katehighlightingindexer PROPERTIES IMPORTED_LOCATION ${KATEHIGHLIGHTINGINDEXER_EXECUTABLE})
elseif(CMAKE_CROSSCOMPILING)
    # For cross-compilation, try to get the host Qt prefix
    get_target_property(QT_QMAKE_EXECUTABLE Qt6::qmake IMPORTED_LOCATION)
    if(QT_QMAKE_EXECUTABLE)
        execute_process(
            COMMAND ${QT_QMAKE_EXECUTABLE} -query QT_HOST_PREFIX
            OUTPUT_VARIABLE NATIVE_PREFIX
            OUTPUT_STRIP_TRAILING_WHITESPACE
        )
    endif()
    if(NOT NATIVE_PREFIX)
        set(NATIVE_PREFIX "/usr")
    endif()
    message(STATUS "Building katehighlightingindexer against ${NATIVE_PREFIX}")

    include(ExternalProject)
    ExternalProject_Add(native_katehighlightingindexer
                        SOURCE_DIR ${CMAKE_SOURCE_DIR}
                        CMAKE_ARGS -DKSYNTAXHIGHLIGHTING_USE_GUI=OFF
                            -DCMAKE_PREFIX_PATH=${NATIVE_PREFIX}
                            -DCMAKE_INSTALL_PREFIX=${CMAKE_CURRENT_BINARY_DIR}
                            -DQT_MAJOR_VERSION=6
                        INSTALL_COMMAND ""
                        BUILD_BYPRODUCTS ${CMAKE_CURRENT_BINARY_DIR}/native_katehighlightingindexer-prefix/src/native_katehighlightingindexer-build/bin/katehighlightingindexer
                    )
    add_executable(katehighlightingindexer IMPORTED GLOBAL)
    add_dependencies(katehighlightingindexer native_katehighlightingindexer)
    set_target_properties(katehighlightingindexer PROPERTIES IMPORTED_LOCATION
        ${CMAKE_CURRENT_BINARY_DIR}/native_katehighlightingindexer-prefix/src/native_katehighlightingindexer-build/bin/katehighlightingindexer)
else()
    # host build
    add_executable(katehighlightingindexer katehighlightingindexer.cpp ../lib/worddelimiters.cpp)
    if(WIN32)
        set_target_properties(katehighlightingindexer PROPERTIES WIN32_EXECUTABLE FALSE)
    endif()
    if(XercesC_FOUND)
        add_definitions(-DHAS_XERCESC)
        target_compile_options(katehighlightingindexer PRIVATE
            $<$<CXX_COMPILER_ID:MSVC>:/EHsc>
            $<$<NOT:$<CXX_COMPILER_ID:MSVC>>:-fexceptions>
        )
        target_link_libraries(katehighlightingindexer Qt6::Core XercesC::XercesC)
    else()
        target_link_libraries(katehighlightingindexer Qt6::Core)
    endif()
endif()
