# Contributions

## Team

**1 Tag von München zur Zugspitze**

- Yanwei Xu
- Xiang Shi
- Weitong Lin

This repository documents a team project developed for the EESTech Challenge
2025, Munich Local Round. It is published in Yanwei Xu's portfolio with full
attribution to all team members.

## Yanwei Xu

Primary responsibility: firmware integration, deployment, debugging, and
technical documentation.

Specific contributions:

- Prepared and maintained the ModusToolbox application.
- Integrated multiple DEEPCRAFT-generated model versions.
- Verified model input/output counts, label order, and memory fit.
- Built and signed the complete CM33 Secure, CM33 Non-secure, and CM55 image.
- Programmed and verified both internal and external-memory firmware regions.
- Diagnosed stale incremental builds and enforced clean model rebuilds.
- Added a `0.60` confidence threshold.
- Added a five-result rolling window with a three-vote stable-output rule.
- Mapped `unlabeled` and low-confidence results to `Unknown`.
- Diagnosed and corrected the physical East/West display reversal.
- Performed live UART testing and model comparison on the board.
- Maintained known-working rollback versions.
- Consolidated project, firmware, testing, and submission documentation.

## Xiang Shi and Weitong Lin

Primary responsibility: audio data collection and DEEPCRAFT model workflow.

Joint contributions:

- Configured the board and DEEPCRAFT live microphone collection workflow.
- Recorded and labeled stereo direction and Background sessions.
- Introduced controlled direction, distance, volume, and phone-orientation
  variation.
- Built the DEEPCRAFT classification project.
- Compared preprocessing and model configurations.
- Trained and evaluated candidate Conv1D models.
- Reviewed accuracy, F1, loss curves, and confusion matrices.
- Exported the generated `model.c` and `model.h` files for deployment.
- Collected targeted North/South recordings after board-test feedback.

## Shared Decisions

The team jointly:

- Defined the physical direction convention and demonstration setup.
- Reviewed every deployed model on the real board.
- Chose reliability over the highest offline metric.
- Developed the distance-assisted North/South strategy.
- Selected the final model and prepared the challenge submission.
