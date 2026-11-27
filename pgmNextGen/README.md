# pgmNextGen

Runtime pattern imported from the DLS support module `pgmNextGen`, release `4-1`
(GitLab `controls/support/pgmNextGen`): Plane Grating Monochromators on Delta Tau
motion controllers.

`PGM.template`, `AutoDecouple.template` and `pgmNextGenCal.template` are pristine
copies of the module's own templates from `pgmNextGenApp/Db`. pgmNextGen 4-1's
Makefile builds `pgmNextGen.db` and `pgmNextGenCalibration.db` from
`pgmNextGen.substitutions` and `pgmNextGenCalibration.substitutions` with msi. The
expansion that the Makefile and msi did at build time is done per instance by ibek:
each entry in an entity model's `databases:` list is one row of a substitutions file,
in the same order, with the row and its line number quoted in a comment above it. The
files are otherwise pristine.

The substitutions files also load templates from the pmac and positioner support
modules (`pmacVariableWrite.template`, `multipositioner.template`,
`positioner.template` and `motorpositioner.template`). Those come from the generic
IOC image, which must build calc, busy, motor, pmac and positioner, such as ioc-pmac.
Nothing needs compiling. Provenance and the macro mapping are in the header of
[`pgmNextGen.ibek.support.yaml`](pgmNextGen.ibek.support.yaml).

| Entity model | Substitutions file | Templates |
|---|---|---|
| `pgmNextGen.pgmNextGen` | `pgmNextGen.substitutions` | `pmacVariableWrite`, `PGM`, `AutoDecouple`, `multipositioner`, `positioner`, `motorpositioner` |
| `pgmNextGen.pgmNextGenCalibration` | `pgmNextGenCalibration.substitutions` | `pgmNextGenCal`, `multipositioner`, `positioner`, `pmacVariableWrite` |

[`ibek.manifest.yaml`](ibek.manifest.yaml) vendors the support YAML and the three
templates into an IOC instance's `config/`. This README stays in the library.
