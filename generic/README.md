# generic

Runtime pattern imported from the DLS support module `generic`, release `2-39-7`
(`/dls_sw/prod/R3.14.12.7/support/generic/2-39-7`, GitLab `controls/support/generic`):
soft-record scans with menus of recent positioners, triggers and detectors, and
position calculations for four-quadrant beam position monitors.

The four templates are pristine copies of the release's `db/` folder. They are the
VisualDCT output of `genericApp/Db/*.vdb`, already expanded, so they are plain msi
templates. Every 2-39 release has the same records; the templates differ only in the
paths in their `expand()` comments.

## Record types and images

`generic` has no compiled code: `genericApp/src` has no sources, and `generic.dbd`
only gathers record types. The templates use:

| Record type | From |
|---|---|
| `ai`, `ao`, `bo`, `mbbo`, `stringout`, `waveform` | EPICS base |
| `sscan` | sscan |
| `scalcout`, `sseq`, `transform` | calc (`calcSupport.dbd`) |

So any generic IOC image that builds sscan and calc can load them. Among those that
do: `ioc-generic`, `ioc-pmac` and `ioc-streamdevice` (and so
`ioc-streamdevice-runtime`), as well as most of the areaDetector images,
`ioc-motorsim`, `ioc-dlslinuxvac` and `ioc-pgm`. `ioc-positioner` builds calc but not
sscan, so it can load the BPM models but not the scans. The release's `builder.py`
also lists std as a dependency, but no template uses it.

## Entity models

The models are the classes of the release's `etc/builder.py`, each an
`AutoSubstitution` of one template. The parameter names are the template macros,
which are the builder XML attributes, with the template's defaults.

| Entity model | Template |
|---|---|
| `generic.generic_scan` | `generic-scan.template` |
| `generic.generic_2d_scan` | `generic-2d-scan.template` |
| `generic.generic_BPM_vert` | `generic-BPMvert.template` |
| `generic.generic_BPM_diag` | `generic-BPMdiag.template` |

- `name`, `gda_name`, `gda_desc` and `gda_curr_name` are not declared: the templates
  use them only in gui and GDA tag comments, and builder2ibek drops them.
- The BPM offsets `DA` to `DD` and gains `KX` and `KY` are floats, as in the
  ibek-support-dls models they replace, so their `ao` records get `VAL` `1.0` where
  the builder IOCs had `1`, and `0.0` for `0`: the same value.
- The BPM models have PVI screens, `generic-BPMvert.pvi.device.yaml` and
  `generic-BPMdiag.pvi.device.yaml`, first generated from the release's
  `generic-BPM-setup.edl` and maintained by hand here.

Not included: `generic-9menu.template`, which the scan templates already contain, and
`generic-gda-pv.template`, which has no builder class and no records.

Provenance is also in the header of
[`generic.ibek.support.yaml`](generic.ibek.support.yaml), which is vendored with the
pattern.

## Files

[`ibek.manifest.yaml`](ibek.manifest.yaml) vendors the support YAML and the
`*.template` and `*.pvi.device.yaml` files into an IOC instance's `config/`. This
README stays in the library.
