
cmake_minimum_required( VERSION 3.16 )

project( BtxDocs )

set( DOCS_ROOT ${CMAKE_CURRENT_SOURCE_DIR} )
set( CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake ${CMAKE_MODULE_PATH} )

find_package( Doxygen REQUIRED )
find_package( Sphinx REQUIRED )

message( Current binary dir is ${CMAKE_CURRENT_BINARY_DIR} )

set( DOXYGEN_INPUT_DIR ${PROJECT_SOURCE_DIR}/btx-engine )
set( DOXYGEN_OUTPUT_DIR ${CMAKE_CURRENT_BINARY_DIR}/doxygen )
set( DOXYGEN_INDEX_FILE ${DOXYGEN_OUTPUT_DIR}/xml/index.xml )
set( DOXYFILE_IN ${DOCS_ROOT}/source/Doxyfile.in )
set( DOXYFILE_OUT ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile )

# Replace variables inside @@ with the current values
configure_file( ${DOXYFILE_IN} ${DOXYFILE_OUT} @ONLY )

# Doxygen won't create this for us
file( MAKE_DIRECTORY ${DOXYGEN_OUTPUT_DIR} )

# Get the public headers'n'stuff
# Defines BTX_COMMON_SOURCES
# Todo: maybe have a btx-public repository so we don't have to include the entire engine here
add_subdirectory( btx-engine )

# Only regenerate Doxygen when the Doxyfile or public headers change
add_custom_command( OUTPUT ${DOXYGEN_INDEX_FILE}
					DEPENDS ${BTX_COMMON_SOURCES}
					COMMAND ${DOXYGEN_EXECUTABLE} ${DOXYFILE_OUT}
					MAIN_DEPENDENCY ${DOXYFILE_OUT} ${DOXYFILE_IN}
					COMMENT "Generating docs"
					VERBATIM )

# Nice named target so we can run the job easily
add_custom_target( Doxygen ALL DEPENDS ${DOXYGEN_INDEX_FILE} )

set( SPHINX_SOURCE ${CMAKE_CURRENT_SOURCE_DIR}/source )
set( SPHINX_BUILD ${CMAKE_CURRENT_SOURCE_DIR}/docs )
set( SPHINX_INDEX_FILE ${SPHINX_BUILD}/index.html )

# Only regenerate Sphinx when:
# - Doxygen has rerun
# - Our doc files have been updated
# - The Sphinx config has been updated
add_custom_command( OUTPUT ${SPHINX_INDEX_FILE}
					COMMAND 
					${SPHINX_EXECUTABLE} -b html
					# Tell Breathe where to find the Doxygen output
					-Dbreathe_projects.BtxDocs=${DOXYGEN_OUTPUT_DIR}/xml
					${SPHINX_SOURCE} ${SPHINX_BUILD}
					WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
					DEPENDS
					# Other docs files you want to track should go here (or in some variable)
					${DOCS_ROOT}/source/index.rst
					${DOXYGEN_INDEX_FILE}
					MAIN_DEPENDENCY ${DOCS_ROOT}/source/conf.py
					COMMENT "Generating documentation with Sphinx" )

# Nice named target so we can run the job easily
add_custom_target( Sphinx ALL DEPENDS ${SPHINX_INDEX_FILE} )

# Add an install target to install the docs
include( GNUInstallDirs )
install( DIRECTORY ${SPHINX_BUILD}
	DESTINATION ${CMAKE_INSTALL_DOCDIR} )
