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
    usage: immich_screenshots_organizer.py [-h] [-u] [--include-exifless] [--archive-screens]  [-n LIBRARY_NAME] [-p IMPORT_PATH] [-c CHUNK_SIZE] [-C FETCH_CHUNK_SIZE] [-l {CRITICAL,ERROR,WARNING,INFO,DEBUG}] album_name api_url api_key

    Create Immich album for screenshots and add them. Hide(archive) if desired.

    positional arguments:
      album_name             The album name where the screenshots will reside
      api_url               The root API URL of immich, e.g. https://immich.mydomain.com/api/
      api_key               The Immich API Key to use

    options:
      -h, --help            show this help message and exit
   
      -u, --unattended      Do not ask for user confirmation after identifying screenshots. Set this flag to run script as a cronjob. (default: False)
   
      --include-low-res
                            Will add photos that are low resolution(under 1400x1400/~2MP) to the screenshots album as well. (default: False)
   
      --with-smartsearch
                            Will add screenshots found using smartsearch to the screenshots album as well. (default: False)
   
      --archive-screens
                            Will archive the screenshots as to hide them from the timeline. (default: False)
   
      -n LIBRARY_NAME, --library-name LIBRARY_NAME
                            When set, will only fetch assets from the first library that matches library_name. If --import-path is also set --library-name will precede. (default: None)
   
      -p IMPORT_PATH, --import-path IMPORT_PATH
                            When set, will only fetch assets from the first library that matches import_path. If --library-name is also set --library-name will precede. (default: None)
   
      -c CHUNK_SIZE, --chunk-size CHUNK_SIZE
                            Maximum number of assets to add to the screenshots album with a single API call. (default: 2000)
   
      -C FETCH_CHUNK_SIZE, --fetch-chunk-size FETCH_CHUNK_SIZE
                            Maximum number of assets to fetch with a single API call. (default: 5000)
   
      -l {CRITICAL,ERROR,WARNING,INFO,DEBUG}, --log-level {CRITICAL,ERROR,WARNING,INFO,DEBUG}
                            Log level to use. (default: INFO)
    ```

__Plain example without optional arguments:__
```bash
python3 ./immich_screenshots_organizer.py screenshotsAlbumName https://immich.mydomain.com/api thisIsMyApiKeyCopiedFromImmichWebGui
```
__Note: if specifying a library, import_path is preferred over library_name, because library names aren't unique.__


## Choosing the correct `import_path`
The import path  `/path/to/external/lib/` is the path you have mounted your external library into the Immich container.  
If you are following [Immich's External library Documentation](https://immich.app/docs/guides/external-library), you are
using an environment variable called `${EXTERNAL_PATH}` which is mounted to `/usr/src/app/external` in the Immich container. Your `import_path` to pass to the script is `/usr/src/app/external`.

## How it works

The script utilizies [Immich's REST API](https://immich.app/docs/api/) to query all images indexed by Immich, then 
queries the image info and checks in the exif data if 'exposureTime' is none, because screenshots never have that 
property set. Then it creates the screenshots album if it doesn't exist yet and adds the images to them. If desired it will also update the images to be archived.

## Improvements

The mechanism for detecting screenshots is very simple and can thus also fail. For example if a picture is stripped of 
all it's exif data(this can happen with Whatsapp, OneDrive, etc.) it is impossible to accurately detect if a picture is 
a screenshot with the API that Immich provides. I'm aware that Apple devices will add 'screenshot' as comment under 
exifComment and XMPComment, but since this is not exposed by the Immich API trying to read this out would add 
unnecessary complexity.

I've also investigated using smart search to find screenshots, but this can be unreliable as regular pictures
can also show up in the resulting assets. For now smartsearch can be toggled on using the `--with-smartsearch` flag.

Ways to improve the mechanism would be to make an educated guess based on the dimensions of the images and the size of 
the file. The `--include-low-res` flag partly does this for photos smaller than ~2MP. However, with enough compression 
an actual photo would still pass as a screenshot. For now this is the best it can do.
