#!/bin/bash
# text format helper
bold=$(tput bold)
normal=$(tput sgr0)

# program variables
CONFIG_FILE_LOC="$HOME"/.config/ytdlp-auth-download.config
PRINT_USAGE=0
OUTPUT_FLAGS=""
DOWNLOAD_FLAGS=""
FINAL_LOC="$(pwd)"
DOWNLOAD_SUBDIR="$RANDOM"
SOURCE_LOC=""
SOURCE_URL=""

# config variables
TEMP_DIR="/tmp/ytdlp-auth-download/"
THREADS=1
USE_SUBS=0
COOKIES_BROWSER=""
COOKIES_FILE=""

# arg variables
OUTPUT_FILE_NAME=""
SOURCE_URL=""
ARGS_COOKIES_FILE=""
ARGS_COOKIES_BROWSER=""
ARGS_CONFIG_FILE_LOC=""
ARGS_USE_SUBS="0"
ARGS_THREADS=""
ARGS_TEMP_DIR=""
ARGS_OUTPUT_DIR=""
ARGS_IGNORE_CONFIG=0

cleanup() {
  if [ -d "$TEMP_DIR/$DOWNLOAD_SUBDIR" ]; then
    find "${TEMP_DIR:?}/$DOWNLOAD_SUBDIR" -iname '*temp*' -exec rm {} \;
    echo "moving downloads to $FINAL_LOC"
    mv "${TEMP_DIR:?}/$DOWNLOAD_SUBDIR/"* "$FINAL_LOC"
    rm -rf "${TEMP_DIR:?}/$DOWNLOAD_SUBDIR"
  fi
}

trap cleanup EXIT ERR 

set -e

usage() {
  echo "${bold}Usage:${normal} ytdlp-auth-download ( -c | -b ) url | list-file
                        [ -t | --temp-dir ] [ -n | --num-threads ]
                        [ -o | --output ] [ -s | --subtitles ]
                        [ -C | --config ] [-h | --help ] 
                        [ -i | --ignore-config] [ -d | --output-dir ]"
  echo ""
  echo "${bold}Description:${normal}  A bash script wrapper around yt-dlp designed to make it easier to download
              videos from sites where cookie-based authorization is needed."
  echo ""
  echo "${bold}Options -b or -c MUST be supplied in order to properly authorize."
  echo ""
  echo "Extended help:"
  echo "${bold}-c, --cookies-file${normal}      a cookies.txt netscape-format cookies file to pass to yt-dlp in order to
                        authorize with a website. Required if ${bold}-b${normal} is not passed."
  echo ""
  echo "${bold}-b, --browser${normal}           takes the name of the browser you have valid login cookies for the website you are
                        attempting to download from. Required if ${bold}-c${normal} is not passed."
  echo ""
  echo "${bold}-t, --temp-dir${normal}          the location for yt-dlp to do its work inside inside. Useful for downloading
                        to remote storage appliances as threaded downloads generate hundreds to thousands
                        of file parts"
  echo ""
  echo "${bold}-n, --num-threads${normal}       the number of concurrent threads to download with. Default is 1."
  echo ""
  echo "${bold}-o, --output${normal}            output filename including file extension to save the final download as."
  echo ""
  echo "${bold}-s, --subtitles${normal}         option flag which enables the download of a .vtt subtitles file if one is available"
  echo ""
  echo "${bold}-C, --config${normal}            location of a config file if not located at the default location.
                        default location is $HOME/.config/ytdlp-auth-download.config"
  echo ""
  echo "${bold}-h, --help${normal}              print this help message"
  echo ""
  echo "${bold}-i, --ignore-config${normal}     tell this script not to set parameters from a config file if one exists."
  echo ""
  echo "${bold}-d, --output-dir${normal}        the final location for the downloaded file(s). Defaults to the present directory."
  exit 2
}

parse_list_file() {
  while IFS='|' read -r url output_file output_dir; do
    DOWNLOAD_SUBDIR="$RANDOM"
    mkdir -p "$TEMP_DIR/$DOWNLOAD_SUBDIR"
    if [ "$output_dir" != "" ]; then
      FINAL_LOC=$output_dir
    fi
    if [ "$output_file" != "" ]; then
      OUTPUT_FILE_NAME="$output_file"
    fi
    SOURCE_URL="$url"
    do_download
  done < $SOURCE_LOC
}

do_download() {
  # set output flags
  if [ "$OUTPUT_FILE_NAME" != "" ]; then
    OUTPUT_FLAGS="-o $OUTPUT_FILE_NAME"
  fi

  mkdir -p "$FINAL_LOC"
  cd "$TEMP_DIR/$DOWNLOAD_SUBDIR"
  yt-dlp $DOWNLOAD_FLAGS "$SOURCE_URL" $OUTPUT_FLAGS
  cleanup
}

PARSED_ARGUMENTS=$(getopt -n ytdlp-auth-download -o isho:C:c:b:n:t:d: --long ignore-config,subtitles,help,output:,config:,cookies-file:,browser:,num-threads:,temp-dir:output-dir: -- "$@")
VALID_ARGUMENTS=$?
if [ "$VALID_ARGUMENTS" != "0" ]; then
  usage
fi

eval set -- "$PARSED_ARGUMENTS"

while :
do
  case "$1" in
    -i | --ignore-config) ARGS_IGNORE_CONFIG=1        ; shift   ;;
    -s | --subtitles)     ARGS_USE_SUBS=1             ; shift   ;;
    -h | --help)          PRINT_USAGE=1               ; shift   ;;
    -o | --output)        OUTPUT_FILE_NAME="$2"       ; shift 2 ;;
    -C | --config)        ARGS_CONFIG_FILE_LOC="$2"   ; shift 2 ;;
    -c | --cookies-file)  ARGS_COOKIES_FILE="$2"      ; shift 2 ;;
    -b | --browser)       ARGS_COOKIES_BROWSER="$2"   ; shift 2 ;;
    -n | --num-threads)   ARGS_THREADS="$2"           ; shift 2 ;;
    -t | --temp-dir)      ARGS_TEMP_DIR="$2"          ; shift 2 ;;
    -d | --output-dir)    ARGS_OUTPUT_DIR="$2"        ; shift 2 ;;
    --) shift; break ;;
    *) echo "Unexpected argument encountered: $1"
       usage ;;
  esac
done

if [ "$PRINT_USAGE" != "0" ]; then
  usage
fi

if [ "${ARGS_IGNORE_CONFIG}" -eq 0 ]; then
  # load config file if one was passed as an argument
  if [ "${ARGS_CONFIG_FILE_LOC}" != "" ] && [ -f "${ARGS_CONFIG_FILE_LOC}" ]; then
    CONFIG_FILE_LOC="$ARGS_CONFIG_FILE_LOC"
  elif [ "${ARGS_CONFIG_FILE_LOC}" != "" ] && [ ! -f "${ARGS_CONFIG_FILE_LOC}" ]; then
    echo "Provided config file not found or could not be opened.
          Ensure that the file exists and that it has the proper
          permissions for reading."
    exit 1
  fi

  # grab and load config if it exists
  if [ -f "$CONFIG_FILE_LOC" ]; then
    . "$CONFIG_FILE_LOC"
  fi
fi

# set program parameters from args, overwriting config vals where present
if [ "${ARGS_THREADS}" != "" ]; then
  THREADS="$ARGS_THREADS"
fi

if [ "${ARGS_TEMP_DIR}" != "" ]; then
  TEMP_DIR="$ARGS_TEMP_DIR"
fi

if [ "${ARGS_USE_SUBS}" -eq 1 ]; then
  USE_SUBS=1
fi

if [ "${ARGS_OUTPUT_DIR}" != "" ]; then
  FINAL_LOC="$ARGS_OUTPUT_DIR"
fi

# validate conflicting args
if [ "${ARGS_COOKIES_FILE}" != "" ] && [ "${ARGS_COOKIES_BROWSER}" != "" ]; then
  echo "please provide either cookies file OR cookies browser,not both"
  usage
fi

# overwrite config/default vals from passed args
if [ "${ARGS_COOKIES_FILE}" != "" ]; then
  COOKIES_FILE="$ARGS_COOKIES_FILE"
  COOKIES_BROWSER=""
elif [ "${ARGS_COOKIES_BROWSER}" != "" ]; then
  COOKIES_BROWSER="$ARGS_COOKIES_BROWSER"
  COOKIES_FILE=""
fi

# make sure we have cookies
if [ "${COOKIES_FILE}" == "" ] && [ "${COOKIES_BROWSER}" == "" ]; then
  echo "Must provide either cookies file or browser to extract cookies from."
  usage
elif [ "${COOKIES_FILE}" != "" ] && [ "${COOKIES_BROWSER}" != "" ]; then
  echo "please provide either cookies file OR cookies browser, not both"
  usage
fi 

# confirm we have a url to pass to yt-dlp
if [ "$1" == "" ]; then
  echo "Please supply a URL for a video to download."
  usage
fi

SOURCE_LOC="$1"

# ensure temp dir exists

# build download flags
DOWNLOAD_FLAGS="-N $THREADS"

if [ "$USE_SUBS" != 0 ]; then
  DOWNLOAD_FLAGS="$DOWNLOAD_FLAGS --write-subs"
fi

if [ "${COOKIES_FILE}" != "" ]; then
  DOWNLOAD_FLAGS="$DOWNLOAD_FLAGS --cookies $COOKIES_FILE"
elif [ "${COOKIES_BROWSER}" != "" ]; then
  DOWNLOAD_FLAGS="$DOWNLOAD_FLAGS --cookies-from-browser $COOKIES_BROWSER"
fi


# check if we've been provided a list file or a URL to try, then run our downloads
if [ -f "${SOURCE_LOC}" ]; then
  parse_list_file
else
  SOURCE_URL="$SOURCE_LOC"
  do_download
fi

exit 0
