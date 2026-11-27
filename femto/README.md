# femto

Runtime pattern imported from the DLS support module `femto`, release `4-10-1`
(`/dls_sw/prod/R3.14.12.7/support/femto/4-10-1`, GitLab `controls/support/femto`):
gain control and readback records for the Femto DLPCA-200, DHPCA-100 and DDPCA-300
current amplifiers.

Ten of the eleven templates are pristine copies of the release's `db/` folder. They
are the VisualDCT output of `femtoApp/Db/*.vdb`, already expanded, so they are plain
msi templates. Releases 4-9 and 4-10 have the same `db/` folder. The eleventh,
`FluxCalcs.template`, is translated (see [FluxCalcs](#fluxcalcs)).

## Record types and images

The femto library holds only aSub routines (`femtoApp/src/asubFunctions.c`), which
only `FluxCalcs` and `FemtoCurrent` call. With `FluxCalcs` translated and
`FemtoCurrent` left out, the templates use:

| Record type | From |
|---|---|
| `ai`, `ao`, `bi`, `bo`, `calc`, `calcout`, `mbbo`, `stringout` | EPICS base |
| `transform` | calc (`calcSupport.dbd`) |

So any generic IOC image that builds calc can load them, such as `ioc-generic` and
`ioc-streamdevice-runtime`.

## DTYP and Hy8001

The gain, high-speed, AC/DC and overload bits (`bo` records, and the `bi` of
`femto_overload`) take `DTYP` from the `DTYP` parameter, which defaults to `Hy8001`
as in the release: device support for a Hytec VME digital I/O card, which no Linux
generic IOC builds. A Linux IOC sets `DTYP` to `Soft Channel` and points the
`OUT*`/`INPOL` links at the PVs of its remote I/O, as every Linux builder IOC that
uses femto does. The default stays so that the templates match the release; an IOC
that drives a Hy8001 card needs a generic IOC image that builds its device support.

## Entity models

The models and their parameters are those of `femto` in ibek-support-dls, which
follow the classes of the release's `etc/builder.py`, so existing `ioc.yaml` files
validate unchanged.

| Entity model | Template |
|---|---|
| `femto.femto100` | `femto100.template` |
| `femto.femto200` | `femto200.template` |
| `femto.femto300` | `femto300.template` |
| `femto.femto_acdc100` | `femto_acdc100.template` |
| `femto.femto_acdc200` | `femto_acdc200.template` |
| `femto.femto_overload` | `femto_overload.template` |
| `femto.femto_ai` | `femto_ai.template` |
| `femto.femto_scale` | `femto_scale.template` |
| `femto.simulation_femto` | `simulation_femto.template` |
| `femto.simulation_femto_ai` | `simulation_femto_ai.template` |
| `femto.FluxCalcs` | `FluxCalcs.template` (translated) |

The amplifier models and `FluxCalcs` have PVI screens (`*.pvi.device.yaml`), first
generated from the release's EDM screens for ibek-support-dls and maintained by hand
here.

Not included: `FemtoCurrent`, an aSub on the femto library that no IOC uses;
`FemtoFix.template`, which `builder.py` has no class for; and the `generic_BPM_diag`
model of ibek-support-dls/femto, which belongs to the `generic` pattern in this
library.

## FluxCalcs

`FluxCalcs` converts a femto ADC voltage into the photodiode current and the photon
flux. The release's template does it in two aSub routines from
`femtoApp/src/asubFunctions.c`: `bcoeffCompile` gathers the seven responsivity
coefficients `B0`..`B6` into a waveform, and `fluxCalculation` computes the current,
responsivity and flux from it. Here EPICS base records compute the same values from
the same macros, so no compiled code is needed:

| Record | Release | Here | Computes, as in `fluxCalculation()` |
|---|---|---|---|
| `FCALC` | aSub `fluxCalculation`, VAL 0 | calcout, VAL is the current | `femtoCurrent()`: `VOLTAGE / femtoGain[GAINSETTING]`, 0 for a setting outside 0..13; the gains are 10^3..10^9 for 0..6 and 10^5..10^11 for 7..13 |
| `FCURR` | ao | ao, plus `FLNK` to `RESPONSIVITY` | written by `FCALC` |
| `RESPONSIVITY` | ao | calc | `responsivity()`: the sum of `Bi * ENERGY^i` for i = 0..6 |
| `FLUX` | ao | calc | `FCURR / (1000 * 1.60217657e-19 * ENERGY * RESPONSIVITY)`; 0 when the denominator is 0 or the result negative |
| `BCALC`, `BCOEFF` | aSub `bcoeffCompile`, waveform | removed | `RESPONSIVITY` reads `B0`..`B6` directly |
| `B0`..`B6` | ai | ai, unchanged | |

The calc expressions apply the C operations in the same order with the same `pow()`,
so they give the same doubles. `FLUX`, `RESPONSIVITY`, `FCURR` and `B0`..`B6`, the PVs
that `FluxCalcs.edl` and the PVI screen show, keep their names, units and precision.
Comments in the template trace each record to the C.

Provenance is also in the header of [`femto.ibek.support.yaml`](femto.ibek.support.yaml),
which is vendored with the pattern.

## Files

[`ibek.manifest.yaml`](ibek.manifest.yaml) vendors the support YAML and the
`*.template` and `*.pvi.device.yaml` files into an IOC instance's `config/`. This
README stays in the library.
