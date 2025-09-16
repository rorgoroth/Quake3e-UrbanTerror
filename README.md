# Quake3e-Urbanterror

This is a Modded Quake3e with minimal changes to be able to run Urbanterror based on <https://github.com/omg-urt/urbanterror-slim>. It has better performance than stock binary. If you have fps drops with stock binary, try it.

Latest builds: <https://github.com/rorgoroth/mingw-cmake-env/releases>

Download and extract to where you have the stock urbanterror binary. Then run.

# Quake3e: Features and Technical Details

This client aims to be fully compatible with original baseq3 and other mods while trying to be fast, reliable and bug-free.
It is based on ioquake3-r1160 (latest non-SDL revision) with upstream patches and many custom improvements.

**Common changes/additions:**

* a lot of security, performance and bug fixes
* much improved autocompletion (map, demo, exec and other commands), in-game **\\callvote** argument autocompletion
* **\\com\_affinityMask** - bind Quake3e process to bitmask-specified CPU core(s)
* raized filesystem limits, much faster startup with 1000+ pk3 files in use, level restart times were also reduced as well
* **\\fs\_locked** **0**|1 - keep opened pk3 files locked or not, removes pk3 file limit when unlocked

**Client-specific changes/additions:**

* raw mouse input support, enabled automatically instead of DirectInput(**\\in\_mouse 1**) on Windows XP and newer windows operating systems
* unlagged mouse processing, can be reverted by setting **\\in\_lagged 1**
* MOUSE4 and MOUSE5 works in **\\in\_mouse -1** mode
* **\\minimize** in-game command to minimize main window, can be used with binds/scripting
* [**\\in\_minimize**](#in_minimize) - hotkey for minimize/restore main window (direct replacement for Q3Minimizer)
* **\\in\_forceCharset** 0|**1**|2 - try to translate non-ASCII chars in keyboard input (**1**) or force EN/US keyboard layout (2)
* **\\in\_nograb** **0**|1 - do not capture mouse in game, may be useful during online streaming
* **\\s\_muteWhenUnfocused** 0|**1**
* **\\s\_muteWhenMinimized** 0|**1**
* **\\s\_device** - linux-only, specified sound device to use with ALSA, enter aplay -L in your shell to see all available options
* **\\screenshotBMP** and **\\screenshotBMP clipboard** commands
* hardcoded PrintScreen key - for "\\screenshotBMP clipboard"
* hardcoded Shift+PrintScreen - for "\\screenshotBMP"
* **\\com\_maxfpsUnfocused** - will save cpu when inactive,set to your desktop refresh rate, for example
* **\\com\_skipIdLogo** **0**|1\- skip playing idlogo movie at startup
* **\\com\_yieldCPU** <milliseconds> - try to sleep specified amout of time between rendered frames when game is active, this will greatly reduce CPU load, use **0** only if you're experiencing some lags (also it is usually reduces performance on integrated graphics because CPU steals GPU's power budget)
* **\\r\_defaultImage** <filename>|#rgb|#rrggbb - replace default (missing) image texture by either exact file or solid #rgb|#rrggbb background color
* **\\r\_vbo** **0**|1 - use Vertex Buffer Objects to cache static map geometry, may improve FPS on modern GPUs, increases hunk memory usage by 15-30MB (map-dependent)

* **\\r\_fbo** **0**|1 - use framebuffer objects, enables gamma correction in windowed mode and allows arbitrary size (i.e. greater than logical desktop resolution) screenshot/video capture, required for bloom, hdr rendering, anti-aliasing, greyscale effects, OpenGL 3.0+ required
* **\\r\_hdr** \-1|**0**|1 - select texture format for framebuffer:
      -1 - 4-bit, for testing purposes, heavy color banding, might not work on all systems
       0 - 8 bit, default, moderate color banding with multi-stage shaders
       1 - 16 bit, enhanced blending precision, no color banding, might decrease performance on AMD/Intel GPUs
  * **[\\r\_bloom](#r_bloom)** **0**|1|2 - high-quality light bloom postprocessing effect
* **\\r\_dlightMode** 0|**1**|2 - dynamic light mode
   0 - VQ3 'fake' dynamic lights
   1 - new high-quality per-pixel dynamic lights, slightly faster than VQ3's on modern hardware
   2 - same as 1 but applies to all MD3 models too
* **\\r\_modeFullscreen** - dedicated mode string for fullscreen mode, set to -2 to use desktop resolution, set empty to use **\\r\_mode** in all cases
* **\\r\_nomip** **0**|1\- apply picmip only on worldspawn textures
* **\\r\_neatsky** **0**|1 - nopicmip for skyboxes
* **\\r\_greyscale** \[**0**..1.0\] - desaturates rendered frame, requires **[\\r\_fbo 1](#r_fbo)**, can be changed on the fly
* **\\r\_mapGreyScale** \[-1.0..1.0\] - desaturate world map textures only, works independently from **\\r\_greyscale**, negative values desaturates lightmaps only
* **\\r\_ext\_multisample** **0**|2|4|6|8 - multi-sample anti-aliasing, requires **[\\r\_fbo 1](#r_fbo)**, can be changed on the fly
* **\\r\_ext\_supersample** **0**|1 - super-sample anti-aliasing, requires **[\\r\_fbo 1](#r_fbo)**
* **\\r\_noborder** **0**|1 - to draw game window without border, hold ALT to drag & drop it with opened console
* **\\r\_noportals** **0**|1|2 - disable in-game portals (1), and mirrors too (2)
* negative **\\r\_overBrightBits** - force hardware gamma in windowed mode _(not actual with **[\\r\_fbo 1](#r_fbo)**)_
* [**\\video-pipe**](#video-pipe) <filename> - redirect captured video to ffmpeg input pipe and save encoded file as <filename>
* [**\\r\_renderWidth** & **\\r\_renderHeight**](#arr) - arbitrary resolution rendering, requires **[\\r\_fbo 1](#r_fbo)**
* **\\cl\_conColor \[RRR GGG BBB AAA\]** - custom console color, non-archived, use **\\seta** command to set archive flag and store in config
* **\\cl\_autoNudge** \[**0**..1\] - automatic time nudge that uses your average ping as the time nudge, values:
       0 - use fixed **\\cl\_timeNudge**
       (0..1\] - factor of median average ping to use as timenudge
* **\\cl\_mapAutoDownload** **0**|1 - automatic map download for play and demo playback (via automatic [**\\dlmap**](#dlmap) call)
* less spam in console (try to set "\\developer 1" to see what important things you missing)
* faster shader loading, tolerant to non-fatal errors
* **fast client downloads (http/ftp redirection)**
* [**\\download** and **dlmap** commands](#dlmap) - fast client-initiated downloads from specified map server
* **you can use \\record during \\demo playback**
* [**conditional shader stages**](#condstages)
* **linear dynamic lights**

**Server-specific changes/additions:**

* **\\sv\_levelTimeReset** **0**|1 - reset or do not reset leveltime after new map loads, when enabled - fixes gfx for clients affected by "frameloss" bug, however it may be necessary disable in case of troubles with GTV
* **\\sv\_maxclientsPerIP** - limit number of simultaneous connections from the same IP address
* much improved DDoS protection
* **\\sv\_minPing** and **\\sv\_maxPing** were removed because of new much better client connecion code
* userinfo filtering system, see docs/filter.txt
* **rcon** now is always available on dedicated servers
* **rconPassword2** - hidden master rcon password that can be set only from command line, i.e.
       **+set rconPassword2 "123456"**
    can be used to change/revoke compromised **rconPassword**
* significally reduced memory usage for client slots

* * *

**\\in\_minimize**

cvar that specifies hotkey for fast minimize/restore main windows, set values in form
  **\\in\_minimize "ctrl+z"**
  **\\in\_minimize "lshift+ralt+\\"**
  or so then do **\\in\_restart** to apply changes.

* * *

**Fast client downloads**

Usually downloads in Q3 is slow because all dowloaded data is requested/revieced from game server directly. So called \`download redirection' allows very fast downloads because game server just tells you from where you should download all required mods/maps etc. - it can be internet or local http/ftp server
So what you need:

* set **sv\_dlURL** cvar to download location, for example:
     **sets sv\_dlURL "ftp://myftp.com/games/q3"**
    Please note that your link should point on root game directory not particular mod or so

* Fill your download location with files.
      Please note that you should post ALL files that client may download, don't worry about pk3 overflows etc. on client side as client will download and use only required files

* * *

**Fast client-initiated downloads**

You can easy download pk3 files you need from any ftp/http/etc resource via following commands:
**\\download** **filename**\[.pk3\]
**\\dlmap** **mapname**\[.pk3\]

**\\dlmap** is the same as **\\download** but also will check for map existence

**cl\_dlURL** cvar must points to download location, where (optional) **%1** pattern will be replaced by **\\download|dlmap** command argument (filename/mapmane) and HTTP header may be analysed to obtain destination filename

For example:

  1) If **cl\_dlURL** is **<http://127.0.0.1>** and you type **\\download promaps.pk3** -   resulting download url will be **<http://127.0.0.1/promaps.pk3>**
  2) If **cl\_dlURL** is **<http://127.0.0.1/%1>** and you type **\\dlmap dm17** -   resulting download url will be **<http://127.0.0.1/dm17>**
  Also in this case HTTP-header will be analysed and resulting filename may be changed

To stop download just specify '-' as argument:

  **\\dlmap -**

**cl\_dlDirectory** cvar specifies where to save downloaded files:

  **0** - save in current game directory
  **1** - save in fs\_basegame (baseq3) directory

* * *

**Built-in URL-filter**

There is ability to launch Quake3 1.32e client directly from your browser by just clicking on URL that contains **q3a://** instead of usual **http://** or **ftp://** protocol headers

What you need to do:

* copy **q3url\_add.cmd**/**q3url\_rem.cmd** in quake3e.exe directory
* run **q3url\_add.cmd** if you want to add protocol binding or **q3url\_add.cmd** if you want to remove it
* type in your browser [**q3a://**127.0.0.1](q3a://127.0.0.1) and follow it - if you see quake3e launching and trying to connect then everything is fine

* * *

**Condifional shader stages**

Optional "if"-"elif"-"else" keywords can be used to control which shaders stages can be loaded depending from some cvar values.
For example, old shader:

console
{
 nopicmip
 nomipmaps
 {
  map image1.tga
  blendFunc blend
  tcmod scale 1 1
  }
}

New shader:

console
{
 nopicmip
 nomipmaps
 if ( $r\_vertexLight == 1 && $r\_dynamicLight )
 {
  map image1.tga
  blendFunc blend
  tcmod scale 1 1
 }
 else
 {
  map image2.tga
  blendFunc add
  tcmod scale 1 1
 }
}

lvalue-only conditions are supported, count of conditions inside if()/elif() is not limited

* * *

**Redurect captured video to ffmpeg input pipe**

In order to use this functionality you need to install ffmpeg package (on linux) or put ffmpeg binary near quake3e executable (on windows).

Use **\\cl\_aviPipeFormat** to control encoder parameters passed to ffmpeg, see ffmpeg documentation for details, default value is set according to YouTube recommendations:

\-preset medium -crf 23 -c:v libx264 -flags +cgop -pix\_fmt yuvj420p -bf 2 -c:a aac -strict -2 -b:a 160k -movflags faststart

If you need higher bitrate - decrease **\-crf** parameter, if you need better compression at cost of cpu time - set **\-preset** to _slow_ or _slower_.

And since ffmpeg can utilize all available CPU cores for faster encoding - make sure you have **\\com\_affinityMask** set to 0.

* * *

**Arbitrary resolution rendering**

Use **\\r\_renderWidth** and **\\r\_renderHeight** cvars to set persistant rendering resolution, i.e. game frame will be rendered at this resolution and later upscaled/downscaled to window size set by either **\\r\_mode** or **\\r\_modeFullscreen** cvars.
Cvar **\\r\_renderScale** controls upscale/downscale behavior:

* 0 - disabled
* 1 - nearest filtering, stretch to full size
* 2 - nearest filtering, preserve aspect ratio (black bars on sides)
* 3 - linear filtering, stretch to full size
* 4 - linear filtering, preserve aspect ratio (black bars on sides)

It may be useful if you want to render and record 4k+ video on HD display or if you're preferring playing at low resolution but your monitor or GPU driver can't set video|scaling mode properly.

* * *

**High-Quality Bloom**

Requires **[\\r\_fbo 1](#r_fbo)**, available operation modes via **\\r\_bloom** cvar:

* 0 - disabled
* 1 - enabled
* 2 - enabled + applies to 2D/HUD elements too

**\\r\_bloom\_threshold** - color level to extract to bloom texture, default is 0.6

**\\r\_bloom\_threshold\_mode** - color extraction mode:

* 0 - (r|g|b) >= threshold
* 1 - (r+g+b)/3 >= threshold
* 2 - luma(r,g,b) >= threshold

**\\r\_bloom\_modulate** - modulate extracted color:

* 0 - off (color=color, i.e. no changes)
* 1 - by itself (color=color\*color)
* 2 - by intensity (color=color\*luma(color))

**\\r\_bloom\_intensity** - final bloom blend factor, default is 0.5

**\\r\_bloom\_passes** - count of downsampled passes (framebuffers) to blend on final bloom image, default is 5

**\\r\_bloom\_blend\_base** - 0-based, topmost downsampled framebuffer to use for final image, high values can be used for stronger haze effect, results in overall weaker intensity

**\\r\_bloom\_filter\_size** - filter size of Gaussian Blur effect for each pass, bigger filter size means stronger and wider blur, lower value are faster, default is 6

**\\r\_bloom\_reflection** - bloom lens reflection effect, value is an intensity factor of the effect, negative value means blend only reflection and skip main bloom texture

* * *

End Of Document

* * *
