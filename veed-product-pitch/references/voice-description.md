# Voice Description (text mode)

Guidance for collecting a good `voice_description` when the spokesperson
speaks from a text script (mode == "text"). Only relevant on that branch —
skip entirely for audio mode.

## How to ask
"How would you like the voice to sound? For best results, describe multiple
attributes — gender, age, accent, tone, energy. For example: 'Professional
female voice, mid-30s, British accent, warm and confident' works much better
than just 'British accent'. Or say 'auto' to let VEED pick a voice based on
the spokesperson image."

## If the description is too short
If the user gave a very short description (1-3 words like "British accent"),
gently prompt for more before running:

"That's a good start — to get the best voice match, could you add a bit more?
E.g., gender, age range, and overall tone (calm / energetic / authoritative)?"

Then update voice_description with their fuller answer.

## Why the opt-out handling matters
Fabric interprets `voice_description` verbatim. If you pass the literal string
"auto" (or "skip", "default", etc.), Fabric treats it as a description of the
voice and produces a strange-sounding result. Opt-out phrases must map to
"no voice_description flag", never to a literal value. This is the CRITICAL
rule kept inline in SKILL.md.
