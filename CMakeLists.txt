CMAKE_MINIMUM_REQUIRED(VERSION 2.8.7)

PROJECT(lmms)

SET(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake/modules" ${CMAKE_MODULE_PATH})

IF(COMMAND CMAKE_POLICY)
	CMAKE_POLICY(SET CMP0005 NEW)
	CMAKE_POLICY(SET CMP0003 NEW)
ENDIF(COMMAND CMAKE_POLICY)

INCLUDE(AddFileDependencies)
INCLUDE(CheckIncludeFiles)
INCLUDE(FindPkgConfig)

SET(VERSION_MAJOR "1")
SET(VERSION_MINOR "1")
SET(VERSION_PATCH "3")
#SET(VERSION_SUFFIX "")
SET(VERSION "${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH}")
IF(VERSION_SUFFIX)
	SET(VERSION "${VERSION}-${VERSION_SUFFIX}")
ENDIF(VERSION_SUFFIX)

INCLUDE(DetectMachine)


OPTION(WANT_ALSA	"Include ALSA (Advanced Linux Sound Architecture) support" ON)
OPTION(WANT_CALF	"Include CALF LADSPA plugins" ON)
OPTION(WANT_CAPS	"Include C* Audio Plugin Suite (LADSPA plugins)" ON)
OPTION(WANT_CARLA	"Include Carla plugin" ON)
OPTION(WANT_CMT		"Include Computer Music Toolkit LADSPA plugins" ON)
OPTION(WANT_JACK	"Include JACK (Jack Audio Connection Kit) support" ON)
OPTION(WANT_OGGVORBIS	"Include OGG/Vorbis support" ON)
OPTION(WANT_PULSEAUDIO	"Include PulseAudio support" ON)
OPTION(WANT_PORTAUDIO	"Include PortAudio support" ON)
OPTION(WANT_SDL		"Include SDL (Simple DirectMedia Layer) support" ON)
OPTION(WANT_SF2		"Include SoundFont2 player plugin" ON)
OPTION(WANT_GIG		"Include GIG player plugin" ON)
OPTION(WANT_STK		"Include Stk (Synthesis Toolkit) support" ON)
OPTION(WANT_SWH		"Include Steve Harris's LADSPA plugins" ON)
OPTION(WANT_TAP		"Include Tom's Audio Processing LADSPA plugins" ON)
OPTION(WANT_VST		"Include VST support" ON)
OPTION(WANT_VST_NOWINE	"Include partial VST support (without wine)" OFF)
OPTION(WANT_WINMM	"Include WinMM MIDI support" OFF)
OPTION(WANT_QT5		"Build with Qt5" OFF)


IF(LMMS_BUILD_APPLE)
	SET(WANT_SF2 OFF)
	SET(WANT_ALSA OFF)
	SET(WANT_PULSEAUDIO OFF)
	SET(WANT_VST OFF)
	SET(STATUS_ALSA "<not supported on this platform>")
	SET(STATUS_PULSEAUDIO "<not supported on this platform>")
	# MacPorts: /opt/local/lib
	LINK_DIRECTORIES(${LINK_DIRECTORIES} /opt/local/lib)
ENDIF(LMMS_BUILD_APPLE)


IF(LMMS_BUILD_WIN32)
	SET(WANT_ALSA OFF)
	SET(WANT_JACK OFF)
	SET(WANT_PULSEAUDIO OFF)
	SET(WANT_SYSTEM_SR OFF)
	SET(WANT_WINMM ON)
	SET(LMMS_HAVE_WINMM TRUE)
	SET(STATUS_ALSA "<not supported on this platform>")
	SET(STATUS_JACK "<not supported on this platform>")
	SET(STATUS_PULSEAUDIO "<not supported on this platform>")
	SET(STATUS_WINMM "OK")
ELSE(LMMS_BUILD_WIN32)
	SET(STATUS_WINMM "<not supported on this platform>")
ENDIF(LMMS_BUILD_WIN32)



CHECK_INCLUDE_FILES(stdint.h LMMS_HAVE_STDINT_H)
CHECK_INCLUDE_FILES(stdlib.h LMMS_HAVE_STDLIB_H)
CHECK_INCLUDE_FILES(pthread.h LMMS_HAVE_PTHREAD_H)
CHECK_INCLUDE_FILES(semaphore.h LMMS_HAVE_SEMAPHORE_H)
CHECK_INCLUDE_FILES(unistd.h LMMS_HAVE_UNISTD_H)
CHECK_INCLUDE_FILES(sys/types.h LMMS_HAVE_SYS_TYPES_H)
CHECK_INCLUDE_FILES(sys/ipc.h LMMS_HAVE_SYS_IPC_H)
CHECK_INCLUDE_FILES(sys/shm.h LMMS_HAVE_SYS_SHM_H)
CHECK_INCLUDE_FILES(sys/time.h LMMS_HAVE_SYS_TIME_H)
CHECK_INCLUDE_FILES(sys/wait.h LMMS_HAVE_SYS_WAIT_H)
CHECK_INCLUDE_FILES(sys/select.h LMMS_HAVE_SYS_SELECT_H)
CHECK_INCLUDE_FILES(stdarg.h LMMS_HAVE_STDARG_H)
CHECK_INCLUDE_FILES(signal.h LMMS_HAVE_SIGNAL_H)
CHECK_INCLUDE_FILES(sched.h LMMS_HAVE_SCHED_H)
CHECK_INCLUDE_FILES(sys/soundcard.h LMMS_HAVE_SYS_SOUNDCARD_H)
CHECK_INCLUDE_FILES(soundcard.h LMMS_HAVE_SOUNDCARD_H)
CHECK_INCLUDE_FILES(fcntl.h LMMS_HAVE_FCNTL_H)
CHECK_INCLUDE_FILES(sys/ioctl.h LMMS_HAVE_SYS_IOCTL_H)
CHECK_INCLUDE_FILES(ctype.h LMMS_HAVE_CTYPE_H)
CHECK_INCLUDE_FILES(string.h LMMS_HAVE_STRING_H)
CHECK_INCLUDE_FILES(process.h LMMS_HAVE_PROCESS_H)
CHECK_INCLUDE_FILES(locale.h LMMS_HAVE_LOCALE_H)

LIST(APPEND CMAKE_PREFIX_PATH "${CMAKE_INSTALL_PREFIX}")

IF(WANT_QT5)
	SET(QT5 TRUE)

	FIND_PACKAGE(Qt5Core REQUIRED)
	FIND_PACKAGE(Qt5Gui REQUIRED)
	FIND_PACKAGE(Qt5Widgets REQUIRED)
	FIND_PACKAGE(Qt5Xml REQUIRED)


ELSE()
	SET(QT5 FALSE)

	# check for Qt4
	SET(QT_MIN_VERSION "4.6.0" COMPONENTS QtCore QtGui QtXml)
	FIND_PACKAGE(Qt4 REQUIRED)
	SET(QT_USE_QTXML 1)
	IF(WIN32)
		SET(QT_TRANSLATIONS_DIR "${MINGW_PREFIX}/share/qt4/translations/")
	ENDIF(WIN32)
	IF(EXISTS "${QT_TRANSLATIONS_DIR}")
		MESSAGE("-- Found Qt translations in ${QT_TRANSLATIONS_DIR}")
		ADD_DEFINITIONS(-D'QT_TRANSLATIONS_DIR="${QT_TRANSLATIONS_DIR}"')
	ENDIF()
	IF(NOT WIN32)
		STRING(REPLACE "-DQT_DLL" "" QT_DEFINITIONS "${QT_DEFINITIONS}")
	ENDIF()
	INCLUDE("${QT_USE_FILE}")
ENDIF()

# check for libsndfile
PKG_CHECK_MODULES(SNDFILE REQUIRED sndfile>=1.0.11)
IF(NOT SNDFILE_FOUND)
	MESSAGE(FATAL_ERROR "LMMS requires libsndfile1 and libsndfile1-dev >= 1.0.11 - please install, remove CMakeCache.txt and try again!")
ENDIF(NOT SNDFILE_FOUND)

IF(WANT_CALF)
	SET(LMMS_HAVE_CALF TRUE)
	SET(STATUS_CALF "OK")
ELSE(WANT_CALF)
	SET(STATUS_CALF "not built as requested")
ENDIF(WANT_CALF)

IF(WANT_CAPS)
	SET(LMMS_HAVE_CAPS TRUE)
	SET(STATUS_CAPS "OK")
ELSE(WANT_CAPS)
	SET(STATUS_CAPS "not built as requested")
ENDIF(WANT_CAPS)

IF(WANT_CMT)
	SET(LMMS_HAVE_CMT TRUE)
	SET(STATUS_CMT "OK")
ELSE(WANT_CMT)
	SET(STATUS_CMT "not built as requested")
ENDIF(WANT_CMT)

IF(WANT_SWH)
	SET(LMMS_HAVE_SWH TRUE)
	SET(STATUS_SWH "OK")
ELSE(WANT_SWH)
	SET(STATUS_SWH "not built as requested")
ENDIF(WANT_SWH)

IF(WANT_TAP)
	SET(LMMS_HAVE_TAP TRUE)
	SET(STATUS_TAP "OK")
ELSE(WANT_TAP)
	SET(STATUS_TAP "not built as requested")
ENDIF(WANT_TAP)


# check for CARLA
IF(WANT_CARLA)
	PKG_CHECK_MODULES(CARLA carla-standalone>=1.9.5)
	IF(CARLA_FOUND)
		SET(LMMS_HAVE_CARLA TRUE)
		SET(STATUS_CARLA "OK")
	ELSE(CARLA_FOUND)
		SET(STATUS_CARLA "not found, please install the latest carla")
	ENDIF(CARLA_FOUND)
ENDIF(WANT_CARLA)


# check for SDL
IF(WANT_SDL)
	SET(SDL_BUILDING_LIBRARY TRUE)
	FIND_PACKAGE(SDL)
	IF(SDL_FOUND)
		SET(LMMS_HAVE_SDL TRUE)
		SET(STATUS_SDL "OK")
	ELSE(SDL_FOUND)
		SET(STATUS_SDL "not found, please install libsdl1.2-dev (or similar) "
			"if you require SDL support")
	ENDIF(SDL_FOUND)
ENDIF(WANT_SDL)
IF(NOT LMMS_HAVE_SDL)
	SET(SDL_INCLUDE_DIR "")
ELSE(NOT LMMS_HAVE_SDL)
	IF(NOT SDL_INCLUDE_DIR)
		SET(SDL_INCLUDE_DIR "${CMAKE_FIND_ROOT_PATH}/include")
	ENDIF(NOT SDL_INCLUDE_DIR)
ENDIF(NOT LMMS_HAVE_SDL)


# check for Stk
IF(WANT_STK)
	FIND_PACKAGE(STK)
	IF(STK_FOUND)
		SET(LMMS_HAVE_STK TRUE)
		SET(STATUS_STK "OK")
	ELSE(STK_FOUND)
		SET(STK_INCLUDE_DIR "")
		SET(STATUS_STK "not found, please install libstk0-dev (or similar) "
			"if you require the Mallets instrument")
	ENDIF(STK_FOUND)
ENDIF(WANT_STK)


# check for PortAudio
IF(WANT_PORTAUDIO)
	FIND_PACKAGE(Portaudio)
	IF(PORTAUDIO_FOUND)
		SET(LMMS_HAVE_PORTAUDIO TRUE)
		SET(STATUS_PORTAUDIO "OK")
	ELSE(PORTAUDIO_FOUND)
		SET(STATUS_PORTAUDIO "not found, please install portaudio19-dev (or similar, version >= 1.9) "
			"if you require PortAudio support")
	ENDIF(PORTAUDIO_FOUND)
ENDIF(WANT_PORTAUDIO)


# check for PulseAudio
IF(WANT_PULSEAUDIO)
	FIND_PACKAGE(PulseAudio)
	IF(PULSEAUDIO_FOUND)
		SET(LMMS_HAVE_PULSEAUDIO TRUE)
		SET(STATUS_PULSEAUDIO "OK")
	ELSE(PULSEAUDIO_FOUND)
		SET(STATUS_PULSEAUDIO "not found, please install libpulse-dev (or similar) "
			"if you require PulseAudio support")
	ENDIF(PULSEAUDIO_FOUND)
ENDIF(WANT_PULSEAUDIO)
IF(NOT LMMS_HAVE_PULSEAUDIO)
	SET(PULSEAUDIO_INCLUDE_DIR "")
	SET(PULSEAUDIO_LIBRARIES "")
ENDIF(NOT LMMS_HAVE_PULSEAUDIO)


# check for OGG/Vorbis-libraries
IF(WANT_OGGVORBIS)
	FIND_PACKAGE(OggVorbis)
	IF(OGGVORBIS_FOUND)
		SET(LMMS_HAVE_OGGVORBIS TRUE)
		SET(STATUS_OGGVORBIS "OK")
	ELSE(OGGVORBIS_FOUND)
		SET(STATUS_OGGVORBIS "not found, libogg-dev and libvorbis-dev (or similar) "
			"is highly recommended")
	ENDIF(OGGVORBIS_FOUND)
ENDIF(WANT_OGGVORBIS)


# check whether to enable OSS-support
IF(LMMS_HAVE_SOUNDCARD_H OR LMMS_HAVE_SYS_SOUNDCARD_H)
	SET(LMMS_HAVE_OSS TRUE)
	SET(STATUS_OSS "OK")
ELSE(LMMS_HAVE_SOUNDCARD_H OR LMMS_HAVE_SYS_SOUNDCARD_H)
	SET(STATUS_OSS "<not found or not supported on this platform>")
ENDIF(LMMS_HAVE_SOUNDCARD_H OR LMMS_HAVE_SYS_SOUNDCARD_H)


# check for ALSA
IF(WANT_ALSA)
	FIND_PACKAGE(Alsa)
	IF(ALSA_FOUND)
		SET(LMMS_HAVE_ALSA TRUE)
		SET(STATUS_ALSA "OK")
	ELSE(ALSA_FOUND)
		SET(STATUS_ALSA "not found, please install libasound2-dev (or similar) "
			"if you require ALSA support")
	ENDIF(ALSA_FOUND)
ENDIF(WANT_ALSA)
IF(NOT LMMS_HAVE_ALSA)
	SET(ASOUND_LIBRARY "")
ENDIF(NOT LMMS_HAVE_ALSA)


# check for JACK
IF(WANT_JACK)
	PKG_CHECK_MODULES(JACK jack>=0.77)
	IF(JACK_FOUND)
		SET(LMMS_HAVE_JACK TRUE)
		SET(STATUS_JACK "OK")
	ELSE(JACK_FOUND)
		SET(STATUS_JACK "not found, please install libjack0.100.0-dev (or similar) "
			"if you require JACK support")
	ENDIF(JACK_FOUND)
ENDIF(WANT_JACK)


# check for FFTW3F-library
PKG_CHECK_MODULES(FFTW3F REQUIRED fftw3f>=3.0.0)


# check for Fluidsynth
IF(WANT_SF2)
	PKG_CHECK_MODULES(FLUIDSYNTH fluidsynth>=1.0.7)
	IF(FLUIDSYNTH_FOUND)
		SET(LMMS_HAVE_FLUIDSYNTH TRUE)
		SET(STATUS_FLUIDSYNTH "OK")
	ELSE(FLUIDSYNTH_FOUND)
		SET(STATUS_FLUIDSYNTH "not found, libfluidsynth-dev (or similar)"
			"is highly recommended")
	ENDIF(FLUIDSYNTH_FOUND)
ENDIF(WANT_SF2)

# check for libgig
If(WANT_GIG)
	PKG_CHECK_MODULES(GIG gig)
	IF(GIG_FOUND)
		SET(LMMS_HAVE_GIG TRUE)
		SET(STATUS_GIG "OK")
	ELSE(GIG_FOUND)
		SET(STATUS_GIG "not found, libgig needed for decoding .gig files")
	ENDIF(GIG_FOUND)
ENDIF(WANT_GIG)

# check for pthreads
IF(LMMS_BUILD_LINUX OR LMMS_BUILD_APPLE)
	FIND_PACKAGE(Threads)
ENDIF(LMMS_BUILD_LINUX OR LMMS_BUILD_APPLE)

# check for WINE
IF(WANT_VST)
	FIND_PACKAGE(Wine)
	IF(WINE_FOUND)
		SET(LMMS_SUPPORT_VST TRUE)
		SET(STATUS_VST "OK")
	ELSE(WINE_FOUND)
		SET(STATUS_VST "not found, please install (lib)wine-dev (or similar) - 64 bit systems additionally need gcc-multilib and g++-multilib")
	ENDIF(WINE_FOUND)
ENDIF(WANT_VST)
IF(LMMS_BUILD_WIN32)
	SET(LMMS_SUPPORT_VST TRUE)
	SET(STATUS_VST "OK")
ENDIF(LMMS_BUILD_WIN32)


# check for libsamplerate
PKG_CHECK_MODULES(SAMPLERATE REQUIRED samplerate>=0.1.8)


CONFIGURE_FILE("${CMAKE_SOURCE_DIR}/lmmsconfig.h.in" "${CMAKE_BINARY_DIR}/lmmsconfig.h")
CONFIGURE_FILE("${CMAKE_SOURCE_DIR}/lmmsversion.h.in" "${CMAKE_BINARY_DIR}/lmmsversion.h")
CONFIGURE_FILE("${CMAKE_SOURCE_DIR}/lmms.rc.in" "${CMAKE_BINARY_DIR}/lmms.rc")
CONFIGURE_FILE("${CMAKE_SOURCE_DIR}/plugins/zynaddsubfx/zynaddsubfx.rc.in" "${CMAKE_BINARY_DIR}/plugins/zynaddsubfx/zynaddsubfx.rc")

# set compiler flags
SET(WERROR_FLAGS "-Wall -Werror=unused-function -Wno-sign-compare -Wno-strict-overflow")
OPTION(USE_WERROR "Add -werror to the build flags. Stops the build on warnings" OFF)
IF(${USE_WERROR})
   SET(WERROR_FLAGS "${WERROR_FLAGS} -Werror")
ENDIF()

# Due to a regression in gcc-4.8.X, we need to disable array-bounds check
IF (CMAKE_COMPILER_IS_GNUCXX AND ((CMAKE_CXX_COMPILER_VERSION VERSION_EQUAL "4.8.0") OR (CMAKE_CXX_COMPILER_VERSION VERSION_GREATER "4.8.0") OR LMMS_BUILD_WIN32))
   SET(WERROR_FLAGS "${WERROR_FLAGS} -Wno-array-bounds")
ENDIF()

IF(NOT CMAKE_BUILD_TYPE)
	message(STATUS "Setting build type to 'Release' as none was specified.")
	set(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build." FORCE)
	# Set the possible values of build type for cmake-gui
	SET_PROPERTY(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release"
  		"MinSizeRel" "RelWithDebInfo")
ENDIF()

SET(CMAKE_C_FLAGS "${WERROR_FLAGS} ${CMAKE_C_FLAGS}")
SET(CMAKE_CXX_FLAGS "-fno-exceptions ${WERROR_FLAGS} ${CMAKE_CXX_FLAGS}")
SET(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -DLMMS_DEBUG")
SET(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -DLMMS_DEBUG")


# people simply updating git will still have this and mess up build with it
FILE(REMOVE include/lmmsconfig.h)

FILE(GLOB LMMS_INCLUDES "${CMAKE_SOURCE_DIR}/include/*.h")

# Get list of all committers from git history, ordered by number of commits
FIND_PACKAGE(Git)
IF(GIT_FOUND)
	SET(CONTRIBUTORS "${CMAKE_BINARY_DIR}/CONTRIBUTORS")
	EXECUTE_PROCESS(
		COMMAND "${GIT_EXECUTABLE}" shortlog -sne
		COMMAND cut -c8-
		OUTPUT_FILE "${CONTRIBUTORS}"
		WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}"
		TIMEOUT 1)
ENDIF(GIT_FOUND)

# embedded resources stuff
IF(WIN32 OR WIN64)
	# compile buildtools native
	SET(BIN2RES_CPP "${CMAKE_SOURCE_DIR}/buildtools/bin2res.cpp")
	SET(BIN2RES "${CMAKE_BINARY_DIR}/bin2res")
	ADD_CUSTOM_TARGET(bin2res COMMAND g++ "\"${BIN2RES_CPP}\"" -o "\"${BIN2RES}\"" DEPENDS "${BIN2RES_CPP}")
ELSE(WIN32 OR WIN64)
	ADD_EXECUTABLE(bin2res buildtools/bin2res.cpp)
	GET_TARGET_PROPERTY(BIN2RES bin2res LOCATION)
ENDIF(WIN32 OR WIN64)

# we somehow have to make LMMS-binary depend on MOC-files
ADD_FILE_DEPENDENCIES("${CMAKE_BINARY_DIR}/lmmsconfig.h")

# set up apple vars before traversing into data/scripts
SET(MACOSX_BUNDLE_ICON_FILE "lmms.icns")
SET(MACOSX_BUNDLE_GUI_IDENTIFIER "LMMS")
SET(MACOSX_BUNDLE_LONG_VERSION_STRING "${VERSION}")
SET(MACOSX_BUNDLE_BUNDLE_NAME "LMMS")
SET(MACOSX_BUNDLE_SHORT_VERSION_STRING "${VERSION}")
SET(MACOSX_BUNDLE_BUNDLE_VERSION "${VERSION}")
SET(MACOSX_BUNDLE_COPYRIGHT "Tobias Doerffel, 2008-2010")
SET(MACOSX_BUNDLE_MIMETYPE "application/x-lmms-project")
SET(MACOSX_BUNDLE_MIMETYPE_ICON "project.icns")
SET(MACOSX_BUNDLE_MIMETYPE_ID "net.sourceforge.lmms")
SET(MACOSX_BUNDLE_PROJECT_URL "http://lmms.io")

IF(WIN32)
	SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-attributes")
ELSE(WIN32)
	SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC -DPIC")
ENDIF(WIN32)

# make sub-directories
ADD_SUBDIRECTORY(src)
ADD_SUBDIRECTORY(plugins)
ADD_SUBDIRECTORY(tests)
ADD_SUBDIRECTORY(data)
ADD_SUBDIRECTORY(doc)

ADD_CUSTOM_COMMAND(OUTPUT "${CMAKE_BINARY_DIR}/lmms.1.gz"
				COMMAND gzip -c "\"${CMAKE_SOURCE_DIR}/lmms.1\"" > "\"${CMAKE_BINARY_DIR}/lmms.1.gz\""
				DEPENDS "${CMAKE_SOURCE_DIR}/lmms.1"
				COMMENT "Generating lmms.1.gz")

ADD_CUSTOM_TARGET(manpage ALL
				DEPENDS "${CMAKE_BINARY_DIR}/lmms.1.gz")

# install headers

IF(LMMS_BUILD_LINUX)
	INSTALL(FILES ${LMMS_INCLUDES}
		"${CMAKE_BINARY_DIR}/lmmsconfig.h"
		"${CMAKE_BINARY_DIR}/lmmsversion.h"
		"${CMAKE_SOURCE_DIR}/src/gui/embed.cpp"
		DESTINATION "${CMAKE_INSTALL_PREFIX}/include/lmms/")
ENDIF(LMMS_BUILD_LINUX)

# package ZynAddSubFX into win32 build
IF(LMMS_BUILD_WIN32)
	IF(EXISTS "${CMAKE_SOURCE_DIR}/extras")
		ADD_SUBDIRECTORY("${CMAKE_SOURCE_DIR}/extras/data/presets")
		FILE(GLOB ZASF_BINARIES
			"${CMAKE_SOURCE_DIR}/extras/plugins/zynaddsubfx/zynaddsubfx.dll"
			"${CMAKE_SOURCE_DIR}/extras/plugins/zynaddsubfx/remote_zynaddsubfx.exe")
		INSTALL(FILES "${ZASF_BINARIES}" DESTINATION "${PLUGIN_DIR}")
	ENDIF(EXISTS "${CMAKE_SOURCE_DIR}/extras")
ENDIF(LMMS_BUILD_WIN32)

#
# add distclean-target
#
ADD_CUSTOM_TARGET(distclean
			COMMAND make clean
			COMMAND rm -rf `find -name cmake_install.cmake` `find -name Makefile` `find -type d -name CMakeFiles` CMakeCache.txt lmmsconfig.h lmms.1.gz)

#
# add tarball-target
#
SET(TMP "lmms-${VERSION}")
ADD_CUSTOM_TARGET(dist
			COMMAND make clean
			COMMAND rm -rf "${TMP}"
			COMMAND mkdir -p "${TMP}"
			COMMAND cp AUTHORS build_mingw32 build_mingw64 CMakeLists.txt configure COPYING INSTALL lmms.1 lmms.rc.in lmms.spec.in lmmsconfig.h.in lmmsversion.h.in README TODO "${TMP}"
			COMMAND cp -r buildtools cmake data doc include plugins src "${TMP}"
			COMMAND rm -rf `find "${TMP}" -name cmake_install.cmake` `find "${TMP}" -name Makefile` `find "${TMP}" -type d -name CMakeFiles` "${TMP}/CMakeCache.txt"
			COMMAND tar cjf lmms-${VERSION}-src.tar.bz2 "${TMP}"
			COMMAND rm -rf "${TMP}")

#
# display configuration information
#

MESSAGE("\n"
"Installation Summary\n"
"--------------------\n"
"* Install Directory           : ${CMAKE_INSTALL_PREFIX}\n"
)

MESSAGE(
"Supported audio interfaces\n"
"--------------------------\n"
"* ALSA                        : ${STATUS_ALSA}\n"
"* JACK                        : ${STATUS_JACK}\n"
"* OSS                         : ${STATUS_OSS}\n"
"* PortAudio                   : ${STATUS_PORTAUDIO}\n"
"* PulseAudio                  : ${STATUS_PULSEAUDIO}\n"
"* SDL                         : ${STATUS_SDL}\n"
)

MESSAGE(
"Supported MIDI interfaces\n"
"-------------------------\n"
"* ALSA                        : ${STATUS_ALSA}\n"
"* OSS                         : ${STATUS_OSS}\n"
"* WinMM                       : ${STATUS_WINMM}\n"
)

MESSAGE(
"Supported file formats for project export\n"
"-----------------------------------------\n"
"* WAVE                        : OK\n"
"* OGG/VORBIS                  : ${STATUS_OGGVORBIS}\n"
)

MESSAGE(
"Optional plugins\n"
"----------------\n"
"* Carla Patchbay & Rack       : ${STATUS_CARLA}\n"
"* SoundFont2 player           : ${STATUS_FLUIDSYNTH}\n"
"* Stk Mallets                 : ${STATUS_STK}\n"
"* VST-instrument hoster       : ${STATUS_VST}\n"
"* VST-effect hoster           : ${STATUS_VST}\n"
"* CALF LADSPA plugins         : ${STATUS_CALF}\n"
"* CAPS LADSPA plugins         : ${STATUS_CAPS}\n"
"* CMT LADSPA plugins          : ${STATUS_CMT}\n"
"* TAP LADSPA plugins          : ${STATUS_TAP}\n"
"* SWH LADSPA plugins          : ${STATUS_SWH}\n"
"* GIG player                  : ${STATUS_GIG}\n"
)

MESSAGE(
"\n"
"-----------------------------------------------------------------\n"
"IMPORTANT:\n"
"after installing missing packages, remove CMakeCache.txt before\n"
"running cmake again!\n"
"-----------------------------------------------------------------\n"
"\n\n")

INCLUDE(InstallRequiredSystemLibraries)
SET(CPACK_PACKAGE_DESCRIPTION_SUMMARY "LMMS - easy music production for everyone!")
SET(CPACK_PACKAGE_VENDOR "LMMS Developers")
IF(LMMS_BUILD_APPLE)
	CONFIGURE_FILE("${CMAKE_SOURCE_DIR}/README" "${CMAKE_BINARY_DIR}/README.txt" COPYONLY)
	CONFIGURE_FILE("${CMAKE_SOURCE_DIR}/COPYING" "${CMAKE_BINARY_DIR}/COPYING.txt" COPYONLY)
	SET(CPACK_PACKAGE_DESCRIPTION_FILE "${CMAKE_BINARY_DIR}/README.txt")
	SET(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_BINARY_DIR}/COPYING.txt")
ELSE(LMMS_BUILD_APPLE)
	SET(CPACK_PACKAGE_DESCRIPTION_FILE "${CMAKE_SOURCE_DIR}/README")
	SET(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_SOURCE_DIR}/COPYING")
ENDIF(LMMS_BUILD_APPLE)

SET(CPACK_PACKAGE_VERSION_MAJOR "${VERSION_MAJOR}")
SET(CPACK_PACKAGE_VERSION_MINOR "${VERSION_MINOR}")
SET(CPACK_PACKAGE_VERSION_PATCH "${VERSION_PATCH}")
IF(VERSION_SUFFIX)
	SET(CPACK_PACKAGE_VERSION_PATCH "${VERSION_PATCH}-${VERSION_SUFFIX}")
ENDIF(VERSION_SUFFIX)
SET(CPACK_PACKAGE_INSTALL_DIRECTORY "LMMS")
IF(WIN32)
	SET(CPACK_PACKAGE_ICON "${CMAKE_SOURCE_DIR}/data/nsis_branding.bmp")
	SET(CPACK_NSIS_MUI_ICON "${CMAKE_SOURCE_DIR}/data/lmms.ico")
	SET(CPACK_NSIS_INSTALLED_ICON_NAME "lmms.exe")
	SET(CPACK_NSIS_DISPLAY_NAME "LMMS ${VERSION}")
	SET(CPACK_NSIS_HELP_LINK "http://lmms.io")
	SET(CPACK_NSIS_URL_INFO_ABOUT "http://lmms.io")
	SET(CPACK_NSIS_CONTACT "lmms-devel@lists.sourceforge.net")
	SET(CPACK_PACKAGE_EXECUTABLES "lmms.exe;LMMS")
	SET(CPACK_NSIS_MENU_LINKS "lmms.exe;LMMS")
	SET(CPACK_NSIS_DEFINES "!include ${CMAKE_SOURCE_DIR}/cmake/nsis/FileAssociation.nsh")
	SET(CPACK_PACKAGE_FILE_NAME "lmms-${VERSION}-win32")
	SET(CPACK_NSIS_EXTRA_INSTALL_COMMANDS "
	  \\\${registerExtension} \\\"$INSTDIR\\\\lmms.exe\\\" \\\".mmp\\\" \\\"LMMS Project\\\"
	  \\\${registerExtension} \\\"$INSTDIR\\\\lmms.exe\\\" \\\".mmpz\\\" \\\"LMMS Project (compressed)\\\"
	  ")
	SET(CPACK_NSIS_EXTRA_UNINSTALL_COMMANDS "
	  \\\${unregisterExtension} \\\".mmp\\\" \\\"LMMS Project\\\"
	  \\\${unregisterExtension} \\\".mmpz\\\" \\\"LMMS Project (compressed)\\\"
	  DeleteRegKey HKCR \\\"LMMS Project\\\"
	  DeleteRegKey HKCR \\\"LMMS Project (compressed)\\\"
	  ")
ELSE(WIN32)
	SET(CPACK_STRIP_FILES "bin/lmms;${PLUGIN_DIR}/*.so")
	SET(CPACK_PACKAGE_EXECUTABLES "lmms" "LMMS binary")
ENDIF(WIN32)

IF(WIN64)
	SET(CPACK_PACKAGE_FILE_NAME "lmms-${VERSION}-win64")
	SET(CPACK_NSIS_DEFINES "${CPACK_NSIS_DEFINES}
		InstallDir \\\"\\\$PROGRAMFILES64\\\\${CPACK_PACKAGE_INSTALL_DIRECTORY}\\\"")
ENDIF(WIN64)

SET(CPACK_SOURCE_GENERATOR "TBZ2")
SET(CPACK_SOURCE_PACKAGE_FILE_NAME "lmms-${VERSION}")
INCLUDE(CPack)

