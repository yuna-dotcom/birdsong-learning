# birdsong-learning
computational neuroscience
# Birdsong Learning Simulator

A bio-inspired simulation of how juvenile birds learn to sing by imitating a tutor. A bird starts with a random song and iteratively refines it over 500 steps until it sounds as close as possible to the tutor.

---

## How it works

A fixed **tutor song** is generated once and acts as the learning target throughout. A **bird** is initialised with a random sequence of sound events. On each step, the bird generates 5 mutated versions of its current song, scores each one against the tutor using a perceptual audio metric, and keeps the best — if it's an improvement. After 500 steps, audio snapshots and plots are exported so you can hear and see the learning progress.

---

## Project structure

```
├── main.py        # Entry point — runs the full pipeline and produces outputs
├── tutor.py       # Generates the fixed reference song from sound primitives
├── bird.py        # Bird class — holds song events, renders audio, mutates
├── learning.py    # Learning loop — scores candidates and updates the bird
├── audio.py       # Audio utilities — save and play .wav files
└── outputs/       # Generated audio and plots written here
```

---

## Files

### `tutor.py`
Defines the sound primitives used by both the tutor and the bird, and builds the reference song from them.

| Function | Description |
|---|---|
| `note(freq, dur, vr, vd)` | Sine wave at a fixed pitch, with optional vibrato (rate and depth) |
| `chirp_note(f0, f1, dur)` | Frequency glide from f0 to f1 along a smooth S-curve |
| `silence(dur)` | Zero-filled gap used between events |
| `bar(*segments)` | Concatenates any number of segments into one waveform |
| `create_tutor_song()` | Assembles a two-phrase reference song (rising call + falling answer) |

Both `note` and `chirp_note` apply a `sin(πt/dur)^1.2` amplitude envelope so every sound fades in and out smoothly.

---

### `bird.py`
A `Bird` represents its song as a list of parametric events (notes and chirps) rather than raw audio. This makes mutation meaningful — changing an event changes a musical gesture, not random samples.

| Method | Description |
|---|---|
| `_random_event()` | Generates a single random note or chirp event |
| `_render(events)` | Converts an event list into a NumPy waveform, trimmed or looped to match tutor length |
| `_mutate(strength)` | Nudges frequencies and durations with Gaussian noise; occasionally adds or drops an event |
| `_mutate_structure()` | Reorders events structurally — swap, reverse, shuffle, duplicate, or drop |
| `sing()` | Returns the current waveform |
| `export_audio(i)` | Saves the current song as `outputs/bird_{i}.wav` |

---

### `learning.py`
Contains the learning loop — a greedy hill-climber that drives the bird toward the tutor sound.

| Function / Method | Description |
|---|---|
| `mfcc_error(a, b)` | Scores two waveforms by comparing their MFCC matrices — lower means more perceptually similar |
| `Learning.run(steps, export_every)` | Outer loop: runs `step()` 500 times, decays mutation strength, exports audio every 100 steps |
| `Learning.step(strength, n_candidates)` | Core update: generates 5 mutants, scores each, keeps the best if it improves on the current error |

**Mutation strength** decays linearly from 1.0 to 0.05 over training. High strength early on biases toward structural mutations (broad exploration); low strength later biases toward fine frequency nudging (local refinement).

**Per step:** 5 candidate songs are generated and evaluated. Over 500 steps, up to 2,500 songs are scored in total.

---

### `main.py`
Orchestrates the full pipeline:
1. Generates and saves the tutor song
2. Initialises a bird
3. Runs 500 learning steps
4. Plots fundamental frequency trajectories at steps 0, 100, 200, 300, 400
5. Plots the MFCC error curve over time

---

## Outputs

| File | Description |
|---|---|
| `outputs/tutor.wav` | The reference song the bird is learning |
| `outputs/bird_0.wav` | Bird's song before any learning |
| `outputs/bird_100.wav` … `bird_400.wav` | Snapshots every 100 steps |
| `outputs/frequency_progression.png` | F0 trajectory of tutor vs bird at each checkpoint |
| `outputs/learning.png` | MFCC error over 500 training steps |

---

## Dependencies

```
numpy
scipy
librosa
sounddevice
matplotlib
```

Install with:
```bash
pip install numpy scipy librosa sounddevice matplotlib
```

---

## Running

```bash
python main.py
```

Audio playback lines are commented out by default — uncomment the `sd.play` / `sd.wait` calls in `main.py` to hear the tutor and bird live during the run.
