# mpv-config

The mpv configs and scripts i use.

## Requirements

- [subliminal](https://github.com/Diaoul/subliminal) is required for autosub
- [ffsubsync](https://github.com/smacke/ffsubsync) is required for autosubsync
- [ffmpeg 6+](https://ffmpeg.org/) is required for dynamic-crop
- [guessit](https://github.com/guessit-io/guessit) is required for guess-media-title

## Installation

1. Install requirements and make sure they are on your path
2. Clone into your mpv config directory
3. Create suitable `main.lua` files for `dynamic-crop`, `thumbfast` and `ModernZ`. See [1](https://github.com/Ashyni/mpv-scripts/pull/27), [2](https://github.com/po5/thumbfast/pull/136) and [3](https://github.com/Samillion/ModernZ/issues/133)
4. Install ModernZ font to system or by moving it to the appropriate mpv fonts directory.

## Scripts

- [autosubsync](https://github.com/joaquintorres/autosubsync-mpv)
  - Provides a keybind (`n`) to sync subtitles to audio
  - **Changes:** Added overwrite old subtitle option
- [autosub](https://github.com/davidde/mpv-autosub)
  - Automatically downloads subtitles via subliminal
  - **Changes:** Added option to toggle logging to osd of enabled subtitle
- [dynamic-crop](https://github.com/Ashyni/mpv-scripts)
  - Dynamically crops away blackbars from content
- [pause-when-minimize](https://github.com/mpv-player/mpv/blob/master/TOOLS/lua/pause-when-minimize.lua)
  - Pauses playback when mpv is minimized
- [ModernZ](https://github.com/Samillion/ModernZ)
  - OSC replacement
- [thumbfast](https://github.com/po5/thumbfast)
  - Generates seek thumbnails on-the-fly
- [guess-media-title](https://github.com/zenwarr/mpv-config/blob/master/scripts/guess-media-title.lua)
  - Uses guessit to detect media title and sets it via force-media-title
  - **Changes:** Added title format options
