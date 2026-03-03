# QORA-TTS 1.7B - Pure Rust Text-to-Speech with Voice Cloning

Pure Rust TTS engine with voice cloning. No Python, no CUDA, no safetensors needed. Single executable + Q4 binary = portable TTS.

Based on **Qwen3-TTS-12Hz-1.7B-Base** (Apache 2.0).

## Quick Start

```bash
# Voice cloning with included voice
qora-tts.exe --model-path . --ref-audio voices/luna.wav --text "Hello, how are you?" --language english

# Different voice
qora-tts.exe --model-path . --ref-audio voices/adam.wav --text "Good morning!" --language english

# Clone your own voice (any 24kHz WAV)
qora-tts.exe --model-path . --ref-audio my_recording.wav --text "Custom voice" --language english

# Control length (codes = seconds x 12.5)
qora-tts.exe --model-path . --ref-audio voices/luna.wav --text "Short" --max-codes 100

# Custom output path
qora-tts.exe --model-path . --ref-audio voices/sagar.wav --text "Hi there" --language english --output greeting.wav
```

## Files

```
  qora-tts.exe          4.3 MB   Inference engine
  model.qora-tts     1559 MB     Q4 weights (talker + predictor + decoder + speaker encoder)
  config.json           4.4 KB   Model configuration
  tokenizer.json         11 MB   Tokenizer (151,936 vocab)
  vocab.json            2.7 MB   Vocabulary
  merges.txt            1.6 MB   BPE merges
  tokenizer_config.json 7.2 KB   Tokenizer config
  voices/                         Reference WAV files for voice cloning
```

**No safetensors needed.** Everything loads from `model.qora-tts`.

## Model Info

| Property | Value |
|----------|-------|
| Base Model | Qwen3-TTS-12Hz-1.7B-Base |
| Type | Voice cloning only (no built-in speakers) |
| Quantization | Q4 (4-bit symmetric, group_size=32) |
| Binary Size | 1559 MB |
| Sample Rate | 24 kHz mono WAV |
| Languages | English, Chinese, German, Italian, Portuguese, Spanish, Japanese, Korean, French, Russian |

## Architecture

| Component | Details |
|-----------|---------|
| **Talker** | 28 layers, hidden=2048, 16/8 GQA heads, SwiGLU 6144 |
| **Code Predictor** | 5 layers, hidden=1024, 16 code groups |
| **Speech Decoder** | 8-layer transformer + Vocos vocoder |
| **Speaker Encoder** | ECAPA-TDNN (3 Res2Net blocks, 2048-dim output) |

## CLI Arguments

| Flag | Default | Description |
|------|---------|-------------|
| `--model-path <dir>` | `.` | Directory containing model.qora-tts + config |
| `--text <text>` | "Hello, how are you today?" | Text to synthesize |
| `--ref-audio <wav>` | - | **Required** - reference WAV for voice cloning |
| `--language <name>` | english | Target language |
| `--output <path>` | output.wav | Output WAV path |
| `--max-codes <n>` | 500 | Max code timesteps (~n/12.5 seconds) |

## Included Voices

| Female | Male |
|--------|------|
| luna, anushri, beth, caty, cherie, ember, faith, hope, jessica, kea, riya, vidhi, velvety | adam, charles, david, hale, heisenberg, joe, peter, quentin, sagar, steven, titan, true |

All 24kHz WAV files in `voices/`. Use any 3-10 second clean speech recording for custom voice cloning.

## Performance (i5-11500, 16GB RAM, CPU-only)

| Phase | Time |
|-------|------|
| Model Load | ~0.8s (from binary) |
| Voice Extraction | ~5-10s |
| Prefill | ~3-8s |
| Code Generation | ~2.5s/code |
| Audio Decode | ~0.5s/frame |
| Memory | ~1560 MB |

## Converting from Safetensors

If you have the original safetensors, convert to binary:

```bash
qora-tts.exe --model-path <safetensors_dir> --save model.qora-tts --text "x" --max-codes 1
```

This creates a single binary with all weights (talker + predictor + decoder + speaker encoder). After conversion, safetensors files are no longer needed.

---

**Built with QORA - Pure Rust AI Inference**
