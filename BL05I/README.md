# BL05I

Runtime pattern imported from the DLS support module `BL05I` (the I05 beamline's
support module, which also carries the templates of the J branch, BL05J), release
`2-95` (GitLab `controls/support/BL05I`).

Most of its templates use only soft records: `ai`, `ao`, `bi`, `bo`, `calc`, `calcout`,
`dfanout`, `fanout`, `longout`, `mbbi`, `mbbiDirect`, `mbbo`, `seq`, `stringin`,
`stringout` and `subArray` from EPICS base, `scalcout`, `sseq` and `transform` from
calc, `busy` from busy, and `timestamp` from std (in `BL05I_Helium_recovery.template`
only). They need no compiled code: any generic IOC image that builds calc and busy can
load them, and one that also builds std for `BL05I_He_Recovery`. Three models need more:

- `BL05I_changeHexapodMirror` and `BL05I_changeHexapodMirror_withRef` read and write
  a Delta Tau PLC's variables through the pmac driver, so they need a pmac image such
  as `ioc-pmac` (which also has calc and busy). `PmacController` names the IOC's
  `pmac.GeoBrick` (or `pmac.PMAC`) entity.
- `BL05I_waterchiller` uses StreamDevice with `Hostlink.proto`, and an asyn record,
  so it needs an image with StreamDevice and asyn. The protocol file is not in this
  pattern: vendor the `HostLink` pattern of
  [ibek-runtime-streamdevice](https://github.com/epics-containers/ibek-runtime-streamdevice)
  into the same IOC instance. It puts `Hostlink.proto` in `config/`, and the IOC
  places it on `STREAM_PROTOCOL_PATH` at start-up.

## Files: pristine and translated

Every template is a pristine copy of the release, except the two hexapod mirror
templates, which are translated:

| File | Status |
|---|---|
| `BL05I_changeHexapodMirror.template` | translated from `db/BL05I_changeHexapodMirror.template` |
| `BL05I_changeHexapodMirror_withReference.template` | translated from `db/BL05I_changeHexapodMirror_withReference.template` |
| `BL05I_waterChiller.template` and every other `*.template` | pristine, byte-identical to the release |
| `BL05I_waterChiller_settings.req` | the PVs of the chiller template's autosave tags |
| `*.pvi.device.yaml` | PVI screens for the hexapod mirror and chiller models, maintained by hand |

**Why translated.** In the release, the hexapod mirror records talk to the GeoBrick
through StreamDevice and BL05I's own `changeHexapod.proto`. Here they use the pmac
driver's own variable access instead: generic pmac variable access instead of a
beamline protocol file, decision 2026-11-27. Reads use `PMAC_VI<loop>_<var>` (I/O Intr
records polled by the driver), writes `PMAC_WI_<var>`, and the mirror-change command
`PMAC_C_WRITE_CMD`, all asyn parameters of pmac 2-7-9's controller. Each template's
header says what changed, and each translated record has a `# was:` comment giving its
StreamDevice `DTYP` and `INP`/`OUT`. Only the transport changed: the record logic,
including the defects listed below, is as released.

How each file was obtained is also recorded in the header of
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
| `BL05I.BL05I_changeHexapodMirror` | `BL05I_changeHexapodMirror.template` |
| `BL05I.BL05I_changeHexapodMirror_withRef` | `BL05I_changeHexapodMirror_withReference.template` |
| `BL05I.BL05I_waterchiller` | `BL05I_waterChiller.template` |
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

## Hexapod mirror and water chiller notes

- **Hexapod mirror: `PmacController`** replaces the builder class's `PORT`, which was
  the asyn IP port to the GeoBrick (`pmac.pmacAsynIPPort`). It is now the controller
  entity, and the records use its asyn port, e.g. `BRICK06` rather than
  `BRICK06_Port`. The entities therefore live in the motion IOC that drives the
  hexapod.
- **Hexapod mirror: polling.** The release polled the mirror, state, status and PLC run
  flag every 0.2 s with a `FAN:UPDATE` fanout record, now gone. The pmac driver polls
  them instead: the run flag on its fast loop (the controller's poll period), which
  `BSY:CHECK` needs in order to see the PLC start; the mirror, state and status on its
  medium loop (2 s); and the mirror index variables, which the release read once at
  start-up, on its slow loop (5 s). The records are I/O Intr, so they process when a
  value changes.
- **Hexapod mirror: `YMTR`** names the hexapod's Y motor record, which this pattern
  does not create. Changing mirror writes its soft limits (`.HLM`, `.LLM`) and reads
  its position.
- **Water chiller: `port`** is an asyn IP or serial port to the Omron PLC, which
  talks C-mode HostLink. Set `<sensor>PRES` to `0` for each pressure, flow or
  temperature sensor or valve the system does not have.
- **Water chiller: `WPDESC` and `UVDESC` defaults.** The template gives each of these
  two different defaults in different records; the model's defaults are the longer of
  each pair, so a record that used the shorter one gets the longer description when
  the macro is not set.

## Upstream defects carried as-is

- `BL05I_changeHexapodMirror.template`: the completion sequence processes
  `$(P)$(R):GET:RVB:STATUS`, a record that does not exist (the template has
  `GET:STATUS`), so its forced status refresh does nothing. The `_withReference`
  template has this fixed.
- Both hexapod templates build the change command from `DMD:MIRROR`'s `VAL`, the state
  number, not `RVAL`, the mirror index the PLC reported. The two agree only while the
  PLC numbers its mirrors 1 and 2 (and 3 for the reference).

## Not included

- `auto_BL05I-digitelMpcIonpOverride` uses StreamDevice with `digitelMpc.proto`, and
  no i05 IOC uses it.

## Files

[`ibek.manifest.yaml`](ibek.manifest.yaml) vendors the support YAML and the
`*.template`, `*.req` and `*.pvi.device.yaml` files into an IOC instance's `config/`.
This README stays in the library.
