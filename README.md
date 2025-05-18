# Immich Screen recordings Organizer

This is a fork of [immich-screenshots-organizer](https://github.com/GreaseHeadD/immich-screenshots-organizer) from GreaseHeadD, but instead of organizing screenshots, it organizes screen recordings.

__Current compatibility:__ Immich v1.131.x and up

## Disclaimer
This script is based on the following repositories: 
- [immich-folder-album-creator](https://github.com/Salvoxia/immich-folder-album-creator/tree/main)
- [immich-screenshots-organizer](https://github.com/GreaseHeadD/immich-screenshots-organizer)

# Table of Contents
1. [Usage (Bare Python Script)](#bare-python-script)
2. [How It Works](#how-it-works)
3. [Improvements](#improvements)

## Usage
### Bare Python Script
1. Download the script and its requirements
    ```bash
    curl https://raw.githubusercontent.com/RandomDebugGuy/immich-screen-recording-organizer/main/immich_screenshots_organizer.py -o immich_screenshot_organizer.py
    curl https://raw.githubusercontent.com/RandomDebugGuy/immich-screen-recording-organizer/main/requirements.txt -o requirements.txt
    ```
2. Install requirements
    ```bash
    pip3 install -r requirements.txt
    ```
3. Run the script
    ```
usage: immich_screenrecord_organizer.py [-h] [-d] [-q QUERY] [--include-low-res] [-a] [-n LIBRARY_NAME] [-p IMPORT_PATH] [-u] [-c CHUNK_SIZE] [-C FETCH_CHUNK_SIZE] [-l {CRITICAL,ERROR,WARNING,INFO,DEBUG}] screen_recordings album_name api_url api_key

Organize screen recordings in immich via the search API using a specific filename match query

positional arguments:
  screen_recordings     ah fuck it i know what this does i probably wont push it to github idk
  album_name            The album name for your screen recordings
  api_url               The root API URL of immich, e.g. https://immich.mydomain.com/api/
  api_key               The Immich API Key to use

options:
  -h, --help            show this help message and exit
  -d, --dry-run         Don't change anything, but see what would be done (default: False)
  -q, --query QUERY     The search query to use. Default is screen (default: None)
  --include-low-res     Include photos that are low resolution(under 1400x1400/~2MP). Default is false (default: False)
  -a, --archive-recordings
                        Archives all screen recordings to hide them from the timeline. Default is false (default: False)
  -n, --library-name LIBRARY_NAME
                        The name of the library to look for screen recordings in, if empty all libraries will be searched. (default: None)
  -p, --import-path IMPORT_PATH
                        The import path of the library to look for screen recordings in, if left empty all libraries will be searched (default: None)
  -u, --unattended      Do not ask for user confirmation after identifying albums. Set this flag to run script as a cronjob. (default: False)
  -c, --chunk-size CHUNK_SIZE
                        Maximum number of assets to add to an album with a single API call (default: 2000)
  -C, --fetch-chunk-size FETCH_CHUNK_SIZE
                        Maximum number of assets to fetch with a single API call (default: 5000)
  -l, --log-level {CRITICAL,ERROR,WARNING,INFO,DEBUG}
                        Log level to use (default: INFO)
    ```

__Plain example without optional arguments:__
```bash
python3 ./immich_screenrecord_organizer.py screenrecordingsAlbumName https://immich.mydomain.com/api thisIsMyApiKeyCopiedFromImmichWebGui --other-Arguments
```
__Note: if specifying a library, import_path is preferred over library_name, because library names aren't unique.__


## Choosing the correct `import_path`
The import path  `/path/to/external/lib/` is the path you have mounted your external library into the Immich container.  
If you are following [Immich's External library Documentation](https://immich.app/docs/guides/external-library), you are
using an environment variable called `${EXTERNAL_PATH}` which is mounted to `/usr/src/app/external` in the Immich 
container. Your `import_path` to pass to the script is `/usr/src/app/external`.

## How it works

The original script utilized [Immich's REST API](https://immich.app/docs/api/) to query all images indexed by Immich, then 
queried the image info and checked in the exif data if 'exposureTime' is none, because screenshots never have that 
property set.

This script cannot use that method because videos (in my own testing) do not have exposureTime or anything else easily
distinguishable set in the exif data. As such, it instead filters only videos, and searches file names that have 'screen'.
As such, it is slightly less reliable, because if your screen recordings have a different alias (eg. ScrRec instead of
ScreenRecording, etc.), then they will not be found. To fix this however, you can set the `--query` option to search for
any particular pattern with your screen recordings.

## Improvements

The only possible way of distinguishing screen recordings from regular videos, in my own testing, is by filtering videos 
and searching for 'screen' in filenames.

Ways to improve the mechanism would be to make an educated guess based on the dimensions of the videos and the size of 
the file. The `--include-low-res` flag partly does this for videos smaller than ~2MP. However, with enough compression 
an actual video would still pass as a screen recording. For now this is the best it can do.

# Contributing

If you have any ideas/suggestions for new features or detection mechanisms, please open an issue. I would love to
hear your thoughts!
