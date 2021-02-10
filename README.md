Bad Apple CMake
=====

CMake script that displays Bad Apple in your terminal using [Braille Patterns](https://en.wikipedia.org/wiki/Braille_Patterns) and with slight ANSI escape codes for cursor positioning.

How it works
-----

### https://auahdark687291.blogspot.com/2021/02/bad-apple-in-cmake-how-it-works.html

Video
-----

### https://www.youtube.com/watch?v=Ro92Qs0JLvg

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

Changing Dimensions
-----

Execute then copy the dimensions you desired.

```
luajit -e "for i = 1, 360 do local w = i * 4 / 3 if w % 1 == 0 and w % 2 == 0 and i % 4 == 0 then print(w, i) end end"
```

Some limitations apply, check the CMakeLists.txt file of `WIDTH` and `HEIGHT` variable declaration for more details.

License
-----

Just, public domain.
