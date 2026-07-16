# DeathCam

**Every Minecraft death becomes an MP4 in your Discord — automatically.**
A chase camera follows the victim through their final minute, the game audio and
their real voice-chat screams are mixed in, and the video is posted to the death
channel. Nobody touches anything.

Built as the video pipeline for **[PermaDeath](https://github.com/TVTvirus/PermaDeath)**,
but works with any server that records per-player replays with
[ServerReplay](https://modrinth.com/mod/server-replay) (Flashback format).

## How it works

```
death → ServerReplay cuts the victim's replay → queue on the server host
      → a worker PC (with a real GPU) downloads it
      → launches a dedicated Minecraft instance with Flashback + the DeathCam mod
      → the mod opens the replay, plants a tracking camera on the victim,
        exports the last 62 seconds to MP4 and closes the game
      → ffmpeg mixes the death voice clip on top (from PermaDeath's voice recorder),
        normalizes to a web-standard MP4 under 9MB (plays inline in Discord)
      → uploaded back; the bot posts it
```

Render time: ~1 minute per death on a mid-range GPU (hardware encoder).

## Components

| Folder | What it is |
|---|---|
| `mod/` | Fabric **client** mod that automates Flashback: opens a replay from a job file, sets up a chase camera (two `TrackEntityKeyframe`s), starts the export and exits. |
| `worker/` | Bash worker + systemd user timer for the render PC: syncs pending replays over SSH, launches the render instance (dedicated Prism profile), post-processes with ffmpeg and uploads. |

## Setup (render PC)

1. Build `mod/` (`JAVA_HOME=<jdk25> ./gradlew build`; download the
   [Flashback](https://modrinth.com/mod/flashback) jar into `mod/libs/` first — its
   license does not allow redistribution).
2. Create a dedicated Prism Launcher instance with Fabric + Fabric API + Flashback +
   this mod, and a dedicated Prism data profile (`-d`) so it never collides with the
   Prism you play on.
3. Adjust hosts/paths at the top of `worker/gaturro-render-worker`, install the
   systemd user units, enable the timer.
4. The worker defers while you are playing (it costs GPU); `--force` overrides.

Hard-earned notes: Prism ignores `--launch` while initializing (launch twice), wait for
the previous render's java to fully die between jobs, always re-encode the export with
`-movflags +faststart` or Discord shows a player that never plays, and never point the
camera offsets at (0,0,0) unless you enjoy staring at the inside of a skull.

---

## En español

Cada muerte de Minecraft se convierte sola en un MP4 en tu Discord: cámara que persigue
a la víctima en su último minuto, con el audio del juego y sus gritos reales del chat de
voz mezclados. Es el pipeline de video de
**[PermaDeath](https://github.com/TVTvirus/PermaDeath)**; funciona con cualquier server
que grabe replays por jugador con ServerReplay. Render: ~1 minuto por muerte en una GPU
normal. Ver arriba para el montaje.
