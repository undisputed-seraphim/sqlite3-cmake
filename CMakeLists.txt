cmake_minimum_required(VERSION 3.8)
project(sqlite3 VERSION 3.29.0)

if(NOT WIN32)
	find_package(Threads REQUIRED)
endif()

# SQLite built-in extensions
option(BUILD_ENABLE_DBSTAT_VTAB "Enables dbstat virtual table"                  ON)
option(BUILD_ENABLE_FTS3        "Enables full text searches version 3"         OFF)
option(BUILD_ENABLE_FTS4        "Enables full text searches version 3 & 4"     OFF)
option(BUILD_ENABLE_FTS5        "Enables full text searches version 5"          ON)
option(BUILD_ENABLE_GEOPOLY     "Enables Geopoly extention"                    OFF)
option(BUILD_ENABLE_ICU         "Enables international components for unicode" OFF)
option(BUILD_ENABLE_JSON1       "Enables JSON SQL functions"                    ON)
option(BUILD_ENABLE_RBU         "Enables resumable bulk update extension"      OFF)
option(BUILD_ENABLE_RTREE       "Enables R*TRee index extension"                ON)

option(MAX_COLUMN "Maximum number of columns (0 to default)" 0)
option(THREADSAFE "Thread safety"               ON)

include_directories(${CMAKE_SOURCE_DIR}/src)
add_library(sqlite3 STATIC
	src/sqlite3.c
	src/sqlite3.h
	src/sqlite3ext.h
)
target_compile_definitions(sqlite3 PUBLIC
	$<$<BOOL:${BUILD_ENABLE_DBSTAT_VTAB}>:SQLITE_ENABLE_DBSTAT_VTAB>
	$<$<BOOL:${BUILD_ENABLE_FTS3}>:SQLITE_ENABLE_FTS3>
	$<$<BOOL:${BUILD_ENABLE_FTS4}>:SQLITE_ENABLE_FTS4>
	$<$<BOOL:${BUILD_ENABLE_FTS5}>:SQLITE_ENABLE_FTS5>
	$<$<BOOL:${BUILD_ENABLE_GEOPOLY}>:SQLITE_ENABLE_GEOPOLY>
	$<$<BOOL:${BUILD_ENABLE_ICU}>:SQLITE_ENABLE_ICU>
	$<$<BOOL:${BUILD_ENABLE_JSON1}>:SQLITE_ENABLE_JSON1>
	$<$<BOOL:${BUILD_ENABLE_RBU}>:SQLITE_ENABLE_RBU>
	$<$<BOOL:${BUILD_ENABLE_RTREE}>:SQLITE_ENABLE_RTREE>
	
	$<$<BOOL:${MAX_COLUMN}>:SQLITE_MAX_COLUMN=${MAX_COLUMN}>
	$<$<BOOL:${THREADSAFE}>:SQLITE_THREADSAFE>
)
set_target_properties(sqlite3 PROPERTIES OUTPUT_NAME_DEBUG sqlite3d)

add_executable(sqlite src/shell.c)
target_link_libraries(sqlite PUBLIC sqlite3	
	$<$<NOT:$<BOOL:${WIN32}>>:dl>
	$<$<NOT:$<BOOL:${WIN32}>>:m>
	$<$<NOT:$<BOOL:${WIN32}>>:Threads::Threads>
)
set_target_properties(sqlite PROPERTIES OUTPUT_NAME sqlite3)

# Uncomment this for single-threaded variant (faster)
#add_definitions(-DSQLITE_THREADSAFE=0)

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

install(TARGETS sqlite RUNTIME DESTINATION bin)
install(FILES src/sqlite3.h src/sqlite3ext.h DESTINATION include)
install(TARGETS sqlite3 EXPORT sqlite-targets LIBRARY DESTINATION lib ARCHIVE DESTINATION lib)
install(EXPORT sqlite-targets DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/sqlite NAMESPACE sqlite::)

configure_package_config_file(
	${CMAKE_CURRENT_SOURCE_DIR}/sqlite-config.cmake.in
	${CMAKE_CURRENT_BINARY_DIR}/sqlite-config.cmake
	INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/sqlite
)

write_basic_package_version_file(
	${CMAKE_CURRENT_BINARY_DIR}/sqlite-version.cmake
	COMPATIBILITY AnyNewerVersion
)
install(
	FILES
		${CMAKE_CURRENT_BINARY_DIR}/sqlite-config.cmake
		${CMAKE_CURRENT_BINARY_DIR}/sqlite-version.cmake
	DESTINATION
		${CMAKE_INSTALL_LIBDIR}/cmake/sqlite
)