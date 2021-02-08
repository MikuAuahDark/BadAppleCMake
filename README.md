Bad Apple CMake
=====

CMake script that displays Bad Apple in your terminal using [Braille Patterns](https://en.wikipedia.org/wiki/Braille_Patterns) and with slight ANSI escape codes for cursor positioning.

How it works
-----

TODO in my blog post.

Video
-----

TODO: Add video link here

Requirements
-----

* [youtube-dl](https://github.com/ytdl-org/youtube-dl)

* [FFmpeg](https://www.ffmpeg.org/)

Run
-----

```
cmake -Bbuild -H.
cmake --build build --target BadApple
```

Notes
-----

* If you got errors when downloading the video, try to update your youtube-dl.

* If you got errors when converting frames, either your build of FFmpeg doesn't support VP9 decoding or doesn't support [PBM](https://en.wikipedia.org/wiki/Netpbm#PBM_example) decoding. This is rare, you should ask to the person who compiled the FFmpeg!

* You can't stop the process in Windows when you use MSBuild (default). Ninja generator doesn't have this problem though.

License
-----

Just, public domain.
