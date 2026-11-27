# BLfeedback

Runtime pattern imported from the DLS support module `BLfeedback`, release `2-26`
(GitLab `controls/support/BLfeedback`). It carries only `feedback.template`, a
pristine copy from that release: a PID feedback loop on an `epid` record.

The template uses `ao`, `bo` and `calcout` from EPICS base, `transform` from calc and
`epid` from std. A generic IOC image that builds calc and std can load it without
compiling. The entity model and its parameters are those of `BLfeedback.feedback` in
ibek-support-dls, and `feedback.pvi.device.yaml` comes from there too. Provenance is
in the header of [`BLfeedback.ibek.support.yaml`](BLfeedback.ibek.support.yaml).

| Entity model | Template |
|---|---|
| `BLfeedback.feedback` | `feedback.template` |

Not included: `beamPositionCalc` and `characterise` are asyn port drivers compiled
into the module's library; the other templates have no user being converted yet.

[`ibek.manifest.yaml`](ibek.manifest.yaml) vendors the support YAML, the template and
the PVI device YAML into an IOC instance's `config/`. This README stays in the
library.
