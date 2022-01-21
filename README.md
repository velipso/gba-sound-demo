GBA Sound Demo
==============

Playing the Demos
-----------------

The .gba files are in the repo, so you can download them directly:

* [rates.gba](https://raw.githubusercontent.com/velipso/gba-sound-demo/main/rates.gba)
* [song.gba](https://raw.githubusercontent.com/velipso/gba-sound-demo/main/song.gba)

![rates.gba](https://github.com/velipso/gba-sound-demo/raw/main/rates.png)

The rates demo will play a tone, and allow you to switch the sample rate, bit depth, dithering,
and hardware sample rate.

Controls:
* L/R - sweep tone
* Start - set tone to 256Hz
* Select - set tone to 4096Hz
* D-pad - change settings
* A/B - change between top or bottom settings (for quick comparison)

![song.gba](https://github.com/velipso/gba-sound-demo/raw/main/song.png)

The song demo will play a song at two sample rates -- the top is controlled via vblank, the bottom
is controlled via timers.

Controls:
* Start - restart song
* Anything else - swap between sample rates

How to Build
------------

You will need [gvasm](https://github.com/velipso/gvasm) to assemble the .gvasm files into .gba
files.

After install gvasm, run:

```
gvasm make rates.gvasm
```

In order to build the song demo, you must first download some music.

Here's what I did:

1. Installed [youtube-dl](https://ytdl-org.github.io/youtube-dl/download.html)

2. Installed [sox](http://sox.sourceforge.net/)

3. Downloaded the Super Mario 3 Medley:

```
youtube-dl -x --audio-format wav -o song.temp --prefer-ffmpeg \
  'https://www.youtube.com/watch?v=5Nu6BxCnyE0'
```

4. Convert it to 13K/8-Bit and 32K/8-Bit:

```
sox song.wav -r 13379 -b 8 -c 1 song-A.wav
sox song.wav -r 13379 -b 8 -c 1 song-A.raw
sox song.wav -r 32768 -b 8 -c 1 song-B.wav
sox song.wav -r 32768 -b 8 -c 1 song-B.raw
```

5. Build the .gba file:

```
gvasm make song.gvasm
```
