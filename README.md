# PSOC Edge Siren Direction Classifier

An embedded Edge AI system that classifies the direction of an emergency
vehicle siren on the Infineon PSOC Edge E84 AI Kit.

This project was created for the **EESTech Challenge 2025, Munich Local
Round**, organized by EESTEC LC Munich in collaboration with Infineon
Technologies, by team **1 Tag von München zur Zugspitze**:

- Yanwei Xu
- Xiang Shi
- Weitong Lin

## What It Does

The system uses two enabled onboard microphones and a DEEPCRAFT-generated
Conv1D model to classify:

- East
- West
- North
- South
- Background
- Unknown

The model runs on the Arm Cortex-M55 with Ethos-U55 support. Predictions are
reported over the KitProg3 UART as both raw and stabilized outputs.

```text
Raw output: North      (0.7405)
Stable output: North
```

## My Contribution

My primary responsibility was the embedded deployment and system integration:

- Integrated generated `model.c` and `model.h` files into the CM55 firmware.
- Verified the two-input/six-output model interface and label ordering.
- Built, signed, programmed, and verified the CM33 Secure, CM33 Non-secure,
  and CM55 firmware image.
- Diagnosed model-cache and external SMIF programming issues.
- Added confidence rejection and a rolling majority-vote stabilizer.
- Calibrated the physical East/West output mapping.
- Debugged model behavior through live UART scores and board testing.
- Maintained rollback versions and selected the final deployed model.
- Consolidated the technical documentation and final submission package.

See [CONTRIBUTIONS.md](./CONTRIBUTIONS.md) for team attribution.

## Architecture

```mermaid
flowchart LR
    A["Two onboard PDM microphones"] --> B["Stereo PCM capture"]
    B --> C["Normalization"]
    C --> D["DEEPCRAFT preprocessor + Conv1D model"]
    D --> E["Six class scores"]
    E --> F["0.60 confidence threshold"]
    F --> G["Five-result rolling vote"]
    G --> H["Raw and stable UART output"]
```

## Engineering Highlights

### Stable Embedded Output

The firmware adds a deployment layer around the generated model:

- Minimum confidence: `0.60`
- Prediction window: five results
- Stable output: at least three matching votes
- Low-confidence and `unlabeled` predictions become `Unknown`

This suppresses isolated prediction jumps while retaining raw scores for
debugging.

### North/South Ambiguity

East and West align with the baseline between the two enabled microphones and
therefore provide strong stereo evidence. North and South lie near the
perpendicular bisector and are much harder to distinguish with only two
channels.

The final dataset introduced a reproducible distance difference as an
additional acoustic cue for North and South. This made all four directions
observable within the hackathon constraints, while East and West remained the
most stable.

### Model Selection by Board Behavior

One candidate achieved `87.03%` test accuracy and `82.43%` test F1 in
DEEPCRAFT, but repeatedly classified physical North as South on the board. We
rejected that candidate and continued targeted data collection. The final
selection was based on deployed behavior rather than offline metrics alone.

## Hardware and Tools

| Component | Configuration |
|---|---|
| Board | Infineon PSOC Edge E84 AI Kit |
| Audio input | Two onboard PDM microphones |
| Inference CPU | Arm Cortex-M55 |
| Accelerator | Ethos-U55 |
| Model workflow | DEEPCRAFT Studio |
| Firmware workflow | ModusToolbox 3.8 |
| Toolchain | GNU Arm Embedded 14.2.1 |
| Collection format | Stereo, 16 kHz, 16-bit PCM |

## Repository Structure

```text
.
|-- firmware/          Complete ModusToolbox application
|-- docs/              Dataset, integration, and testing notes
|-- model/             DEEPCRAFT code-generation report
|-- release-assets/    Local final HEX for GitHub Release upload
|-- CONTRIBUTIONS.md   Team attribution and my role
|-- NOTICE.md          Infineon source and license notice
`-- README.md
```

`release-assets/` is intentionally ignored by Git. The final HEX should be
published as a GitHub Release asset.

## Build and Program

Install ModusToolbox 3.8 and its default packages, then run from `firmware/`:

```bash
make getlibs
make clean
make program
```

The clean step is important when changing generated models because older
timestamps can otherwise allow stale model objects to be reused.

Open the KitProg3 UART at:

```text
115200 baud, 8 data bits, no parity, 1 stop bit
```

## Final Model

| Field | Value |
|---|---|
| Model ID | `86afe4fe-3f64-4c14-b192-78fa0d9feaaa` |
| Inputs | 2 |
| Outputs | 6 |
| Labels | unlabeled, Background, East, North, South, West |
| Generated model memory | 15,576 bytes |
| Generated scratch memory | 1,661,952 bytes |
| Linked CM55 SoCMEM data | 1,933,848 bytes |
| Remaining CM55 heap | 933,352 bytes |

Final firmware SHA256:

```text
3528764E57229482D1F71966FC1DCA6DA5B31F30E45F5D5D05576312D4958397
```

## Observed Board Behavior

| Input | Observation |
|---|---|
| East | Stable |
| West | Stable |
| North | Detectable and reaches stable North; occasional South/Unknown transition |
| South | Detectable but less stable than East/West |
| Background/silence | Background or Unknown behavior works |

Evaluation was performed through repeated live serial-monitor observation. We
do not claim a formal directional accuracy because a logged five-trial
benchmark was not completed.

## Documentation

- [Dataset and training](./docs/DATASET_AND_TRAINING.md)
- [Firmware integration](./docs/FIRMWARE_INTEGRATION.md)
- [Testing and limitations](./docs/TESTING_AND_LIMITATIONS.md)
- [Model report](./model/code_generation_report.md)

## Attribution and License

The firmware is derived from Infineon's PSOC Edge DEEPCRAFT deploy-audio code
example and the Infineon Hackathon repository. The original Infineon EULA is
preserved inside [`firmware/LICENSE`](./firmware/LICENSE).

See [NOTICE.md](./NOTICE.md) before reusing or redistributing the firmware.
