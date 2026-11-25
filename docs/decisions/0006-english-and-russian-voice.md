# ADR-0006: Voice works in English and Russian

**Status:** Accepted · **Date:** 2026-11-25 · **Decided by:** owner (design Q&A)

## Context

The household speaks English and Russian. People may switch language between commands, and a Russian sentence may contain an English device name.

What the research found (November 2026):

- No single open speech-to-text model is clearly best for both languages on a CPU. The Russian-specialised GigaAM family is far more accurate on short Russian commands than the multilingual models: in one comparison an earlier GigaAM release scored a 1.9% word error rate against 19.5% for Parakeet v3, whose automatic language detection is also reported to pick the wrong language.
- No candidate is documented as handling a switch of language inside one utterance.
- A Russian-only model can write an English name only as a Cyrillic approximation.
- For Russian speech output, word stress is the main quality risk.

## Decision

- Both languages are first-class. A voice feature is not done until it works in both.
- Two specialised speech-to-text models are used instead of one multilingual model (picks in [ADR-0020](0020-voice-pipeline-and-speech-stack.md)).
- The language of a request is decided in this order:
  1. by the wake word used, one per language (satellites);
  2. by the language setting of the device (push-to-talk in the app);
  3. by decoding with both models and keeping the transcript the intent matcher can resolve.
- Every room and device has a display name and aliases in both languages, including a Cyrillic spelling of any English name.
- Entity identifiers are plain ASCII, with the names and aliases in both languages attached, so a model never has to write Cyrillic in a tool call.
- Intent patterns exist per language. A reply is spoken in the language of the request.
- Mixing languages inside one utterance is supported only for device names.
- The command regression set and the voice test set contain English, Russian and Russian-with-English-name items ([ADR-0018](0018-llm-runtime-and-tiers.md)).

## Consequences

- Two speech-to-text models stay loaded. Both are small enough for the 16 GB laptop.
- Intent patterns and aliases are written twice.
- Russian speech output needs either a voice with built-in stress handling or a stress pre-processor.
- The language model must handle Russian requests, which is the least certain part of the model choice.

## Revisit when

A single model handles both languages, including switches inside an utterance, at the accuracy of the specialised pair.

Evidence: [speech and audio notes](../../research_notes/Local%20smart%20home%20stack%20options/speech_and_audio.md), sections on speech-to-text and text-to-speech.
