# BL21I

Runtime pattern imported from the DLS support module `BL21I` (the I21 beamline's
support module), release `3-11` (GitLab `controls/support/BL21I`). Its templates are
pristine copies of that release.

The templates use only soft records from EPICS base, calc (`sseq`, `scalcout`,
`transform`) and busy (`busy`), so the pattern needs no compiled code: any generic IOC
image that builds calc and busy can load it.

How each file was obtained is recorded in the header of
[`BL21I.ibek.support.yaml`](BL21I.ibek.support.yaml). That header is vendored with the
pattern, so the provenance travels into every IOC instance that uses it.

## Entity models

| Entity model | Template | Description |
|---|---|---|
| `BL21I.ARMAirControl` | `BL21I_ARM_Air_Control.template` | Combined ARM air bearing control |
| `BL21I.AirBearingInterlockWait` | `BL21I_Air_Bearing_Interlock_Wait.template` | Opens the air valves and waits for the motor interlock to clear |
| `BL21I.SGMTravellerAirControl` | `BL21I_SGM_Traveller_Air_Control.template` | SGM traveller air bearing control |
| `BL21I.SMPLValveSequence` | `BL21I_SMPL_Valve_Sequence.template` | Valve sequence for reinstating vacuum after a sample change |
| `BL21I.TTHBumpstripMonitor` | `BL21I_TTH_Bumpstrip_Monitor.template` | Counter of TTH bumpstrip contacts |
| `BL21I.M5LimitLatch` | `BL21I_M5_Smarpod_Limit_Latch.template` | Latch for the M5 limit switches |
| `BL21I.M5LimitStop` | `BL21I_M5_Smarpod_Limit_Stop.template` | Stops the smarpods when a limit is triggered |
| `BL21I.SMPLTempsController` | `BL21I_SMPL_Temps_Controller.template` | Lakeshore336 control for the sample chamber |
| `BL21I.SMPLCollisionLimits` | `BL21I_SMPL_Collision_Limits.template` | Sample manipulator / M5 collision limits |
| `BL21I.S2DiodeDifference` | `BL21I_S2_Diode_Difference.template` | Normalised difference of two slit drain currents |
| `BL21I.FeedbackAutoPv` | `BL21I_Feedback_Auto_Pv.template` | Shutter-like AutoPV for BLFeedback |

## Not included

- `SCM10` is the `scientificInstSCM10` pattern in ibek-runtime-streamdevice.
- `FS1Control`, `DewarScales`, `ADCAcquisition`, `ADCAcquisitionGroup` and
  `M4ADCSupport` belong to the EtherCAT IOC BL21I-DI-IOC-01: they write to EtherCAT
  asyn ports or link to EtherCAT GenericADC and ERIO PVs, and that EtherCAT support is
  being replaced.

## Files

[`ibek.manifest.yaml`](ibek.manifest.yaml) vendors the support YAML and the
`*.template` files into an IOC instance's `config/`. This README stays in the library.
