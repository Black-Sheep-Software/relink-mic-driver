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

## Status (25 November 2026): working, test-signed, confirmed against real hardware

Closed the two gaps above as far as a disposable test VM allows:

- **Test-signed and installed for real.** Secure Boot disabled and
  `bcdedit /set testsigning on` on the lab's Windows 11 test VM, a
  self-signed Code Signing cert generated there, and the real CI
  artifact signed with `signtool` and verified with
  `signtool verify /pa` (0 errors, real DigiCert timestamp). Installed
  via `pnputil`/`devcon`; both endpoints show `Status: OK` in Windows
  and survive a full reboot cleanly. Production signing (a real
  SignPath project) is still not applied for - this remains test-signed
  only, which is fine for development but not for shipping to users.
- **Fed real audio and confirmed working**, including against a real
  Stadia controller's own built-in microphone (not a headset) - see
  ReLink's own `docs/pi-mic-feed.md` for the full test session. Getting
  there needed three real bugs in this repo's own code, each found from
  an actual failing test rather than guessed:
  1. The render-to-mic ring push lived inside `ReadBytes()`
     (`Source/Main/minwavertstream.cpp`), which only runs when
     `g_DoNotCreateDataFiles` is off - default is on
     (`adapter.cpp:51`), so the push was dead code the first time.
     Moved to a new, unconditional `PushRenderedAudioToMicFeed()`.
  2. The shared ring had one read cursor, which broke as soon as more
     than one capture stream opened on the mic at once (Windows' own
     level-meter display opens its own, independent of whatever app is
     actually recording) - whichever stream's `WriteBytes` ran first on
     a given chunk stole it from the other. Fixed to a proper broadcast
     ring: each `CMiniportWaveRTStream` now owns its own read cursor.
  3. The mic's default format
     (`MicArrayPinSupportedDeviceFormats[0]`) was 32-bit PCM while the
     speaker's (`SpeakerHostPinSupportedDeviceFormats[0]`) is 16-bit -
     since the ring copies raw bytes with no conversion, this produced
     real (non-silent) but scrambled audio until both the format-table
     entry and the actual negotiation range
     (`MicArrayPinDataRangesRawStream`) were corrected to match.
- **Considered and deliberately not done**: dropping the virtual
  speaker side entirely in favor of a direct user-mode-to-kernel feed
  into the mic's capture ring, so only one device ever appears in
  Windows' Sound Settings. Decided against - the current design already
  works, a direct feed trades a proven same-process in-kernel ring for
  a new class of problem (an arbitrary process writing into kernel
  memory), and the UX concern that prompted the idea turned out to be a
  testing artifact, not a real constraint on the shipped client. Worth
  reconsidering later as pure polish, not urgent.
