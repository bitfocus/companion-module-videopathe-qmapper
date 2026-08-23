# companion-module-videopathe-qmapper

A [Bitfocus Companion](https://bitfocus.io/companion) module to control **QMapper**, the video mapping &
playback application for Android, over its local HTTP API.

QMapper is available for free on [videopathe.com](https://videopathe.com).

See [HELP.md](./companion/HELP.md) for the in-Companion help page, and [LICENSE](./LICENSE).

## Scope

The module targets the HTTP API exposed by QMapper on port `2226`. It uses a **polling** model: a single
`GET /api/status` snapshot per tick, completed by `/api/playlist/status`, `/api/warp`, `/api/blackout` and
`/api/language` (each of these is optional — a failure on one of them does not drop the connection).

The default poll interval is `400 ms`, which is fast enough to drive an animated progress bar; the module
also interpolates the playback position between two polls so the bar keeps moving smoothly.

It covers:

- playlist transport, direct index playback, per-item enable/disable, reload and playlist settings
- active source switching (Playlist / NDI / OMT / SRT / RTSP / USB) with fade or cut transition
- disconnection of the network inputs (RTSP, USB, WebRTC)
- blackout (toggle / on / off), with a pulsing button feedback
- warp & mapping: grid display, main output visibility, per-layer visibility, colour correction
- inter-device sync mode (Off / Master / Slave) and the MQTT transport toggle
- UI language switching
- 39 Companion variables covering playback, mapping and device system info
- 16 feedbacks, including two animated ones (progress bar, pulsing blackout)
- ready-made presets grouped by Playlist, Now playing, Sources, Output, Warp, Warp layers, Sync, App and Readouts

## Requirements

- QMapper running on the Android device, with its web server active
- The Android device and the Companion machine on the same network
- Companion 3.x

## Setup

1. Start QMapper on the Android device.
2. Note the device IP address (QMapper shows it in its interface / system info).
3. In Companion, add a **Videopathe: QMapper** connection and fill in:

| Field                                      | Default     | Description                                                                |
| ------------------------------------------ | ----------- | -------------------------------------------------------------------------- |
| **QMapper host / IP**                      | `127.0.0.1` | IP address of the Android device running QMapper, e.g. `192.168.1.50`      |
| **Port**                                   | `2226`      | QMapper HTTP API port                                                      |
| **Poll interval (ms)**                     | `400`       | Feedback refresh rate. Lower = smoother progress bar, more network traffic |
| **Animate progress bar & blackout button** | on          | Enables the moving progress bar and the pulsing blackout button            |

4. Save, and confirm the connection reaches the `ok` status.
5. Drag presets from the module onto your buttons.

## Actions

| Action                                          | Options                                                | Notes                                                                                                                           |
| ----------------------------------------------- | ------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------- |
| General: Refresh state now                      | —                                                      | Forces an immediate poll                                                                                                        |
| Playlist: Play / Pause / Stop / Next / Previous | —                                                      | Transport                                                                                                                       |
| Playlist: Play item at index                    | index (0-based)                                        |                                                                                                                                 |
| Playlist: Toggle item enabled at index          | index (0-based)                                        | Enables/disables an item without editing the playlist                                                                           |
| Playlist: Reload                                | —                                                      |                                                                                                                                 |
| Playlist: Settings                              | repeat mode, auto restart, source transition, duration | Repeat: none / one / all                                                                                                        |
| Source: Switch active source                    | source, transition, fade duration, USB device key      | Sources: Playlist, NDI, OMT, SRT, RTSP (last URL), USB capture. The USB device key is optional — blank opens the default device |
| Source: Disconnect a network input              | RTSP / USB / WebRTC                                    | Returns to the playlist source                                                                                                  |
| Output: Blackout                                | toggle / on / off                                      |                                                                                                                                 |
| Warp: Show mapping grid                         | toggle / on / off                                      |                                                                                                                                 |
| Warp: Main output visibility                    | toggle / on / off                                      |                                                                                                                                 |
| Warp: Layer visibility                          | layer, toggle / on / off                               | The layer list is rebuilt automatically from QMapper; custom names are accepted                                                 |
| Warp: Colour correction                         | brightness, contrast, red, green, blue                 | Brightness `-1…1`, the others `0…2`                                                                                             |
| Sync: Set sync mode                             | Off / Master / Slave                                   | Multi-device synchronisation                                                                                                    |
| MQTT: Enable / disable                          | toggle / enable / disable                              | The MQTT transport, separate from the sync mode above                                                                           |
| App: Language                                   | Français / English                                     |                                                                                                                                 |

WebRTC is a receiver started by an incoming offer, so it cannot be _switched to_ from a button — only
observed through a feedback and disconnected.

## Feedbacks

| Feedback                             | Type     | Description                                                                           |
| ------------------------------------ | -------- | ------------------------------------------------------------------------------------- |
| Connection is OK                     | boolean  | The module is reaching QMapper                                                        |
| Playlist is playing                  | boolean  |                                                                                       |
| Playlist is stopped / paused         | boolean  |                                                                                       |
| Shuffle is enabled                   | boolean  |                                                                                       |
| Current playlist index matches       | boolean  | Compares against a 0-based index                                                      |
| Blackout is active                   | boolean  |                                                                                       |
| **Blackout — pulsing**               | advanced | The button pulses red while blackout is active                                        |
| Mapping grid is shown                | boolean  |                                                                                       |
| Warp: Main output is visible         | boolean  |                                                                                       |
| Warp: Layer is visible               | boolean  | Per named layer                                                                       |
| Active source matches                | boolean  | Playlist / NDI / OMT / SRT / RTSP / WebRTC / USB                                      |
| Sync mode matches                    | boolean  | Off / Master / Slave                                                                  |
| MQTT is enabled                      | boolean  |                                                                                       |
| UI language matches                  | boolean  |                                                                                       |
| Remaining time comparison            | boolean  | `<`, `<=`, `>=`, `>` against a number of seconds — useful for an end-of-media warning |
| **Playlist progress bar (animated)** | advanced | Draws a progress bar directly on the button, interpolated between polls               |

## Variables

All variables are prefixed with `$(videopathe-qmapper:…)`.

**Connection & device** — `connection_status`, `server_url`, `device_name`, `device_ip`, `app_version`

**Playback** — `playing`, `shuffle`, `position`, `duration`, `remaining`, `progress_percent`

**Current media** — `current_filename`, `current_index`, `enabled_index`, `enabled_count`, `source_mode`,
`source_mode_label`

**Output & mapping** — `blackout`, `grid_visible`, `main_visible`, `layer_count`, `layer_names`, `sync_mode`,
`sync_mode_label`, `mqtt_enabled`, `language`

**Device system info** — `cpu_usage`, `fps`, `cpu_temp`, `ram_used`, `ram_total`, `resolution`,
`refresh_rate`, `active_layers`, `loaded_media`, `connection_type`

## Presets

Ready-made buttons, grouped in the Presets tab. Every preset also carries a "connection lost" feedback that
turns the button dark red when QMapper is unreachable.

- **Playlist** — Play, Pause, Stop, Previous, Next, Shuffle indicator
- **Now playing** — media name + index, animated progress bar with elapsed/remaining time, percent readout
- **Sources** — one switch button per source (Playlist, NDI, OMT, SRT, RTSP, USB), a WebRTC indicator that
  doubles as a disconnect button, and an RTSP disconnect button
- **Output** — pulsing blackout toggle
- **Warp** — mapping grid toggle, main output visibility
- **Warp layers** — one toggle button per named layer, rebuilt whenever QMapper's layer set changes
- **Sync** — Sync Off / Master / Slave, MQTT toggle
- **App** — FR / EN language buttons
- **Readouts** — connection status + active source, system readout (CPU / FPS / temperature)

## Development

```sh
corepack enable
yarn install
yarn build      # compiles TypeScript to dist/
yarn dev        # watch mode — recommended while testing with Companion
yarn lint       # eslint + prettier
yarn format     # applies prettier
yarn package    # builds a .tgz for Companion
```

To test in Companion developer mode, set Companion's **Developer modules path** to the _parent_ folder
containing `companion-module-videopathe-qmapper` — not to the module folder itself — then add a
**Videopathe: QMapper** connection. In watch mode, Companion reloads the module when the files are rebuilt.

## API reference

- `GET /api/status` — aggregated state snapshot (playlist, current media, warp, control, system)
- `GET /api/playlist/status` — authoritative playback position / duration
- `GET /api/warp` — mapping layers and their visibility
- `GET /api/blackout`, `GET /api/language` — extra state
- `POST /api/playlist/*` — transport, index playback, item toggle, reload, settings
- `POST /api/control/source-switch`, `POST /api/control/sync-mode`, `POST /api/control/settings`
- `POST /api/warp/show-grid`, `/api/warp/visibility/main`, `/api/warp/visibility/layer`, `/api/warp/color`
- `POST /api/blackout`, `POST /api/language`, `POST /api/{rtsp,usb,webrtc}-input/disconnect`

QMapper exposes an interactive API documentation inside its own web interface (`http://<device-ip>:2226`).

## Troubleshooting

- **Connection failure** — check the IP address, that both devices are on the same network, and that QMapper
  is in the foreground on the Android device.
- **Choppy progress bar** — lower the poll interval (e.g. `250 ms`) and make sure _Animate_ is enabled.
- **A warp layer is missing from the dropdown** — the list comes from QMapper and only contains _named_
  layers. Rename the layer in QMapper, or type the name manually (the dropdown accepts custom values).

## License

MIT
