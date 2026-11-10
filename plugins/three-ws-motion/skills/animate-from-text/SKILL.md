---
name: animate-from-text
description: Generate a full-body humanoid animation clip (three.js AnimationClip JSON) from a short prompt or a Motion Score using @three-ws/motion. Use when the user wants a procedural wave, sit, bow or custom gesture for a 3D avatar without mocap or a model.
---

# Animate a humanoid from text

`@three-ws/motion` compiles a prompt or a structured **Motion Score** into a three.js `AnimationClip.toJSON()` document using forward and inverse kinematics, balance, ground contact and idle motion. Zero dependencies, deterministic, no network, no API key. Output uses canonical bone names so it retargets onto Mixamo, VRM, Ready Player Me and similar rigs.

## CLI

```bash
npx -y -p @three-ws/motion three-ws-motion "wave hello twice, excitedly" -o wave.json
npx -y -p @three-ws/motion three-ws-motion --score sit.json --no-idle -o sit.json
npx -y -p @three-ws/motion three-ws-motion --actions   # actions the built-in lane knows
npx -y -p @three-ws/motion three-ws-motion --schema    # Motion Score JSON Schema
```

Options: `-o/--out <file>` (default stdout), `--score <file>`, `--name <name>`, `--loop`, `--effort <name>`, `--no-idle`, `--no-root` (clip in place), `--no-fingers`, `--score-out <file>` (also write the score used), `--quiet`. Errors exit with code 2. A prompt outside the built-in vocabulary fails: write a score instead.

## Library

```js
import { motionFromText, compileScore, scoreSchema, validateScore } from '@three-ws/motion';

const { clip, score, matched, warnings } = motionFromText('sit down heavily');
```

Playing it in three.js:

```js
import { AnimationClip, AnimationMixer } from 'three';
new AnimationMixer(avatar).clipAction(AnimationClip.parse(clip)).play();
```

`compileScore(score, { idle, rootMotion, fingers })` returns `{ clip, score, warnings }`. `validateScore(input)` returns `{ ok, score, error }` with the failing path. `scoreSchema()` is the JSON Schema for a tool call.

## Authoring a Motion Score yourself

A score is `{ name, loop?, effort?, beats: [...] }`. Each beat is a pose at an instant with `in` (seconds to reach it) and `hold`. Fields: `posture` (`stand`, `crouch`, `sit`, `lunge`, `bow`, `tiptoe`), `root`, `torso` (`lean`, `twist`, `sideBend`), `arms` (per side: `at` anchor such as `chin`, `sternum`, `overhead`, `thigh`, `front`, plus `palm`, `hand`, `elbow`), `legs`, `gaze`, `face`, `effort`.

Always run `--schema` (or `scoreSchema()`) first and use only the enum values it lists. Example:

```json
{
  "name": "thinking it over",
  "beats": [
    { "label": "still", "posture": "easy", "hold": 0.4 },
    { "label": "hand to chin", "posture": "easy",
      "arms": { "right": { "at": "chin", "palm": "in", "hand": "loose", "elbow": "down" } },
      "gaze": "aside", "face": "focused", "in": 0.5, "hold": 1.2 }
  ]
}
```

Save it as `think.json`, then `three-ws-motion --score think.json -o think.json.clip.json`. Read the stderr summary and any `warning:` lines, and fix the score if there are warnings.
