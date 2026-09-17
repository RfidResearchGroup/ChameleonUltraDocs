# Battery Calibration

The battery percentage is derived from the reported battery voltage using an empirically tuned curve.
The default curve is calibrated so that the lowest stable full-charge voltage still maps to 100%.
This percentage is a state-of-charge estimate, not a direct capacity measurement.
The Lite uses a smaller button cell than the Ultra, but both variants use the same single-cell 4.2 V charging path, so the calibration reference is still based on voltage rather than capacity.

## Recommended procedure

1. Fully charge the device until the charger indicates completion.
2. Disconnect USB and let the battery settle for a few minutes.
3. Wake the device, wait at least 5 seconds, then run `hw battery`.
4. Repeat the command a few times and keep the lowest stable voltage you observe while the pack is still freshly charged.
5. Use the lowest stable full-charge reading as the reference when validating or rebuilding firmware. On the boards discussed in issue #334, the plausible range is about `4160-4190 mV`, so `4160 mV` is a safe conservative reference.

## What to look for

- The full-charge voltage should be stable across repeated reads.
- The `hw battery` command should show `100%` once the settled full-charge voltage is reached.
- The protocol also exposes `GET_BATTERY_INFO_EX`, which adds a compact `condition` field to the standard battery payload.
- The same `hw battery` command now prefers an extended battery-info response and prints a qualitative `condition` hint (`excellent`, `good`, `fair`, `low`, or `critical`) based on the measured voltage and state-of-charge estimate.
- On-device, the battery screen now uses the same condition state to color the bar and adds short blink pulses for `low` and `critical` states.
- This condition hint is a practical classification derived from voltage and state-of-charge; it is not a direct measurement of the cell's true state of health or capacity.
- LED mapping on the device is:

  | Condition | LED color | Blink behavior |
  | --- | --- | --- |
  | `excellent` | cyan | none |
  | `good` | green | none |
  | `fair` | yellow | none |
  | `low` | yellow | 1 extra blink |
  | `critical` | red | 2 extra blinks |

- If the reported percentage dips below `100%` immediately after a full charge, the curve reference is too high for that board.

## State of charge vs state of health

- **State of charge (SoC)** is the current battery fill estimate, expressed here as the reported percentage and derived from the voltage curve.
- **State of health (SoH)** is a longer-term measure of how much usable capacity the cell can still deliver compared with its nominal capacity.
- The current firmware and protocol expose a **health hint**, not a true SoH reading.
- That hint is useful because it gives users a stable, human-readable status without pretending to measure the battery's long-term condition.
- In other words: the percentage tells you **how full the battery is now**, while the condition hint tells you **how the current reading should be interpreted**.
## Notes

- Battery reads are most reliable after wake-up, once the analog path has settled.
- This project does not currently expose a runtime battery calibration command; calibration is performed by validating the measured full-charge reference used by the firmware curve.
