# Stardew Valley on musl libc
A guide on how to run Stardew Valley (Steam version, but without steam) on musl libc!

## Note
- THIS IS NOT PIRACY GUIDE!
- This has only been tested on Alpine Linux Edge (September 2026) on x86_64, though I am sure
that it wouldn't take too much more work to run this emulated under aarch64.
- You need a Linux native Steam copy of the game

## Guide
- Step 1 - Obtain a Linux native Steam copy of the game<br>
I can't stop you from pirating it, but please don't (Unless regional pricing
make it prohibitively expensive, or you can't purchase it for other reasons).
Support indie games :).<br>
Basically, obtain the `steamapps/common/Stardew Valley` folder.
Personally, I did this by installing Steam from flatpak, downloading the game,
and copying the `~/.var/app/com.valvesoftware.Steam/data/Steam/steamapps/common/Stardew Valley/` folder.

- Step 2 - Patch the `Stardew Valley` ELF file to utilize gcompat and `ld-musl-x86_64.so.1` as the
interpreter.<br>
This is standard practice when patching anything pre-compiled away from glibc.<br>
Install `gcompat` and `patchelf`.
I copied the `Stardew Valley` executable to `StardewValleyMusl` so I could keep the original un-edited
file, and so I didn't have to deal with spaces. I encourage you to do the same.<br>
Then, to add the gcompat dependency and change the interpreter, I ran:<br>
`patchelf --set-interpreter /lib/ld-musl-x86_64.so.1 StardewValleyMusl`<br>
`patchelf --add-needed libgcompat.so.0 StardewValleyMusl`<br>

- Step 3 - Replace the native `libopenal.so.1`<br>
With the original version that is packaged with the game, it would segfault sometime into OpenAL's
init. Replacing it with a native version, of which I got from the `openal-soft` package (in Alpine's
repos) fixed the issue.<br>
Basically, by that I mean:<br>
`rm libopenal.so.1`<br>
`cp /usr/lib/libopenal.so.1 libopenal.so.1`<br>

- Step 4 - Run the game!<br>
Run the `StardewValleyMusl` executable to play! In the screenshot below, I ran it in LLDB, but you don't have to.
Just treat it as a normal executable now!
![Screenshot of the game running](/pictures/running.jpg)

## A note on Controller support
Controllers without Steam's support is kind of difficult. This game has no in-game settings for
selecting controllers, or even probing for them. I found that an XInput capable controller was
the only one that would work (an 8BitDo Pro 2 in my case).
I did test whether swapping out the `libSDL2-2.0.so.0` library would change anything, but it didn't.
Though, if you're desperate enough, you could patch the SDL2 source code to use a specific controller,
and it would probably work (Although, I am not even sure that this game utilizes SDL2 for controllers).

## Debugging
Personally I found LLDB to be more reliable than GDB. With GDB, I would randomly get crashes
with the error being `Thread 1 "StardewValleyMu" received signal SIG35, Real-time event 35.`,
and to be honest, I don't know how to debug that. LLDB worked anyway.
