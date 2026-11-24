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

Not yet verified building or working in this fork - see the repository's
own README and `.github/workflows/` for the (upstream-authored) build and
signing pipeline, which still needs this fork's own SignPath project
credentials (`SIGNPATH_API_TOKEN`/`SIGNPATH_ORG_ID`/`SIGNPATH_PROJECT_SLUG`)
before the signing workflow can run.
