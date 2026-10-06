# Pomodoro Timer (riciolus fork)

A Pomodoro timer plugin for Noctalia.

> **Fork notice** — This is a fork of
> [`thepunkoff/pomodoro`](https://github.com/noctalia-dev/community-plugins/tree/main/pomodoro)
> v1.3.0 from the official community plugin catalog
> [`noctalia-dev/community-plugins`](https://github.com/noctalia-dev/community-plugins)
> (itself originally ported from the legacy v4
> [Pomodoro Timer](https://github.com/noctalia-dev/legacy-v4-plugins/tree/main/pomodoro)).
> All credit for the original plugin goes to **thepunkoff**. This fork only adds
> the changes listed below; the original MIT license
> ([`LICENCE.txt`](LICENCE.txt)) applies.

## What's changed in this fork

- **Adjust the time of the current session** — the panel has `−5m −1m +1m +5m`
  buttons under the clock. They change the total *and* the remaining time of the
  current stage (work or break), for the current session only, live — no need to
  open the settings. Totals are clamped to a minimum of 1 minute.
- **IPC control** — the timer can be driven from the command line (and therefore
  from compositor keybinds): see [IPC](#ipc).
- Plugin id renamed to `riciolus/pomodoro`, version `1.4.0`.

Everything else (sessions, cycles, auto-start, sounds, notifications, widget,
settings) is unchanged from upstream.

## Features

- **Sessions**: Configurable sessions based on the standard format (work - short break - long break). Durations can be configured in the settings.
- **Cycles**: Configurable number of (work - short break) cycles before a long break.
- **Auto-start**: Optionally auto-start breaks and/or work sessions.
- **Bar Widget**: Shows status and remaining time on the bar widget when the panel is closed.
- **Notifications**: Toast notification + alarm sound when work/break finishes.
- **Ticking Sound**: Optional tick/tock sounds each second while a work session is running.

## Plugin

| Field | Value |
| --- | --- |
| ID | `riciolus/pomodoro` |
| Entries | Bar widget: `widget`; panel: `panel`; service: `pomodoro` |

## Install

The plugin lives in the local drop-in plugin directory and hot-reloads when a
`.luau` file changes:

```
~/.local/share/noctalia/plugins/pomodoro/
```

Enable it in **Settings → Plugins** (or `noctalia msg plugins enable
riciolus/pomodoro`), then add the bar widget `Pomodoro Timer`. If the upstream
`thepunkoff/pomodoro` is also installed, disable it so only one copy runs.

## Usage

1. Enable plugin in settings
2. Add bar widget `Pomodoro Timer`
3. Widget appears on the bar, clicking it will toggle the panel.
4. In the panel, use `−5m −1m +1m +5m` to change the length of the current
   session; *Reset* restores the configured duration.

### Alarm Sound Configuration
When [`enable_sounds`](https://docs.noctalia.dev/noctalia/services/audio/) is set to `true` in Noctalia config (or enabled in the GUI with `Services -> Audio -> Shell Sounds`), the notification after completed session will pop up with Noctalia's default sound. If you want to use the bundled alarm sound, set this in Noctalia config:

```toml
[plugin_settings."riciolus/pomodoro"]
use-bundled-alarm-sound = true
```

or use `Use Bundled Alarm Sound` in the GUI plugin settings. Note that after enabling this the notification will play both the default sound and the bundled one at the same time. To silence the default sound use Noctalia's notification filtering:

```toml
[notification]
# ...
    [notification.filter.pomodoro]
    match = "noctalia"
    match_content = "Pomodoro Timer"
    play_sound = false
```

or use `Notifications -> Filtering` in the GUI.

To play the sound even when `enable_sounds` (`Shell Sounds`) is set to false (e.g. if you don't want to enable all the sounds, just pomodoro's), use:

```toml
[plugin_settings."riciolus/pomodoro"]
bypass-noctalia-sound-globals = true
```

or `Bypass Noctalia Sound Globals` in the GUI plugin settings. This will use the `ffplay` tool to play the sound directly, so make sure to have it installed.

### Ticking Sound Configuration
To play a ticking (tick/tock) sound each second while a **work** session is running, enable it in Noctalia config:

```toml
[plugin_settings."riciolus/pomodoro"]
enable-work-timer-sound = true
```

or use `Enable Work Timer Sound` in the GUI plugin settings. Break timers stay silent.

The ticking sound obeys the same global sound rules as the alarm: it plays through Noctalia's sound system when [`enable_sounds`](https://docs.noctalia.dev/noctalia/services/audio/) (`Shell Sounds`) is `true`. To play it even when `enable_sounds` is `false`, combine it with `bypass-noctalia-sound-globals` (which requires `ffplay`):

```toml
[plugin_settings."riciolus/pomodoro"]
enable-work-timer-sound = true
use-bundled-alarm-sound = true
bypass-noctalia-sound-globals = true
```

## Settings

| Setting | Type | Default | Description |
| --- | --- | --- | --- |
| `work-duration` | `int` | `25` | Duration of each work session in minutes. |
| `short-break-duration` | `int` | `5` | Duration of short breaks in minutes. |
| `long-break-duration` | `int` | `15` | Duration of long breaks in minutes. |
| `sessions-before-long-break` | `int` | `4` | Number of sessions before a long break (min=1). |
| `auto-start-work` | `bool` | `false` | Automatically start the work timer after a break. |
| `auto-start-breaks` | `bool` | `false` | Automatically start the break timer after a work session. |
| `enable-work-timer-sound` | `bool` | `false` | Play tick/tock sounds each second while a work session is running. Break timers stay silent. |
| `use-bundled-alarm-sound` | `bool` | `false` | Use the bundled alarm sound when work/break is over. |
| `bypass-noctalia-sound-globals` | `bool` | `false` | Ignore Noctalia's global sound settings and play bundled sounds directly via `ffplay`. |

Changing a duration applies from the next plugin reload; for a one-off change
to the running session use the panel buttons or the `adjust` IPC below.

## IPC

Open the panel:

```sh
noctalia msg panel-toggle riciolus/pomodoro:panel
```

Adjust the current session by a number of seconds (works while running or
paused; great for a keybind):

```sh
noctalia msg plugin riciolus/pomodoro:pomodoro all adjust 60    # +1 minute
noctalia msg plugin riciolus/pomodoro:pomodoro all adjust -60   # -1 minute
noctalia msg plugin riciolus/pomodoro:pomodoro all adjust 300   # +5 minutes
```

Notify the current stage, remaining time and session count:

```sh
noctalia msg plugin riciolus/pomodoro:pomodoro all status
```

## Requirements
- (optional) `ffplay` for playing bundled sounds independently from Noctalia's global sound settings.

## Licensing

This project is licensed under the MIT License (see `LICENCE.txt`, © thepunkoff
and contributors).

Additional assets:
- JetBrains Mono font, which is licensed separately under the SIL Open Font License 1.1 (OFL-1.1). See `THIRD_PARTY_LICENCES/OFL.txt` for the full license text.
- Alarm sound: `alarm.mp3` - Sourced from [Pixabay](https://pixabay.com/) (Royalty-free, [Pixabay Content License](https://pixabay.com/service/license-summary/))
- Tick/Tock sounds: `tick1-9.wav/tock1-9.wav` - Sourced from [Pixabay](https://pixabay.com/sound-effects/film-special-effects-clock-ticking-down-376897/), manually cut into parts. (Royalty-free, [Pixabay Content License](https://pixabay.com/service/license-summary/))
