# VoiceBox

**An offline-first, Rust-native speech recognition and speech synthesis project.**

VoiceBox is an experimental speech technology project focused on building lightweight, locally running speech systems. The long-term goal is to develop our own speech recognition (STT) and text-to-speech (TTS) pipelines, with a native Rust inference runtime and support for English, Hindi, and Hinglish.

Inspired by the idea of rebuilding a lost voice, VoiceBox aims to explore what it takes to give software a voice without depending on cloud APIs.

> **Project status:** Early development / research. The roadmap describes planned capabilities, not features that are already implemented.

## Goals

- Build a speech processing pipeline in Rust.
- Work offline after required models and resources have been installed.
- Begin with a tiny English keyword recognition model.
- Progress from keyword recognition toward general speech transcription.
- Investigate Hindi-English code-switching and Hinglish transcription.
- Develop a separate text-to-speech pipeline.
- Explore custom model training and native inference.
- Optimize for CPU-only systems with limited memory.
- Measure accuracy, latency, memory usage, and model size.
- Keep the architecture modular and testable.

## Non-goals

Initially, VoiceBox is not intended to be:

- A replacement for state-of-the-art commercial speech systems.
- A cloud-based transcription service.
- A wrapper around an online speech API.
- A general-purpose conversational AI assistant.
- A system that requires a GPU to run its basic functionality.

These boundaries may evolve as the project develops.

## Design targets

| Property | Initial target |
|---|---|
| Primary language | Rust |
| Build system | Cargo |
| Operating system | Linux, initially Debian |
| Hardware | CPU-first |
| Memory budget | 8 GB system RAM |
| Initial STT language | English |
| Later STT languages | Hindi and Hinglish |
| Speech synthesis | Separate TTS subsystem |
| Network dependency | None during offline inference |
| Model strategy | Start small; investigate custom training |
| Interface | CLI-first |

These are development targets rather than verified performance guarantees.

## Architecture

VoiceBox is planned as a set of independent components.

```text
                    VOICEBOX
                        |
            +-----------+-----------+
            |                       |
       VoiceBox STT             VoiceBox TTS
            |                       |
      Audio capture             Text input
            |                       |
      Audio preprocessing      Text normalization
            |                       |
      Feature extraction       Text/phoneme encoding
            |                       |
      Model inference          Speech generation
            |                       |
      Decoding                 Waveform processing
            |                       |
        Text output             Audio output
            |                       |
            +-----------+-----------+
                        |
                  VoiceBox Core
            Configuration, CLI and metrics
```

The STT and TTS components will remain independent so they can use different models, preprocessing strategies, and optimization techniques.

## Technology choices

| Technology | Intended role |
|---|---|
| Rust | Main implementation language |
| Cargo | Dependency management, compilation and testing |
| `cpal` | Cross-platform audio input and output |
| `hound` | WAV file reading and writing |
| `rustfft` | Fourier transform operations |
| `candle-core` / `candle-nn` | Candidate tools for neural network inference and experiments |
| `criterion` | Benchmarking Rust code |
| `cargo test` | Unit and integration testing |

The exact dependency set will be chosen as each component is implemented. No dependency should be introduced without a clear purpose.

## Development roadmap

### Phase 0 — Project foundation

**Objective:** Establish a reproducible Rust development environment.

Planned work:

- Create the Cargo project.
- Establish module boundaries and error handling.
- Implement basic CLI commands.
- Add formatting, linting and tests.
- Document build and execution instructions.

**Completion criteria:** The project builds and its tests pass on the target Linux system.

### Phase 1 — Audio I/O and digital signal processing

**Objective:** Understand and process raw audio before introducing a neural model.

Planned work:

- Read and write WAV files.
- Capture microphone input.
- Inspect sample rates, channels and sample formats.
- Convert stereo input to mono when required.
- Implement framing and windowing.
- Explore FFT-based spectral analysis.
- Implement log-Mel spectrogram extraction.
- Test numerical results against reference vectors.

**Completion criteria:** VoiceBox can record, process and replay audio, and its feature extraction is covered by tests.

### Phase 2 — Tiny English keyword recognition

**Objective:** Train and run a small speech model without a cloud service.

Planned work:

- Select an appropriately licensed speech dataset.
- Establish a training and evaluation split.
- Begin with a limited vocabulary.
- Implement or integrate a small model architecture.
- Export trained weights into a documented format.
- Implement model inference in Rust.
- Add an unknown-class or rejection mechanism.
- Evaluate accuracy on unseen speakers and recordings.

**Completion criteria:** A native executable recognizes the supported keywords offline and reports measured performance.

Keyword recognition is not general speech transcription. This phase is a controlled starting point.

### Phase 3 — General English speech recognition

**Objective:** Progress from isolated words to continuous speech.

Research areas:

- Acoustic and sequence modeling.
- Character or subword tokenization.
- Connectionist temporal classification (CTC) and other suitable sequence objectives.
- Greedy and beam-search decoding.
- Text normalization.
- Training-data quality and speaker diversity.
- Word error rate (WER) and other evaluation metrics.

**Completion criteria:** VoiceBox can transcribe a defined set of unseen English utterances, with accuracy and latency documented.

### Phase 4 — Hindi and Hinglish

**Objective:** Investigate speech containing Hindi, English, and code-switching.

Planned work:

- Identify suitable multilingual and code-switched speech datasets.
- Check dataset licenses and usage conditions.
- Evaluate Devanagari and Romanized text representations.
- Measure Hindi, English and mixed-language performance separately.
- Investigate language switching within a sentence.
- Document common transcription errors.

**Completion criteria:** A reproducible evaluation set and benchmark exist for Hindi, English and Hinglish.

Hinglish support will require suitable data and modeling. It is not guaranteed by adding a language label to an English model.

### Phase 5 — Text-to-speech

**Objective:** Build a separate offline speech synthesis pipeline.

Planned work:

- Normalize numbers, punctuation and abbreviations.
- Investigate text, phoneme and token representations.
- Compare lightweight synthesis architectures.
- Explore speech datasets with appropriate licensing.
- Implement waveform generation and audio playback.
- Measure speech quality, synthesis latency and memory usage.

**Completion criteria:** VoiceBox can generate intelligible speech from text and save the generated audio to a file.

Natural-sounding multilingual TTS is a longer-term objective, not an initial guarantee.

### Phase 6 — Performance and deployment

**Objective:** Make the speech engines practical on modest CPU-only hardware.

Planned work:

- Measure model loading and startup costs.
- Benchmark real-time factor and end-to-end latency.
- Measure peak resident memory and model storage.
- Investigate quantization and reduced-precision arithmetic.
- Optimize audio buffers and tensor operations.
- Explore streaming recognition and synthesis.
- Add model metadata and compatibility checks.
- Test offline operation with network access disabled.

**Completion criteria:** Each supported model has reproducible performance measurements and documented resource requirements.

## Proposed repository structure

```text
voiceBox/
├── Cargo.toml
├── Cargo.lock
├── README.md
├── LICENSE
├── .gitignore
├── src/
│   ├── main.rs
│   ├── lib.rs
│   ├── cli.rs
│   ├── config.rs
│   ├── error.rs
│   ├── audio/
│   │   ├── mod.rs
│   │   ├── capture.rs
│   │   ├── wav.rs
│   │   └── features.rs
│   ├── stt/
│   │   ├── mod.rs
│   │   ├── model.rs
│   │   └── decoder.rs
│   └── tts/
│       └── mod.rs
├── tests/
├── benches/
├── docs/
├── scripts/
└── models/
```

This is a proposed structure. Modules should be added as they become necessary; unfinished components should not be created just to fill out the tree.

Model files, downloaded datasets and generated audio should generally stay out of Git. The repository should contain the code and documentation needed to reproduce the workflow.

## Initial setup

Install the Rust toolchain using [rustup](https://rustup.rs/).

On Debian, install the basic native development dependencies:

```bash
sudo apt update
sudo apt install build-essential pkg-config libasound2-dev
```

Create the project:

```bash
cargo new voicebox
cd voicebox
```

Build and run:

```bash
cargo build
cargo run
```

Run tests and format the code:

```bash
cargo test
cargo fmt --check
cargo clippy --all-targets --all-features
```

Audio libraries may require additional system packages depending on the selected backend.

## Development principles

1. **Offline-first:** Required models and resources must be available locally before offline inference can work.
2. **Measure before optimizing:** Record a baseline before changing algorithms or data structures.
3. **Small milestones:** Every phase should produce a working, testable artifact.
4. **Explicit interfaces:** Audio processing, inference, decoding and output should be independently testable.
5. **Reproducibility:** Record model versions, dataset splits, training settings and benchmark conditions.
6. **Honest evaluation:** Report measured accuracy and limitations rather than claiming state-of-the-art performance.
7. **Model provenance:** Document the origin, license and training status of every model.
8. **Privacy:** Local processing should not transmit recordings without explicit user action.

## Evaluation metrics

VoiceBox will use different metrics for different tasks.

| Metric | Purpose |
|---|---|
| Accuracy | Initial keyword classification |
| Word error rate (WER) | General speech transcription |
| Character error rate (CER) | Character-level transcription comparison |
| Real-time factor (RTF) | Processing time relative to audio duration |
| Peak RAM usage | Memory consumption during inference |
| Model size | Storage required for model weights |
| Startup latency | Time to initialize the model |
| Speech quality | Intelligibility and naturalness of generated audio |

Benchmarks should identify the hardware, dataset, model version and measurement procedure. Performance numbers will be published only after measurement.

## Training data and licensing

Before training or distributing a model:

- Verify the dataset license and any restrictions on redistribution.
- Keep speaker privacy and consent requirements in mind.
- Document training and evaluation splits.
- Avoid evaluating solely on recordings used during training.
- Record pretrained model provenance when applicable.
- Review the licenses of dependencies, weights and generated assets.

## Contributing

VoiceBox is initially a learning and research project.

Potential areas for contribution include:

- Audio processing and DSP.
- Rust inference implementations.
- Dataset preparation and evaluation.
- CPU performance optimization.
- Speech recognition experiments.
- Text-to-speech research.
- Testing and documentation.

Contributions should include tests or reproducible evaluation where applicable.

## License

The project license has not yet been selected.

Before publishing, choose a license for the source code and verify the redistribution terms of all bundled dependencies and model weights. A model's license may differ from the license of the code used to run it.

## Current status

VoiceBox is in the planning and initial implementation stage.

The immediate milestone is a Rust audio foundation followed by a tiny English keyword recognizer. General transcription, Hinglish, custom neural inference and TTS are future development goals.

**Build small. Measure everything. Give the machine its voice.**

