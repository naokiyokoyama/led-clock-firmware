# led-clock-firmware

Firmware artifacts for the LED clocks. **Public on purpose** — the clocks fetch
`manifest.json` with no credentials, so there is no token to extract from a
device. Source lives in the private `home_assistant` repo; only built binaries
and this manifest are published.

## Published images contain NO WiFi credentials

Every artifact here is built with `-DNO_BAKED_CREDS`. This is not optional: a
compiled-in SSID/password is recoverable from a `.bin` with `strings`, and this
repo is world-readable.

An already-provisioned clock does not need baked-in credentials — they live in
NVS, which an OTA does not touch. Credentials are only needed for a first USB
flash, and those builds are never published. `release.sh` enforces this by
scanning the built image for every string in the secrets headers and refusing to
publish on a match.

History was rewritten on 2026-08-08 to purge earlier binaries that did contain
credentials.

## How a clock updates

Every 6 hours (and on demand from its settings page) a clock fetches
`manifest.json`, looks up the key matching its own `CLOCK_NAME`, and installs the
image if the manifest version is higher than its own. The MD5 in the manifest —
not any server header — is what the device verifies before committing.

A unit only ever reads its own key, so adding a channel cannot cause an existing
clock to pull the wrong image.

| Channel | Units |
|---|---|
| `momclock` | 2 (NYC) |
| `led-clock-bedroom` | 1 |
| `led-clock-bathroom` | 1 |
| `friendclock` | 1 |

## Publishing

`./release.sh <version> [channel]` in `led-clock-large-blue/`, then commit and
push here. Verify every manifest MD5 against the actual file first. Roll out to
one unit of a multi-unit channel, wait a day, then the rest.
