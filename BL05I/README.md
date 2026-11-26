# BL05I

Runtime pattern imported from the DLS support module `BL05I` (the I05 beamline's
support module, which also carries the templates of the J branch, BL05J), release
`2-95` (GitLab `controls/support/BL05I`). Its templates are pristine copies of that
release.

The templates use only soft records: `ai`, `ao`, `bi`, `bo`, `calc`, `calcout`,
`dfanout`, `fanout`, `longout`, `mbbi`, `mbbiDirect`, `mbbo`, `seq`, `stringin`,
`stringout` and `subArray` from EPICS base, `scalcout`, `sseq` and `transform` from
calc, `busy` from busy, and `timestamp` from std (in `BL05I_Helium_recovery.template`
only). The pattern needs no compiled code: any generic IOC image that builds calc and
busy can load it, and one that also builds std for `BL05I_He_Recovery`.

How each file was obtained is recorded in the header of
[`BL05I.ibek.support.yaml`](BL05I.ibek.support.yaml). That header is vendored with the
pattern, so the provenance travels into every IOC instance that uses it.

## Entity models

| Entity model | Template |
|---|---|
| `BL05I.BL05I_waterchiller_interlock` | `BL05I_waterChiller_interlock.template` |
| `BL05I.BL05I_pgmHomingSteps` | `BL05I_PGM_homingPLC_steps.template` |
| `BL05I.BL05I_FeedbackAutoPv` | `BL05I_Feedback_AutoPV.template` |
| `BL05I.BL05I_HR_manipulator` | `BL05I_HR_manipulator.template` |
| `BL05I.BL05I_HR_manipulatorLimit` | `BL05I_HR_manipulatorLimit.template` |
| `BL05I.BL05I_HR_manipulatorSlit` | `BL05I_HR_manipulatorSlit.template` |
| `BL05I.BL05I_HR_manipulatorXZone` | `BL05I_HR_manipulatorXZone.template` |
| `BL05I.BL05I_HR_manipulatorXZoneSet` | `BL05I_HR_manipulatorXZoneSet.template` |
| `BL05I.BL05I_He_Recovery` | `BL05I_Helium_recovery.template` |
| `BL05I.auto_BL05I_shared_beamStatus` | `BL05I_shared_beamStatus.template` |
| `BL05I.auto_BL05I_HR_beamStatus` | `BL05I_HR_beamStatus.template` |
| `BL05I.auto_BL05I_HR_vacuumOpenSequence` | `BL05I_HR_vacuumOpenSequence.template` |
| `BL05I.auto_BL05I_mks937bGauge_pressureOverride` | `BL05I_mks937bGauge_pressureOverride.template` |
| `BL05I.auto_BL05I_slit_BPM_dummies` | `BL05I_slit_BPM_dummies.template` |
| `BL05I.auto_BL05J_Nano_beamStatus` | `BL05J_Nano_beamStatus.template` |
| `BL05I.auto_BL05J_adamCPW_patch` | `BL05J_adamCPW_patch.template` |
| `BL05I.auto_BL05JmanipulatorSlit` | `BL05JmanipulatorSlit.template` |
| `BL05I.auto_BL05JpvRamp` | `BL05JpvRamp.template` |
| `BL05I.auto_motionProtectionMeaning` | `motionProtectionMeaning.template` |

`auto_BL05I_mks937bGauge_pressureOverride`, `auto_BL05I_slit_BPM_dummies` and
`auto_BL05J_adamCPW_patch` change records that another entity loads, so in
`ioc.yaml` they must come after that entity.

## Not included

- `BL05I_changeHexapodMirror` and `BL05I_changeHexapodMirror_withRef` use
  StreamDevice and `changeHexapod.proto`: they belong in ibek-runtime-streamdevice.
- `BL05I_waterchiller` uses StreamDevice with `HostLink.proto` and an asyn record.
- `auto_BL05I-digitelMpcIonpOverride` uses StreamDevice with `digitelMpc.proto`, and
  no i05 IOC uses it.

## Files

[`ibek.manifest.yaml`](ibek.manifest.yaml) vendors the support YAML and the
`*.template` files into an IOC instance's `config/`. This README stays in the library.
