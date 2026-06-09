# Testing and Limitations

## Repeatable Test Setup

- Keep the board position and orientation fixed.
- Keep the playback device at microphone height.
- Use a fixed playback volume unless testing robustness.
- Stop the siren and allow the prediction window to settle between trials.
- Reproduce the final North/South distance convention.
- Observe both raw and stable outputs over the KitProg3 UART.

UART settings:

```text
115200 baud, 8 data bits, no parity, 1 stop bit
```

## Initial Final-Model Result

| Input | Board observation |
|---|---|
| East | Stable |
| West | Stable |
| North | Detectable and reaches stable North; occasional South/Unknown transition |
| South | Detectable but less stable than East/West |
| Background | Working |
| Silence/uncertain input | Unknown behavior working |

Captured North samples showed:

```text
North score: approximately 0.69 to 0.74
South score: approximately 0.09 to 0.12
Stable output: North
```

## Why East/West Is Stronger

The two enabled microphones form a spatial baseline along the East/West axis.
A source from East or West creates a useful inter-channel timing and level
difference.

North and South are near the perpendicular bisector of that baseline. Their
two-channel signatures are more symmetric, so room reflections, speaker
orientation, and small position changes can dominate the difference.

## Practical North/South Strategy

The final model uses a controlled distance difference as an auxiliary feature.
This made North and South observable within the hackathon constraints.

The consequence is that the demonstration must reproduce the trained
North/South distances. This is a constrained classifier rather than a
general-purpose front/back direction-of-arrival solution.

## Evaluation Method

The final model was evaluated through repeated live observation in the serial
monitor. The team moved the siren source through all four directions and
observed:

- All six class scores
- Raw output
- Stable output
- Transitions between direction, Background, and Unknown

A controlled five-trial-per-direction benchmark was not completed. No formal
directional accuracy is claimed.

## Optional Reproducibility Trial Sheet

Record the stable label for five trials per direction:

| True direction | 1 | 2 | 3 | 4 | 5 | Correct |
|---|---|---|---|---|---|---:|
| East | | | | | | /5 |
| South | | | | | | /5 |
| West | | | | | | /5 |
| North | | | | | | /5 |

Background checks:

| Input | 1 | 2 | 3 | 4 | 5 | Acceptable |
|---|---|---|---|---|---|---:|
| Quiet | | | | | | /5 |
| Conversation | | | | | | /5 |
| Keyboard/room noise | | | | | | /5 |

For a background trial, `Background` or `Unknown` is acceptable.

## Known Limitations

- Only two of the four onboard microphones were enabled.
- North/South remains less stable than East/West.
- The final front/back distinction depends partly on distance.
- Training and validation windows may share a source recording.
- Testing covered a limited room, speaker, and siren source.
- The final board evaluation is observational rather than a logged
  five-trial-per-direction benchmark.

## Future Work

1. Enable and synchronize all four onboard microphones.
2. Export a four-channel DEEPCRAFT preprocessor and model.
3. Split train, validation, and test data by complete recording session.
4. Collect multiple rooms, distances, playback devices, and siren types.
5. Measure actual distances and sound-pressure levels.
6. Tune confidence and temporal hysteresis from a larger independent test set.
