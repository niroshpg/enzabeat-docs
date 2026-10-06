# Architecture decision records

Short, dated notes on the choices that are expensive to reverse.

## Overview: a shared beat, not a shared stream

Every ADR below rests on one premise: a band stays together when everyone can see the same moment in the music, and the cheapest reliable way to give them that is to share *time and position*, never sound.

<p align="center">
  <img src="diagrams/topology.png" alt="Each player's phone hears that player through its built-in microphone, or an optional instrument microphone, and a part tracker on the phone follows that player's own part. The phone sends only its position to a session host, which keeps one shared clock, weighs every position and sends one shared beat back to every score view. Audio never leaves the phone." width="900">
</p>

The phone listens. The host keeps time and combines positions. Every screen draws the same cursor. Each ADR either keeps that split intact or says what it costs.

## ADR-001: Each phone follows its own player, on the phone (2026-10)

The first design sent every phone's audio to a central computer, which tracked all the players at once. That central computer would get a delayed, lossy copy of sound the phone already had, from a phone that already knew which part its player was reading.

So each phone runs its own part tracker. It follows its player through that player's own part, note by note, and highlights the current note immediately. A phone knows what to expect next, which makes the job tractable even in a noisy room.

**Cost:** tracking runs on every phone, so it has to be efficient on mid-range devices, and the same logic has to behave identically on iOS and Android. ADR-008 addresses the second part.

## ADR-002: Audio never leaves the phone (2026-10)

Once tracking happens on the phone, the network only needs to carry a position and a confidence: a few bytes, a few times a second. There is no audio to compress, stream, buffer against Wi-Fi jitter, or protect.

That makes a privacy promise for free (nobody's playing is sent anywhere), and it makes the network requirements small enough that a phone hotspot is plenty.

**Cost:** the session host can never "listen again" to settle a disagreement between phones. It has only what the phones report. Optional session recording, when built, uploads each phone's track after the session, never during it.

## ADR-003: One clock and one beat, drawn locally (2026-10)

The obvious design broadcasts the current bar and beat every time it changes. On Wi-Fi that puts tens of milliseconds of varying delay on every screen, so neighbouring phones disagree and the cursor steps instead of gliding.

Instead every phone keeps its clock in step with the session host's clock, and the host sends occasional reference points: where the band was at a given moment on the shared clock, and how fast it is moving. Each phone draws the cursor itself, every frame.

**Cost:** clock agreement becomes the foundation of the product, so it is proven on real routers and hotspots before anything else is built on it. If it ever fell short, the fallback is to sync clocks acoustically rather than to start streaming audio.

## ADR-004: The built-in microphone is the baseline; an instrument microphone is an upgrade (2026-10)

Enzabeat is for groups who meet and play, often without planning. Requiring a clip-on microphone or an audio interface for every player would rule most of those sessions out.

So the phone's own microphone, on a music stand near its player, has to be good enough on its own. The tracker is designed for it to hear the rest of the band as well. A microphone or pickup mounted on the instrument is supported and improves accuracy for that player, but nothing depends on it, and players can mix both in one session.

**Cost:** the hardest listening conditions are the default ones. Accuracy is measured on built-in microphones first.

## ADR-005: Every player gets a vote, and the cursor never stops (2026-10)

A fixed "two out of three agree" rule breaks with two players or with eight, and it ignores the fact that players rest, solo and stop.

Each phone reports how sure it is, and a phone whose player is resting says so and stays out of the vote. The host weighs the positions it has, so one player's mistake or a dropped phone doesn't move the band's cursor. When nobody is sure, the cursor holds the last tempo instead of freezing, and goes back to following the band when it hears the band again.

**Cost:** the cursor can drift during a long passage nobody's phone can follow. The leader can always nudge or seek (ADR-006).

## ADR-006: The leader always wins (2026-10)

Listening will sometimes be wrong. A rehearsal can't stop to argue with an app.

One player holds the leader role, and it can be handed over. The leader can pause, seek to a bar, set the tempo, switch between following the band and conductor tempo, or lock the cursor so the phones' listening is ignored. A manual action always overrides what the phones hear.

## ADR-007: Local first, and the host can move (2026-10)

The default session runs entirely on one Wi-Fi network or hotspot, with a laptop or a leader phone as the session host. Players join by scanning a code on the host. No account and no internet connection are needed.

Because only small, timing-tolerant messages travel (ADR-002, ADR-003), the same session logic can later run on a relay on the internet, for players in different places or on networks that keep devices apart.

**Cost:** some venue and guest Wi-Fi networks stop devices talking to each other. The app explains how to fall back to a hotspot.

## ADR-008: Two native apps on one shared core (2026-10)

Listening and timing are where the two platforms must behave identically, and where performance matters. They are written once, in a shared core used by both apps and by the session host. The interface is native on each platform: SwiftUI on iOS and Jetpack Compose on Android, with the same rationale and conventions as [Scribatic](https://github.com/niroshpg/scribatic). Audio capture, audio-session handling and background behaviour are exactly what cross-platform UI toolkits abstract away.

**Cost:** two interfaces to keep in step. The behaviour that has to match is small and specified here. How it looks may differ by platform.

## ADR-009: Public design, private implementation (2026-10)

The behaviour above is what the product promises, so it is published. How the tracker listens, how positions are weighed, the message formats and timing protocol, and the measurements behind them stay private. They are the product's real work, and none of the promises above depends on them staying secret.

The implementation repository consumes this one as a submodule. A change to the design is made here, in the open, first.
