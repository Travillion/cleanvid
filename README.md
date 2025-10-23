# cleanvid

This is my personal fork to make my workflow a little more efficient. So far, I have updated the naming convention for the scrubbed SRT file and the EDL file so they work automatically with my playback software, and I have modified the EDL file to use HH:MM:SS.sss timestamps instead of S.sss. This just makes it easier for me to reference between the SRT and EDL files when I am customizing the mutes or skips. My usage is to only generate scrubbed SRT and EDL files (that is, I don't do any transcoding or create files for PlexAutoSkip). I don't believe my modifications have broken any of that functionality, but I also haven't tested it since I don't use those features.

See the banners below for links to the original author/work.

[![Latest Version](https://img.shields.io/pypi/v/cleanvid)](https://pypi.python.org/pypi/cleanvid/) [![Docker Image](https://github.com/mmguero/cleanvid/workflows/cleanvid-build-push-ghcr/badge.svg)](https://github.com/mmguero/cleanvid/pkgs/container/cleanvid)

**cleanvid** is a little script to mute profanity in video files in a few simple steps:

1. The user provides as input a video file and matching `.srt` subtitle file. If subtitles are not provided explicitly, they will be extracted from the video file if possible; if not, [`subliminal`](https://github.com/Diaoul/subliminal) is used to attempt to download the best matching `.srt` file.
2. [`pysrt`](https://github.com/byroot/pysrt) is used to parse the `.srt` file, and each entry is checked against a [list](./src/cleanvid/swears.txt) of profanity or other words or phrases you'd like muted. Mappings can be provided (eg., map "sh*t" to "poop"), otherwise the word will be replaced with *****.
3. A new "clean" `.srt` file is created. with *only* those phrases containing the censored/replaced objectional language.
4. [`ffmpeg`](https://www.ffmpeg.org/) is used to create a cleaned video file. This file contains the original video stream, but the specified audio stream is muted during the segments containing objectional language. That audio stream is re-encoded and remultiplexed back together with the video. Optionally, the clean `.srt` file can be embedded in the cleaned video file as a subtitle track.

You can then use your favorite media player to play the cleaned video file together with the cleaned subtitles.

As an alternative to creating a new video file, cleanvid can create a simple EDL file (see the [mplayer](http://www.mplayerhq.hu/DOCS/HTML/en/edl.html) or KODI [documentation](https://kodi.wiki/view/Edit_decision_list)) or a custom JSON definition file for [PlexAutoSkip](https://github.com/mdhiggins/PlexAutoSkip).

**cleanvid** is part of a family of projects with similar goals:

* 📼 [cleanvid](https://github.com/mmguero/cleanvid) for video files (using [SRT-formatted](https://en.wikipedia.org/wiki/SubRip#Format) subtitles)
* 🎤 [monkeyplug](https://github.com/mmguero/monkeyplug) for audio and video files (using either [Whisper](https://openai.com/research/whisper) or the [Vosk](https://alphacephei.com/vosk/)-[API](https://github.com/alphacep/vosk-api) for speech recognition)
* 📕 [montag](https://github.com/mmguero/montag) for ebooks
## Installation
To install directly from GitHub:


```
python3 -m pip install -U 'git+https://github.com/Travillion/cleanvid'
```

## Prerequisites

[cleanvid](./src/cleanvid/cleanvid.py) requires:

* Python 3
* [FFmpeg](https://www.ffmpeg.org)
* [babelfish](https://github.com/Diaoul/babelfish)
* [delegator.py](https://github.com/kennethreitz/delegator.py)
* [pysrt](https://github.com/byroot/pysrt)
* [subliminal](https://github.com/Diaoul/subliminal)

To install FFmpeg, use your operating system's package manager or install binaries from [ffmpeg.org](https://www.ffmpeg.org/download.html). The Python dependencies will be installed automatically if you are using `pip` to install cleanvid.

## usage

```
usage: cleanvid [-h] [-s <srt>] -i <input video> [-o <output video>] [--plex-auto-skip-json <output JSON>] [--plex-auto-skip-id <content identifier>] [--subs-output <output srt>]
                [-w <profanity file>] [-l <language>] [-p <int>] [-e] [-f] [--subs-only] [--offline] [--edl] [--json] [--re-encode-video] [--re-encode-audio] [-b] [-v VPARAMS] [-a APARAMS]
                [-d] [--audio-stream-index <int>] [--audio-stream-list] [--threads-input <int>] [--threads-encoding <int>] [--threads <int>]

options:
  -h, --help            show this help message and exit
  -s <srt>, --subs <srt>
                        .srt subtitle file (will attempt auto-download if unspecified and not --offline)
  -i <input video>, --input <input video>
                        input video file
  -o <output video>, --output <output video>
                        output video file
  --plex-auto-skip-json <output JSON>
                        custom JSON file for PlexAutoSkip (also implies --subs-only)
  --plex-auto-skip-id <content identifier>
                        content identifier for PlexAutoSkip (also implies --subs-only)
  --subs-output <output srt>
                        output subtitle file
  -w <profanity file>, --swears <profanity file>
                        text file containing profanity (with optional mapping)
  -l <language>, --lang <language>
                        language for extracting srt from video file or srt download (default is "eng")
  -p <int>, --pad <int>
                        pad (seconds) around profanity
  -e, --embed-subs      embed subtitles in resulting video file
  -f, --full-subs       include all subtitles in output subtitle file (not just scrubbed)
  --subs-only           only operate on subtitles (do not alter audio)
  --offline             don't attempt to download subtitles
  --edl                 generate MPlayer EDL file with mute actions (also implies --subs-only)
  --json                generate JSON file with muted subtitles and their contents
  --re-encode-video     Re-encode video
  --re-encode-audio     Re-encode audio
  -b, --burn            Hard-coded subtitles (implies re-encode)
  -v VPARAMS, --video-params VPARAMS
                        Video parameters for ffmpeg (only if re-encoding)
  -a APARAMS, --audio-params APARAMS
                        Audio parameters for ffmpeg
  -d, --downmix         Downmix to stereo (if not already stereo)
  --audio-stream-index <int>
                        Index of audio stream to process
  --audio-stream-list   Show list of audio streams (to get index for --audio-stream-index)
  --threads-input <int>
                        ffmpeg global options -threads value
  --threads-encoding <int>
                        ffmpeg encoding options -threads value
  --threads <int>       ffmpeg -threads value (for both global options and encoding)
```

## Contributing

If you'd like to help improve cleanvid, pull requests will be welcomed!

## Authors

* **Seth Grover** - *Initial work* - [mmguero](https://github.com/mmguero)
* **Travis Nichols** - *Tweaks to suit my personal workflow* - [Travillion](https://github.com/Travillion)

## License

This project is licensed under the BSD 3-Clause License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

Thanks to:

* the developers of [FFmpeg](https://www.ffmpeg.org/about.html)
* [Mattias Wadman](https://github.com/wader) for his [ffmpeg](https://github.com/wader/static-ffmpeg) image
* [delegator.py](https://github.com/kennethreitz/delegator.py) developer Kenneth Reitz and contributors
* [pysrt](https://github.com/byroot/pysrt) developer Jean Boussier and contributors
* [subliminal](https://github.com/Diaoul/subliminal) developer Antoine Bertin and contributors
