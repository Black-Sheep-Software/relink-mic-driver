# This is a fork

This repository is Black Sheep Software's fork of
[VirtualDrivers/Virtual-Audio-Driver](https://github.com/VirtualDrivers/Virtual-Audio-Driver)
by MikeTheTech, maintained for
[ReLink](https://github.com/Black-Sheep-Software/ReLink)'s controller-microphone
relay: a Windows virtual microphone (and speaker) device that ReLink's Windows
client can feed decoded controller audio into, so it appears as a normal
selectable microphone to any app (Discord, a game's voice chat, Windows' own
sound settings).

Forked 24 November 2026, under the upstream project's own MIT license (plus
MS-PL for the Microsoft Sysvad sample code it is built on - see
`THIRD_PARTY_NOTICES.md`), full commit history preserved. See `LICENSE` for
the exact terms; this notice does not change them.

Why fork rather than depend on upstream directly: bundling a kernel-mode
driver with a shipped product needs a signed, stable, versioned release to
point users at - upstream has active development but no tagged release yet.
Maintaining our own fork lets ReLink pin to a known-good, independently
signed build, and keep this driver no more complex than what the relay
actually needs.

## Status (24 November 2026)

Builds for real: `VAD-compile.yml` produces a genuine, verified x64
`VirtualAudioDriver.sys` + a properly version-stamped `.inf` + a real
(unsigned) `.cat`, downloaded and checked directly (`file` confirms a
real PE32+ native binary, not a placeholder). Getting there took six
real, CI-verified fixes to the upstream build pipeline - see the commit
history for the exact sequence (chocolatey VS-workload conflicts,
missing WDK headers, ARM64 tooling gaps, stampinf.exe missing from
PATH, then found at the wrong architecture).

User-visible strings are rebranded (provider/manufacturer, device
description, and the mic/speaker endpoint friendly names - the mic one
now reads "ReLink Controller Microphone", matching the Linux client's
own naming for the same concept) - deliberately *not* the many
internal section names, hardware IDs or GUIDs elsewhere in the INF/C++
source, which carry real interdependency risk for no user-visible gain.

ARM64 is dropped from the build matrix entirely (ReLink itself only
targets x64 today - see its own docs/windows.md) rather than fighting
upstream's own known-fragile ARM64 WDK tooling gap.

Still ahead, and still real work:
- **Signing.** Validation/signing is currently bypassed
  (`SignMode=Off`) to get a real compile proven first - this build is
  unsigned and cannot install on a default Windows system as-is. No
  SignPath project has been applied for yet
  (`SIGNPATH_API_TOKEN`/`SIGNPATH_ORG_ID`/`SIGNPATH_PROJECT_SLUG`,
  needed by `VAD-compile-and-sign.yml`, whose automatic triggers stay
  disabled until that exists).
- **Feeding it real audio.** Nothing yet renders decoded controller
  audio into this device - that's ReLink's own Windows client code
  (a WASAPI render call to the virtual speaker side), not this repo.
