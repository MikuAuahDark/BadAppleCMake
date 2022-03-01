# Tests on older version of CMake are welcome
cmake_minimum_required(VERSION 3.16.3)

project(BadApple LANGUAGES NONE)

if (POLICY CMP0053)
	cmake_policy(SET CMP0053 NEW)
endif ()

# Assert. Just please obey!
execute_process(COMMAND "cmd" "/c" "chcp"
	OUTPUT_VARIABLE CODEPAGE
)
string(REGEX MATCH "([0-9]+)" CODEPAGE_NUM "${CODEPAGE}")
if (NOT ${CODEPAGE_NUM} STREQUAL "65001")
	message(FATAL_ERROR "Windows 10 with UTF-8 mode is required. Do not attempt to remove this assert!")
endif ()

# Hardcoded number of frames, should match with WebM version of the video
set(NFRAMES 3777)

# Dimensions must be 16:9 ration where the width is divisible by 2
# and the height is divisible by 4
# Also you need to ensure the height fits in the terminal or ANSI escape codes
# won't work properly
# Furthermore, adjust the read offset below if WIDTH and/or HEIGHT is not exactly 3 characters
set(WIDTH 256)
set(HEIGHT 144)

if (PRINTER_MODE)
	# Start the show
	option(FRAMES_PATH "a")

	set(BRAILLE_CHAR)
	list(APPEND BRAILLE_CHAR
		"⠀;⠁;⠂;⠃;⠄;⠅;⠆;⠇;⠈;⠉;⠊;⠋;⠌;⠍;⠎;⠏"
		"⠐;⠑;⠒;⠓;⠔;⠕;⠖;⠗;⠘;⠙;⠚;⠛;⠜;⠝;⠞;⠟"
		"⠠;⠡;⠢;⠣;⠤;⠥;⠦;⠧;⠨;⠩;⠪;⠫;⠬;⠭;⠮;⠯"
		"⠰;⠱;⠲;⠳;⠴;⠵;⠶;⠷;⠸;⠹;⠺;⠻;⠼;⠽;⠾;⠿"
		"⡀;⡁;⡂;⡃;⡄;⡅;⡆;⡇;⡈;⡉;⡊;⡋;⡌;⡍;⡎;⡏"
		"⡐;⡑;⡒;⡓;⡔;⡕;⡖;⡗;⡘;⡙;⡚;⡛;⡜;⡝;⡞;⡟"
		"⡠;⡡;⡢;⡣;⡤;⡥;⡦;⡧;⡨;⡩;⡪;⡫;⡬;⡭;⡮;⡯"
		"⡰;⡱;⡲;⡳;⡴;⡵;⡶;⡷;⡸;⡹;⡺;⡻;⡼;⡽;⡾;⡿"
		"⢀;⢁;⢂;⢃;⢄;⢅;⢆;⢇;⢈;⢉;⢊;⢋;⢌;⢍;⢎;⢏"
		"⢐;⢑;⢒;⢓;⢔;⢕;⢖;⢗;⢘;⢙;⢚;⢛;⢜;⢝;⢞;⢟"
		"⢠;⢡;⢢;⢣;⢤;⢥;⢦;⢧;⢨;⢩;⢪;⢫;⢬;⢭;⢮;⢯"
		"⢰;⢱;⢲;⢳;⢴;⢵;⢶;⢷;⢸;⢹;⢺;⢻;⢼;⢽;⢾;⢿"
		"⣀;⣁;⣂;⣃;⣄;⣅;⣆;⣇;⣈;⣉;⣊;⣋;⣌;⣍;⣎;⣏"
		"⣐;⣑;⣒;⣓;⣔;⣕;⣖;⣗;⣘;⣙;⣚;⣛;⣜;⣝;⣞;⣟"
		"⣠;⣡;⣢;⣣;⣤;⣥;⣦;⣧;⣨;⣩;⣪;⣫;⣬;⣭;⣮;⣯"
		"⣰;⣱;⣲;⣳;⣴;⣵;⣶;⣷;⣸;⣹;⣺;⣻;⣼;⣽;⣾;⣿"
	)

	# Helper function to help converting 4 byte of data to 4 braille characters
	# Braille characters are ordered in this order according to unicode
	# (1) (4)
	# (2) (5)
	# (3) (6)
	# (7) (8)
	function(to_braille RESULT BYTE1 BYTE2 BYTE3 BYTE4)
		set(_RESULT0)
		foreach (i RANGE 0 6 2)
			math(EXPR _BTEMP "((${BYTE1} >> (${i} + 1)) & 1) | (((${BYTE2} >> (${i} + 1)) & 1) << 1) | (((${BYTE3} >> (${i} + 1)) & 1) << 2)")
			math(EXPR _BTEMP "${_BTEMP} | (((${BYTE1} >> ${i}) & 1) << 3) | (((${BYTE2} >> ${i}) & 1) << 4) | (((${BYTE3} >> ${i}) & 1) << 5)")
			math(EXPR _BTEMP "${_BTEMP} | (((${BYTE4} >> (${i} + 1)) & 1) << 6) | (((${BYTE4} >> ${i}) & 1) << 7)")
			list(GET BRAILLE_CHAR ${_BTEMP} _BCHAR)
			string(PREPEND _RESULT0 ${_BCHAR})
		endforeach()
		set(${RESULT} ${_RESULT0} PARENT_SCOPE)
	endfunction()

	function(image_index RESULT FRAME_DATA X Y STRIDE)
		math(EXPR NUMERIC_INDEX "(${Y} * ${STRIDE} + ${X}) * 2")
		string(SUBSTRING ${FRAME_DATA} ${NUMERIC_INDEX} 2 _RESULT0)
		math(EXPR _RESULT0 "0x${_RESULT0}")
		set(${RESULT} ${_RESULT0} PARENT_SCOPE)
	endfunction()

	function(zeropad2 RESULT STR)
		string(LENGTH ${STR} SLEN)
		if (SLEN LESS 2)
			set(${RESULT} "0${STR}" PARENT_SCOPE)
		else()
			set(${RESULT} ${STR} PARENT_SCOPE)
		endif()
	endfunction()

	math(EXPR WIDTH_BYTE "(${WIDTH} + 7) / 8")
	math(EXPR WIDTH_BYTE_M1 "${WIDTH_BYTE} - 1")
	math(EXPR HEIGHT_ITER "${HEIGHT} - 1")
	math(EXPR HEIGHT_TERM "${HEIGHT} / 4 + 2")
	string(LENGTH ${WIDTH} WIDTHSTR_LEN)
	string(LENGTH ${HEIGHT} HEIGHTSTR_LEN)
	string(ASCII 27 ESC)
	math(EXPR READ_OFFSET "${WIDTHSTR_LEN} + ${HEIGHTSTR_LEN} + 5")

	foreach (i RANGE 1 ${NFRAMES})
		file(READ "${FRAMES_PATH}/${i}.pbm" FRAME_DATA OFFSET ${READ_OFFSET} HEX)

		# Show current video time
		math(EXPR TIME_SECONDS "${i} / 15")
		math(EXPR TIME_MINUTES "${TIME_SECONDS} / 60")
		math(EXPR TIME_SECONDS "${TIME_SECONDS} % 60")
		zeropad2(TIME_SECONDS ${TIME_SECONDS})
		zeropad2(TIME_MINUTES ${TIME_MINUTES})
		set(WHAT_TO_PRINT "Frames: ${i}, Playback ${TIME_MINUTES}:${TIME_SECONDS}")

		# Due to PBM binary representation, this means we need to process
		# the images at 8x4 pixels
		foreach (y RANGE 0 ${HEIGHT_ITER} 4)
			set(COLUMNS_TO_PRINT)

			foreach (x RANGE 0 ${WIDTH_BYTE_M1})
				# Index 4 bytes down
				image_index(IMAGEDATA1 ${FRAME_DATA} ${x} ${y} ${WIDTH_BYTE})
				math(EXPR YTEMP "${y} + 1")
				image_index(IMAGEDATA2 ${FRAME_DATA} ${x} ${YTEMP} ${WIDTH_BYTE})
				math(EXPR YTEMP "${YTEMP} + 1")
				image_index(IMAGEDATA3 ${FRAME_DATA} ${x} ${YTEMP} ${WIDTH_BYTE})
				math(EXPR YTEMP "${YTEMP} + 1")
				image_index(IMAGEDATA4 ${FRAME_DATA} ${x} ${YTEMP} ${WIDTH_BYTE})
				to_braille(BRAILLE_TEMP ${IMAGEDATA1} ${IMAGEDATA2} ${IMAGEDATA3} ${IMAGEDATA4})
				# Append
				string(APPEND COLUMNS_TO_PRINT ${BRAILLE_TEMP})
			endforeach ()

			list(APPEND WHAT_TO_PRINT ${COLUMNS_TO_PRINT})
		endforeach ()

		list(JOIN WHAT_TO_PRINT "\n-- " THIS_IS_TO_PRINT)
		message(STATUS "${THIS_IS_TO_PRINT}")
		message(STATUS "${ESC}[${HEIGHT_TERM}A")
	endforeach ()
	
	message(STATUS "${ESC}[${HEIGHT_TERM}B")
	return()
else ()
	set(FRAMES_PATH "${CMAKE_CURRENT_BINARY_DIR}/frames")
endif ()

function(severity_printer TEXT RESULT)
	if (NOT RESULT)
		message(FATAL_ERROR "${TEXT}: ${RESULT}")
	else ()
		message(STATUS "${TEXT}: ${RESULT}")
	endif ()
endfunction()

function(find_ytdl PROGNAME RESULT RESULT_CMDLINE)
	find_program(PROGRAM_PATH ${PROGNAME})

	if (PROGRAM_PATH)
		set(${RESULT} ${PROGRAM_PATH} PARENT_SCOPE)
		set(${RESULT_CMDLINE} "" PARENT_SCOPE)
	else ()
		find_package(Python3 COMPONENTS Interpreter)

		if (Python3_Interpreter_FOUND)
			string(REPLACE "-" "_" PROGRAM_MODULE ${PROGNAME})

			execute_process(COMMAND ${Python3_EXECUTABLE} "-c" "import ${PROGRAM_MODULE}"
				RESULT_VARIABLE ERRORLEVEL
			)
			if (${ERRORLEVEL} STREQUAL "0")
				set(${RESULT} ${Python3_EXECUTABLE} PARENT_SCOPE)
				set(${RESULT_CMDLINE} -m ${PROGRAM_MODULE} PARENT_SCOPE)
			else ()
				set(${RESULT} ${PROGNAME}-NOTFOUND PARENT_SCOPE)
				set(${RESULT_CMDLINE} "" PARENT_SCOPE)
			endif ()
		else ()
			set(${RESULT} ${PROGNAME}-NOTFOUND PARENT_SCOPE)
			set(${RESULT_CMDLINE} "" PARENT_SCOPE)
		endif ()
	endif ()
endfunction()

# Look for FFmpeg
find_program(FFMPEG ffmpeg)
severity_printer("FFmpeg" ${FFMPEG})

# Look for youtube-dl
find_ytdl("yt-dlp" YOUTUBE_DL YOUTUBE_DL_ARGS)
if (NOT YOUTUBE_DL)
	find_ytdl("youtube-dl" YOUTUBE_DL YOUTUBE_DL_ARGS)
endif ()

severity_printer("youtube-dl" ${YOUTUBE_DL})

# Download video
add_custom_command(OUTPUT "${CMAKE_CURRENT_BINARY_DIR}/ラグトレイン"
	COMMAND "${YOUTUBE_DL}" ARGS ${YOUTUBE_DL_ARGS} -f "248" -o "${CMAKE_CURRENT_BINARY_DIR}/ラグトレイン" UnIhRpIT7nc
	COMMENT "Downloading ラグトレイン"
	USES_TERMINAL
)

# Convert frames
file(MAKE_DIRECTORY "${FRAMES_PATH}")
set(FILES_TO_GENERATE)

foreach (i RANGE 1 ${NFRAMES})
	list(APPEND FILES_TO_GENERATE "${FRAMES_PATH}/${i}.pbm")
endforeach ()

add_custom_command(OUTPUT ${FILES_TO_GENERATE}
	COMMAND "${FFMPEG}" -hide_banner -y -i "${CMAKE_CURRENT_BINARY_DIR}/ラグトレイン" -filter:v scale=${WIDTH}x${HEIGHT},format=monob "${FRAMES_PATH}/%d.pbm"
	COMMENT "Coverting videos to PBM"
	DEPENDS "${CMAKE_CURRENT_BINARY_DIR}/ラグトレイン"
	VERBATIM
	USES_TERMINAL
)

# The actual target to run
add_custom_target(BadApple ALL
	COMMAND ${CMAKE_COMMAND} "-B${CMAKE_CURRENT_BINARY_DIR}/printer" "-H${CMAKE_CURRENT_SOURCE_DIR}" "-G${CMAKE_GENERATOR}" "-DPRINTER_MODE=1" "-DFRAMES_PATH=${FRAMES_PATH}"
	COMMENT "Starting ラグトレイン"
	DEPENDS ${FILES_TO_GENERATE}
	USES_TERMINAL
)
