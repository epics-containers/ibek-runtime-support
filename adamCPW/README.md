# adamCPW

Runtime pattern imported from the DLS support module `adamCPW`, release `1-3-2`
(GitLab `controls/support/adamCPW`): support for the Adam Equipment CPW plus balance,
typically used to weigh liquid cryogen dewars. `adamCPWplus.template` is a pristine
copy of that release.

The template talks to the balance with asyn records over an asyn port the IOC
creates, and otherwise uses `ai`, `ao`, `bo`, `calcout`, `seq` and `stringin` from
EPICS base and `scalcout` from calc. Any generic IOC image that builds asyn and calc
can load it. Provenance is in the header of
[`adamCPW.ibek.support.yaml`](adamCPW.ibek.support.yaml).

| Entity model | Template |
|---|---|
| `adamCPW.adamCPWplus` | `adamCPWplus.template` |

[`ibek.manifest.yaml`](ibek.manifest.yaml) vendors the support YAML and the template
into an IOC instance's `config/`. This README stays in the library.
