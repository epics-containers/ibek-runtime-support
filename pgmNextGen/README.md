# pgmNextGen

Runtime pattern imported from the DLS support module `pgmNextGen`, release `4-1`
(GitLab `controls/support/pgmNextGen`): Plane Grating Monochromators on Delta Tau
motion controllers. `pgmNextGen.db` and `pgmNextGenCalibration.db` are pristine
copies of the databases a build of that release installs in `db/` (msi expansions of
its substitutions files, so not in the git tree).

The databases use soft records from EPICS base, calc and busy, soft channel motor
records, and asyn records and `asynFloat64` records on the pmac driver's PMAC
variable parameters. A generic IOC image that builds calc, busy, motor and pmac, such
as ioc-pmac, can load them without compiling. Provenance is in the header of
[`pgmNextGen.ibek.support.yaml`](pgmNextGen.ibek.support.yaml).

| Entity model | Database |
|---|---|
| `pgmNextGen.pgmNextGen` | `pgmNextGen.db` |
| `pgmNextGen.pgmNextGenCalibration` | `pgmNextGenCalibration.db` |

[`ibek.manifest.yaml`](ibek.manifest.yaml) vendors the support YAML and the
databases into an IOC instance's `config/`. This README stays in the library.
