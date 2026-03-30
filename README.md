# companion-module-vsthost-defeedback

A [Bitfocus Companion](https://bitfocus.io/companion) module for controlling the **Defeedback** VST plugin running inside [Hermann Seib's VSTHost](http://www.hermannseib.com/english/vsthost.htm) via OSC over UDP.

---

## Requirements

- Bitfocus Companion 3.x
- VSTHost with the Defeedback plugin loaded and OSC enabled
- Both machines (or the same machine) reachable over UDP

---

## Configuration

| Setting | Default | Description |
|---|---|---|
| **Host** | `127.0.0.1` | IP address of the machine running VSTHost |
| **Port** | `7701` | UDP port VSTHost is listening on for OSC messages |
| **Local Port** | `7702` | UDP port this module binds to for receiving OSC replies |
| **Refresh Interval** | `30` | How often (in seconds) to automatically re-poll plugin states. Set to `0` to disable. |
| **Scan Min** | `2` | Lowest VSTHost plugin slot index to scan for Defeedback instances |
| **Scan Max** | `20` | Highest VSTHost plugin slot index to scan (inclusive) |

The module auto-discovers Defeedback instances on startup by scanning plugin slots `[Scan Min, Scan Max]` and checking the display name for the string `"defeedback"` (case-insensitive). Only matching slots are tracked and exposed as controls.

---

## Actions

### Strength

| Action | Description |
|---|---|
| **Set Defeedback Strength** | Set the strength of a specific Defeedback instance to an exact percentage (0–100%). |
| **Strength Up** | Increase strength by 1%. |
| **Strength Down** | Decrease strength by 1%. |

### Mute & Bypass

| Action | Description |
|---|---|
| **Set Defeedback Mute** | Set mute state to On, Off, or Toggle. |
| **Set Defeedback Bypass** | Set bypass state to On, Off, or Toggle. |

### Engine

| Action | Description |
|---|---|
| **Set Engine Run State** | Set the engine run state to On, Off, or Toggle. |

### Utility

| Action | Description |
|---|---|
| **Re-scan Plugins** | Re-query VSTHost for plugin count and re-discover all Defeedback instances. Useful if plugins are loaded or reordered at runtime. |
| **Refresh Plugin States** | Re-poll strength, mute, and bypass for all known Defeedback instances. |
| **Fetch Single Plugin State** | Poll the state of one specific plugin slot on demand. |
| **Send Custom OSC Message** | Send a raw OSC message to VSTHost. Provide the address and arguments as a JSON array (e.g. `[{"type":"i","value":1}]`). |

---

## Feedbacks

| Feedback | Trigger | Default Style |
|---|---|---|
| **Defeedback Muted** | True when the selected plugin instance is muted | Orange background |
| **Defeedback Bypassed** | True when the selected plugin instance is bypassed | Blue background |
| **Engine Running** | True when the VSTHost engine is running | Green background |

---

## Variables

### Global

| Variable | Description |
|---|---|
| `$(module:connection)` | `Connected` or `Disconnected` |
| `$(module:plugin_count)` | Number of discovered Defeedback instances |
| `$(module:engine_running)` | `Running` or `Stopped` |

### Per Plugin

These variables are created dynamically for each discovered Defeedback instance, where `N` is the VSTHost plugin slot index.

| Variable | Description |
|---|---|
| `$(module:plugin_N_name)` | Display name reported by VSTHost |
| `$(module:plugin_N_strength)` | Strength value as an integer (0–100) |
| `$(module:plugin_N_muted)` | `Yes` or `No` |
| `$(module:plugin_N_bypassed)` | `Yes` or `No` |

---

## How It Works

Communication uses OSC over UDP. The module sends OSC messages to VSTHost and listens for replies.

State changes are applied optimistically (the UI updates immediately) and then confirmed or corrected when VSTHost responds. Pending responses time out after 5 seconds. The optional refresh interval keeps state in sync if VSTHost is modified outside of Companion.
