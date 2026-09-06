# Idle Power — Omarchy Shell Plugin

Idle Power is an [Omarchy](https://omarchy.org/) service plugin that provides
separate deadlines for the screensaver, monitor power-off, and system suspend
without adding an idle lock deadline.

It is derived from Omarchy's `omarchy.idle` service and keeps its inhibitor-aware
idle monitoring, screensaver lifecycle handling, and Stay Awake integration.

## Features

- Configurable screensaver, DPMS monitor-off, and suspend deadlines
- No idle lock; Omarchy can still lock normally when the system suspends
- Stops the screensaver when displays power off, without cancelling suspend
- Restores displays on activity after DPMS power-off
- Respects system idle inhibitors and Omarchy's Stay Awake toggle
- Runtime status through the `idle-power` shell IPC target

## Installation

Install and enable the plugin:

```bash
omarchy plugin add https://github.com/pomartel/idle-power.git --enable
```

The plugin ID is `idle-power`. It replaces `omarchy.idle`, so the
first-party service should be disabled in `~/.config/omarchy/shell.json`:

```json
{
  "idle": {
    "screensaver": 300,
    "monitorOff": 1200,
    "suspend": 86400
  },
  "plugins": [
    { "id": "idle-power" }
  ],
  "disabledPlugins": [
    "omarchy.idle"
  ]
}
```

Timeouts are measured in seconds from the beginning of user inactivity. In the
example above, the screensaver starts after 5 minutes, displays power off after
20 minutes, and the computer suspends after 24 hours.
Set `idle.suspend` to `false` to disable automatic suspend while keeping the
screensaver and monitor power-off timers enabled.

When displays power off, the plugin stops the screensaver using the same process
cleanup as Omarchy's lock transition. That expected close does not cancel the
idle cycle. A separate activity monitor wakes the displays when the user returns.

## Stay Awake

The standard Omarchy command continues to control idle behavior:

```bash
omarchy toggle idle stay-awake
omarchy toggle idle allow-idle
omarchy toggle idle status
```

## Status and troubleshooting

Inspect the live service state with:

```bash
omarchy-shell idle-power status | jq
```

Update an installed copy with:

```bash
omarchy plugin update idle-power
```

## Development

Validate the plugin with:

```bash
omarchy plugin validate .
qmllint -I /usr/share/omarchy/shell Service.qml
node --check IdleModel.js
```

## Upstream and license

Idle Power is derived from the `omarchy.idle` service distributed with
[basecamp/omarchy](https://github.com/basecamp/omarchy). See [UPSTREAM.md](UPSTREAM.md)
for details.

The plugin is released under the [MIT License](LICENSE).
