cmake_minimum_required(VERSION 3.10)

if(CMAKE_CURRENT_SOURCE_DIR STREQUAL CMAKE_SOURCE_DIR)
    project(InsightCAEDocumentation NONE)
endif()

find_program(PANDOC pandoc)

if(PANDOC)
    set(DOC_BASE ${CMAKE_CURRENT_SOURCE_DIR}/docs)

    set(DOC_SOURCES
        ${DOC_BASE}/introduction.md
        ${DOC_BASE}/installation/linux.md
        ${DOC_BASE}/installation/windows.md
        ${DOC_BASE}/installation/from-source.md
        ${DOC_BASE}/workflow_automation/insightcae.md
        ${DOC_BASE}/extensions_to_tools/insightcae-extensions.md
        ${DOC_BASE}/cad/insightcad.md
        ${DOC_BASE}/tutorials/insightcae-tutorials.md
        ${DOC_BASE}/api/insightcae-api.md
    )

    set(DOC_RESOURCE_PATHS
        ${DOC_BASE}
        ${DOC_BASE}/installation
        ${DOC_BASE}/workflow_automation
        ${DOC_BASE}/cad
	${DOC_BASE}/extensions_to_tools
	${DOC_BASE}/api
    )
    list(JOIN DOC_RESOURCE_PATHS ":" DOC_RESOURCE_PATH_ARG)

    add_custom_command(
        OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/insightcae-manual.pdf
        COMMAND ${PANDOC}
            ${DOC_SOURCES}
            --toc
            --pdf-engine=xelatex
            --resource-path=${DOC_RESOURCE_PATH_ARG}
            -o ${CMAKE_CURRENT_BINARY_DIR}/insightcae-manual.pdf
        DEPENDS ${DOC_SOURCES}
        COMMENT "Building InsightCAE manual PDF with pandoc"
        VERBATIM
    )

    add_custom_target(insightcae-manual
        DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/insightcae-manual.pdf
    )
else()
    message(STATUS "pandoc not found -- PDF manual target will not be available")
endif()
