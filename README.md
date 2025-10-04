# Quake3e Urban Terror

This is a modern Quake III Arena engine aimed to be fast, secure and compatible with all existing Q3A mods. It is based on last non-SDL source dump of [ioquake3](https://github.com/ioquake/ioq3) with latest upstream fixes applied.

This project is a continuation of innactive [urbanterror-slim](https://github.com/omg-urt/urbanterror-slim), kept fully up-to-date with all relevent changes from [Quake3e](https://github.com/ec-/Quake3e).

Go to [rorgoroth/mingw-cmake-env/releases](https://github.com/rorgoroth/mingw-cmake-env/releases) to download latest Windows binaries.

*This repository does not contain any game content so in order to play you must copy the resulting binaries into your existing Urban Terror installation.*

- [Quake3e Urban Terror](#quake3e-urban-terror)
  - [Key features](#key-features)
  - [Vulkan renderer](#vulkan-renderer)
  - [OpenGL renderer](#opengl-renderer)
  - [OpenGL2 renderer](#opengl2-renderer)
  - [Common changes/additions](#common-changesadditions)
  - [Client-specific changes/additions](#client-specific-changesadditions)
  - [Server-specific changes/additions](#server-specific-changesadditions)
  - [More in-depth command examples](#more-in-depth-command-examples)
    - [\\in\_minimize](#in_minimize)
    - [Fast client downloads](#fast-client-downloads)
    - [Fast client-initiated downloads](#fast-client-initiated-downloads)
    - [Conditional shader stages](#conditional-shader-stages)
    - [Redirect captured video to ffmpeg input pipe](#redirect-captured-video-to-ffmpeg-input-pipe)
    - [Arbitrary resolution rendering](#arbitrary-resolution-rendering)
    - [High-Quality Bloom](#high-quality-bloom)



## Key features

* Optimized OpenGL renderer
* Optimized Vulkan renderer
* Raw mouse input support, enabled automatically instead of DirectInput (`\in_mouse 1`) if available
* Unlagged mouse events processing, can be reverted by setting `in_lagged 1`
* `\in_minimize` - hotkey for minimize/restore main window (win32-only, direct replacement for Q3Minimizer)
* `\video-pipe` - to use external ffmpeg binary as an encoder for better quality and smaller output files
* Significally reworked QVM (Quake Virtual Machine)
* Improved server-side DoS protection, much reduced memory usage
* Raised filesystem limits (up to 20,000 maps can be handled in a single directory)
* Reworked Zone memory allocator, no more out-of-memory errors
* Non-intrusive support for SDL2 backend (video, audio, input), selectable at compile time
* Tons of bug fixes and other improvements

## Vulkan renderer

Based on [Quake-III-Arena-Kenny-Edition](https://github.com/kennyalive/Quake-III-Arena-Kenny-Edition) with many additions:

* High-quality per-pixel dynamic lighting
* Very fast flares (`\r_flares 1`)
* Anisotropic filtering (`\r_ext_texture_filter_anisotropic`)
* Greatly reduced API overhead (call/dispatch ratio)
* Flexible vertex buffer memory management to allow loading huge maps
* Multiple command buffers to reduce processing bottlenecks
* [Reversed depth buffer](https://developer.nvidia.com/content/depth-precision-visualized) to eliminate z-fighting on big maps
* Merged lightmaps (atlases)
* Multitexturing optimizations
* Static world surfaces cached in VBO (`\r_vbo 1`)
* Useful debug markers for tools like [RenderDoc](https://renderdoc.org/)
* Fixed framebuffer corruption on some Intel iGPUs
* Offscreen rendering, enabled with `\r_fbo 1`, all following requires it enabled:
* `screenMap` texture rendering - to create realistic environment reflections
* Multisample anti-aliasing (`\r_ext_multisample`)
* Supersample anti-aliasing (`\r_ext_supersample`)
* Per-window gamma-correction which is important for screen-capture tools like OBS
* You can minimize game window any time during `\video` or `\video-pipe` recording
* High dynamic range render targets (`\r_hdr 1`) to avoid color banding
* Bloom post-processing effect
* Arbitrary resolution rendering
* Greyscale mode

In general, not counting offscreen rendering features you might expect from 10% to 200%+ FPS increase comparing to KE's original version. Highly recommended to use on modern systems

## OpenGL renderer

Based on classic OpenGL renderers from [idq3](https://github.com/id-Software/Quake-III-Arena)/[ioquake3](https://github.com/ioquake/ioq3)/[cnq3](https://bitbucket.org/CPMADevs/cnq3)/[openarena](https://github.com/OpenArena/engine), features:

* OpenGL 1.1 compatible, uses features from newer versions whenever available
* High-quality per-pixel dynamic lighting, can be triggered by `\r_dlightMode` cvar
* Merged lightmaps (atlases)
* Static world surfaces cached in VBO (`\r_vbo 1`)
* All set of offscreen rendering features mentioned in Vulkan renderer, plus:
* Bloom reflection post-processing effect

Performance is usually greater or equal to other opengl1 renderers

## OpenGL2 renderer

Original ioquake3 renderer, performance is very poor on non-nvidia systems, unmaintained

## Common changes/additions

* A lot of security, performance and bug fixes
* Much improved autocompletion (map, demo, exec and other commands), in-game `\callvote` argument autocompletion
* `\com_affinityMask` - bind Quake3e process to bitmask-specified CPU core(s)
* Raized filesystem limits, much faster startup with 1000+ pk3 files in use, level restart times were also reduced as well
* `\fs_locked 0|1` - Keep opened pk3 files locked or not, removes pk3 file limit when unlocked

## Client-specific changes/additions

* Raw mouse input support, enabled automatically instead of DirectInput (`\in_mouse 1`) on Windows XP and newer windows operating systems
* Unlagged mouse processing, can be reverted by setting `in_lagged 1`
* MOUSE4 and MOUSE5 works in `\in_mouse -1` mode
* `\minimize` in-game command to minimize main window, can be used with binds/scripting
* [`\in_minimize`](#in_minimize) - hotkey for minimize/restore main window (direct replacement for Q3Minimizer)
* `\in_forceCharset 0|1|2`
  * `0` - Disabled
  * `1` - Try to translate non-ASCII chars in keyboard input
  * `2` - Force EN/US keyboard layout
* `\in_nograb 0|1` Do not capture mouse in game, may be useful during online streaming.
* `\s_muteWhenUnfocused 0|1`
* `\s_muteWhenMinimized 0|1`
* `\s_device` - Linux-only, specified sound device to use with ALSA, enter aplay -L in your shell to see all available options
* `\screenshotBMP` and `\screenshotBMP clipboard` commands
* Hardcoded PrintScreen key - For "\screenshotBMP clipboard"
* Hardcoded Shift+PrintScreen - For "\screenshotBMP"
* `\com_maxfpsUnfocused` - Will save cpu when inactive,set to your desktop refresh rate, for example
* `\com_skipIdLogo 0|1`- Skip playing idlogo movie at startup
* `\com_yieldCPU <milliseconds>` - Try to sleep specified amout of time between rendered frames when game is active, this will greatly reduce CPU load, use `0` only if you're experiencing some lags (also it is usually reduces performance on integrated graphics because CPU steals GPU's power budget)
* `\r_defaultImage <filename>|#rgb|#rrggbb` - Replace default (missing) image texture by either exact file or solid #rgb|#rrggbb background color
* `\r_vbo 0|1` - Use Vertex Buffer Objects to cache static map geometry, may improve FPS on modern GPUs, increases hunk memory usage by 15-30MB (map-dependent)

* `\r_fbo 0|1` - Use framebuffer objects, enables gamma correction in windowed mode and allows arbitrary size (i.e. greater than logical desktop resolution) screenshot/video capture, required for bloom, hdr rendering, anti-aliasing, greyscale effects, OpenGL 3.0+ required
* `\r_hdr \-1|0|1` - Select texture format for framebuffer:
  * `-1` - 4-bit, for testing purposes, heavy color banding, might not work on all systems
  * `0` - 8 bit, default, moderate color banding with multi-stage shaders
  * `1` - 16 bit, enhanced blending precision, no color banding, might decrease performance on AMD/Intel GPUs

* `\r_bloom 0|1|2` - High-quality light bloom postprocessing effect ([\r_bloom](#r_bloom))
* `\r_dlightMode 0|1|2` - Dynamic light mode
  * `0` - VQ3 'fake' dynamic lights
  * `1` - New high-quality per-pixel dynamic lights, slightly faster than VQ3's on modern hardware
  * `2` - Same as 1 but applies to all MD3 models too
* `\r_modeFullscreen` - Dedicated mode string for fullscreen mode, set to `-2` to use desktop resolution, set empty to use `\r_mode` in all cases.
* `\r_nomip 0|1`- Apply picmip only on worldspawn textures
* `\r_neatsky 0|1` - nopicmip for skyboxes
* `\r_greyscale [0..1.0]` - Desaturates rendered frame, requires [`\r_fbo 1`](#r_fbo), can be changed on the fly.
* `\r_mapGreyScale [-1.0..1.0]` - Desaturate world map textures only, works independently from `\r_greyscale`, Negative values desaturates lightmaps only.
* `\r_ext_multisample 0|2|4|6|8` - Multi-sample anti-aliasing, requires [`\r_fbo 1`](#r_fbo), can be changed on the fly.
* `\r_ext_supersample 0|1` - Super-sample anti-aliasing, requires [`\r_fbo 1`](#r_fbo)
* `\r_noborder 0|1` - To draw game window without border, hold ALT to drag & drop it with opened console.
* `\r_noportals 0|1|2` - Disable in-game portals (1), and mirrors too (2)
* Negative `\r_overBrightBits` - Force hardware gamma in windowed mode, not actual with [`\r_fbo 1`](#r_fbo).
* [`\video-pipe`](#video-pipe) - Redirect captured video to ffmpeg input pipe and save encoded file as <filename>
* [`\r_renderWidth | \r_renderHeight`](#arr) - arbitrary resolution rendering, requires [`\r_fbo 1`](#r_fbo)
* `\cl_conColor [RRR GGG BBB AAA]` - Custom console color, non-archived, use `\seta` command to set archive flag and store in config
* `\cl_autoNudge 0|1` - Automatic time nudge that uses your average ping as the time nudge, values:
  * `0` - Use fixed `\cl_timeNudge`
  * `1` - Factor of median average ping to use as timenudge
* `\cl_mapAutoDownload 0|1` - automatic map download for play and demo playback (via automatic [`\dlmap`](#dlmap) call)
* Less spam in console (try to set `\developer 1` to see what important things you missing)
* Faster shader loading, tolerant to non-fatal errors
* Fast client downloads (http/ftp redirection)
* [`\download | \dlmap`](#dlmap) - Fast client-initiated downloads from specified map server
* You can use `\record` during `\demo playback`
* [Conditional shader stages](#condstages)
* Linear dynamic lights

## Server-specific changes/additions

* `\sv_levelTimeReset 0|1` - Reset or do not reset leveltime after new map loads, when enabled - fixes gfx for clients affected by "frameloss" bug, however it may be necessary disable in case of troubles with GTV
* `\sv_maxclientsPerIP` - Limit number of simultaneous connections from the same IP address
* Much improved DDoS protection
* `\sv_minPing | \sv_maxPing` were removed because of new much better client connection code
* Userinfo filtering system, see docs/filter.txt
* `rcon` Now is always available on dedicated servers
* `rconPassword2` - Hidden master rcon password that can be set only from command line, eg: `+set rconPassword2 "123456"` can be used to change/revoke compromised `rconPassword`
* Significally reduced memory usage for client slots.

## More in-depth command examples

### \in_minimize

cvar that specifies hotkey for fast minimize/restore main windows, set values in form:

* `\in_minimize "ctrl+z"`
* `\in_minimize "lshift+ralt+p"`

or so then do `\in_restart` to apply changes.

### Fast client downloads

Usually downloads in Q3 is slow because all downloaded data is requested/received from game server directly. So called \`download redirection' allows very fast downloads because game server just tells you from where you should download all required mods/maps etc. - it can be internet or local http/ftp server
So what you need:

set `sv_dlURL` cvar to download location, for example: `sets sv_dlURL "ftp://myftp.com/games/q3"`
Please note that your link should point on root game directory not particular mod or so.

Fill your download location with files. Please note that you should post ALL files that client may download, don't worry about pk3 overflows etc. on client side as client will download and use only required files

### Fast client-initiated downloads

You can easy download pk3 files you need from any ftp/http/etc resource via following commands:

* `\download filename.pk3`
* `\dlmap mapname.pk3`

`\dlmap` is the same as `\download` but also will check for map existence

`cl_dlURL` cvar must point to a download location, where (optional) `%1` pattern will be replaced by `\download|dlmap` command argument (filename/mapmane) and HTTP header may be analysed to obtain destination filename

For example:

1) If `cl_dlURL` is `<http://127.0.0.1>` and you type `\download promaps.pk3` -  The resulting download url will be `<http://127.0.0.1/promaps.pk3>`
2) If `cl_dlURL` is `<http://127.0.0.1/%1>` and you type `\dlmap dm17` - The resulting download url will be `<http://127.0.0.1/dm17>` Also in this case HTTP-header will be analysed and resulting filename may be changed

To stop download just specify '-' as argument:

  `\dlmap -`

`cl_dlDirectory` cvar specifies where to save downloaded files:

* `0` - save in current game directory
* `1` - save in fs_basegame (baseq3) directory

### Conditional shader stages

Optional "if"-"elif"-"else" keywords can be used to control which shaders stages can be loaded depending from some cvar values.
For example, old shader:

```
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
```

New shader:

```
console
{
 nopicmip
 nomipmaps
 if ( $r_vertexLight == 1 && $r_dynamicLight )
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
```

lvalue-only conditions are supported, count of conditions inside if()/elif() is not limited

### Redirect captured video to ffmpeg input pipe

In order to use this functionality you need to install ffmpeg package (on Linux) or put ffmpeg binary near quake3e executable (on Windows).

Use `\cl_aviPipeFormat` to control encoder parameters passed to ffmpeg, see [ffmpeg documentation](https://trac.ffmpeg.org/wiki/Encode/YouTube) for details, default value is set according to YouTube recommendations:

`-c:v libx264 -preset slow -crf 18 -c:a aac -b:a 192k -pix_fmt yuv420p`

If you need higher bitrate - decrease `-crf` parameter, if you need better compression at cost of cpu time - set `-preset` to *slow* or *slower*.

And since ffmpeg can utilize all available CPU cores for faster encoding - make sure you have `\com_affinityMask` set to 0.

### Arbitrary resolution rendering

Use `\r_renderWidth` and `\r_renderHeight` cvars to set persistent rendering resolution, i.e. game frame will be rendered at this resolution and later upscaled/downscaled to window size set by either `\r_mode` or `\r_modeFullscreen` cvars.
Cvar `\r_renderScale` controls upscale/downscale behaviour:

* `0` - disabled
* `1` - nearest filtering, stretch to full size
* `2` - nearest filtering, preserve aspect ratio (black bars on sides)
* `3` - linear filtering, stretch to full size
* `4` - linear filtering, preserve aspect ratio (black bars on sides)

It may be useful if you want to render and record 4k+ video on HD display or if you're preferring playing at low resolution but your monitor or GPU driver can't set video/scaling mode properly.

### High-Quality Bloom

Requires [`\r_fbo 1`](#r_fbo), available operation modes via `\r_bloom` cvar:

* `0` - disabled
* `1` - enabled
* `2` - enabled + applies to 2D/HUD elements too

`\r_bloom_threshold` - Colour level to extract to bloom texture, default is 0.6

`\r_bloom_threshold_mode` - Colour extraction mode:

* `0` - (r|g|b) >= threshold
* `1` - (r+g+b)/3 >= threshold
* `2` - luma(r,g,b) >= threshold

`\r_bloom_modulate` - modulate extracted color:

* `0` - off (color=color, i.e. no changes)
* `1` - by itself (color=color\*color)
* `2` - by intensity (color=color\*luma(color))

`\r_bloom_intensity` - final bloom blend factor, default is 0.5

`\r_bloom_passes` - count of downsampled passes (framebuffers) to blend on final bloom image, default is 5

`\r_bloom_blend_base` - 0-based, topmost downsampled framebuffer to use for final image, high values can be used for stronger haze effect, results in overall weaker intensity

`\r_bloom_filter_size` - filter size of Gaussian Blur effect for each pass, bigger filter size means stronger and wider blur, lower value are faster, default is 6

`\r_bloom_reflection` - bloom lens reflection effect, value is an intensity factor of the effect, negative value means blend only reflection and skip main bloom texture
