# catchment-dod-balance

Internal, physics-based validation of a lidar **DEM of Difference (DoD)** by routing
its sediment volume down a flow network and enforcing **sediment continuity (Exner)**.

Each cell's net volume change `V = DoD * area` is accumulated down a D-infinity flow
network to `V_acc` (the upstream-integrated budget). Sediment flux is `Q_out = -V_acc`,
and a channel cannot carry negative sediment, so the physical constraint is

    V_acc <= 0   <=>   cumulative erosion >= cumulative deposition, downstream.

A cell whose `V_acc` climbs positive beyond its error has deposited more than any
upstream erosion can supply -> external input (e.g. an off-map trunk river) or DoD
error. This inverts the "morphological method" (used forward for transport estimation)
into a **quality check / error detector on the DoD itself**, valid where a catchment
sources its sediment locally.

## Install
    pip install -e .[routing,dev]   # routing = richdem (D-infinity + breaching)

## Use
    from catchment_dod_balance import dinf_proportions, mass_balance
    props, valid = dinf_proportions(dem)                 # RichDEM D-infinity + breaching
    out = mass_balance(dod, perror, props, valid, res)   # V_acc, sigma_Vacc, contaminated, surplus

`weighted_accumulation`/`mass_balance` are pure NumPy with the routing injected, so the
core is testable without a router. Error envelope is the independent-error LOWER bound
(omits spatial error correlation) -- treat `surplus` flags as candidates. No bulking.

## Status & the floodplain component
See `docs/mass_balance_handoff.md`: the check false-positives on floodplains of
off-map trunk rivers (overbank deposition is not steered by local drainage). The fix
is handled at the **routing stage, first** (account for the off-map inflow at the tile
entry), not by masking the reported output.

## Provenance
Developed in [MNiMORPH/lidar-diff-icp](https://github.com/MNiMORPH/lidar-diff-icp)
(2008-vs-2021 Minnesota lidar differencing) and extracted here **with its git history**.

## License
GPL-3.0-or-later.
