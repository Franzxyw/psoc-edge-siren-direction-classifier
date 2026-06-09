# Dataset and Training

## DEEPCRAFT Collection Workflow

The PSOC Edge E84 AI Kit was prepared with the Infineon data-collection
firmware. In DEEPCRAFT Studio, the board microphone source was connected to a
Data Track and recorded as live stereo sessions.

Each session was labeled as `East`, `West`, `North`, `South`, or `Background`.
The generated classifier also retains an `unlabeled` output, which the firmware
exposes as `Unknown`.

## Dataset Evolution

The first dataset revealed two quality problems:

- Several North sessions referenced duplicate audio.
- Directional labeled durations were unbalanced.

The team therefore recorded a new balanced dataset instead of relying on the
initial results.

### Five-Session Dataset

The improved dataset contained five independent recordings per direction and
two Background sessions.

| Label | Labeled duration |
|---|---:|
| North | 166.74 s |
| East | 156.24 s |
| South | 168.16 s |
| West | 155.41 s |
| Background | 59.06 s |

### Seven-Session Dataset

Two additional sessions per direction introduced phone-orientation variation:

- Session 06: phone placed flat
- Session 07: phone speaker raised

This revision contained 31 sessions:

- Seven per direction
- Three Background sessions

| Label | Labeled duration |
|---|---:|
| North | 226.27 s |
| East | 218.41 s |
| South | 232.76 s |
| West | 214.74 s |
| Background | 96.34 s |

The four directional classes were balanced within approximately 8%.

### Final Targeted Revision (`final3`)

Board testing showed persistent North/South confusion. The two enabled
microphones are arranged on the East/West axis, so North and South have weak
inter-channel differences.

The final training revision added:

- Two independent North recordings
- Two independent South recordings
- A deliberately different and reproducible source distance for North and
  South

The final project therefore contains 35 sessions.

## Quality Audit

The improved dataset was checked for:

- Stereo channel count
- 16 kHz sample rate
- 16-bit PCM format
- Duplicate audio hashes
- Clipping
- Class-duration balance

The seven-session dataset passed these checks with no duplicate WAV hashes and
no detected clipping.

## Preprocessing and Model Selection

The team compared multiple preprocessing and model complexity options. The
selected pipeline used:

- Contextual Sliding Window preprocessing
- Low-complexity configuration
- Small balanced Conv1D classifier

The low-complexity option trained faster and fit the limited dataset without
unnecessary model size.

Candidate training runs were compared using accuracy, F1, loss curves,
accuracy curves, and confusion matrices. Training was repeated several times,
with typical runs taking approximately 10 to 15 minutes.

## Dataset Split

The later training project used approximately:

```text
Training:   70%
Validation: 20%
Test:       10%
```

This split should be interpreted carefully. If windows from one long recording
are distributed across sets, the samples are not fully independent. The team
therefore used flashed-board testing as the final model-selection criterion.

## Model Selection Lesson

One candidate achieved:

- Test accuracy: 87.03%
- Test F1: 82.43%

Despite these stronger offline numbers, it classified physical North as South
with high confidence on the board. The team rejected that model and selected
the final `final3` revision based on better deployed behavior.
