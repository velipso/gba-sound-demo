GBA Sound Demo
==============

These demos allow you to experiment with different sample rates and bit depths on the Gameboy
Advance.

The GBA is notorious for having bad sound.  Using these demos, you can verify that the GBA hardware
is quite capable of good sound, but it takes special care to setup the timing correctly to produce
a 32K sample rate.

The source code is included, as .gvasm files.

Playing the Demos
-----------------

The .gba files are in the repo, so you can download them directly:

* [rates.gba](https://raw.githubusercontent.com/velipso/gba-sound-demo/main/rates.gba)
* [song.gba](https://raw.githubusercontent.com/velipso/gba-sound-demo/main/song.gba)

Both demos have a blinking indicator in the bottom left -- this is used to verify that timing is
correct when using timer based sample rates (16K, 32K, 65K).  Blinking indicates that the VBlank
and Timer IRQ fired at the same exact time.

Rates Demo
----------

![rates.gba](https://github.com/velipso/gba-sound-demo/raw/main/rates.png)

The rates demo will play a tone, and allow you to switch the sample rate, bit depth, dithering,
and hardware sample rate.

Controls:

* L/R - sweep tone
* Start - set tone to 256Hz
* Select - set tone to 4096Hz
* D-pad - change settings
* A/B - change between top or bottom settings (for quick comparison)

Song Demo
---------

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
