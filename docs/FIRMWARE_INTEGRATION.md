# Firmware Integration

## Base Application

The solution is contained in:

```text
firmware/
```

The application consists of:

- CM33 Secure project
- CM33 Non-secure project
- CM55 project with the generated DEEPCRAFT model

The signed and merged image is programmed to RRAM and external SMIF flash.

## Model Interface

The final generated header defines:

```c
#define IMAI_DATA_IN_COUNT (2)
#define IMAI_DATA_OUT_COUNT (6)
#define IMAI_DATA_OUT_SYMBOLS \
    {"unlabeled", "Background", "East", "North", "South", "West"}
```

The model files are integrated without manually editing generated model code:

```text
proj_cm55/model/model.c
proj_cm55/model/model.h
```

## Application-Level Output Logic

The generated model returns one score per class. The application adds a
deployment layer in `proj_cm55/audio.c`.

### Confidence Rejection

```c
#define OUTPUT_THRESHOLD_SCORE (0.6f)
```

Predictions below the threshold are reported as `Unknown`. The generated
`unlabeled` class is also treated as `Unknown`.

### Temporal Stabilization

```c
#define PREDICTION_WINDOW_SIZE (5u)
#define PREDICTION_MIN_VOTES   (3u)
```

The firmware stores the five most recent accepted or rejected predictions. A
class becomes stable only after receiving at least three votes.

The UART retains both views:

```text
Raw output: North      (0.7405)
Stable output: North
```

This makes model debugging possible without sacrificing demonstration
stability.

### East/West Calibration

Initial board tests showed a deterministic physical East/West reversal between
the DEEPCRAFT recording convention and deployment output. The input channel
order remains the original `[left, right]`; the application corrects the
physical East/West display labels after calibration.

## Build Configuration

```text
TARGET=APP_KIT_PSE84_AI
ML_DEEPCRAFT_CPU=cm55
NN_TYPE=int8x8
```

The final model build completed successfully:

| Resource | Result |
|---|---:|
| Generated model memory | 15,576 bytes |
| Generated scratch memory | 1,661,952 bytes |
| Linked CM55 SoCMEM data | 1,933,848 bytes |
| Remaining CM55 heap | 933,352 bytes |
| CM55 external image partition | 78% used |

## Clean Build Requirement

When restoring older generated model files, their timestamps may be older than
existing build objects. An incremental build can therefore reuse the wrong
model. The reliable workflow is:

```bash
make clean
make program
```

The program step writes and verifies the SMIF region containing the CM55 model.

## Final Artifact

```text
build/app_combined.hex
```

SHA256:

```text
3528764E57229482D1F71966FC1DCA6DA5B31F30E45F5D5D05576312D4958397
```
