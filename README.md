# SNEC updates in 2026 

Samantha Wu and Tony Piro (Carnegie Observatories)

This directory contains a modified version of SNEC.
The modified code has two main changes:

1. It uses a new bolometric corrections table, `BolCorr2.dat`, in place of `BolCorr.dat`.
2. It calculates a **thermalization temperature**, `T_therm`, and writes it out alongside the effective temperature, `T_eff`.

## 1. Updated bolometric corrections table (`BolCorr2.dat`)

- `src/read_BolCorr.F90` now reads `tables/BolCorr2.dat` instead of `tables/BolCorr.dat`. The original code that read `BolCorr.dat` is still in the file, commented out.
  The new table adds ultraviolet bands (NUV, FUV, M2, W1, W2) and drops the PTF R band.
- `magnitudes.dat` now has 17 columns:

  ```
  time  T_for_BC  u  g  r  i  z  U  B  V  R  I  NUV  FUV  M2  W1  W2
  ```

  This is in contrast to the old 13-column `magnitudes.dat`, where the first band was PTF R.

## 2. Thermalization temperature (`T_therm`)

### Physical motivation

We now calculate the thermalization depth based on where the effective thermalization optical depth approaches 1:
$$
\tau_{\rm eff} = \int_r^{R} \sqrt{\kappa_{\rm abs}\,\kappa_{\rm scat}}\;\rho\, dr \approx 1 .
$$

The gas temperature at this radius, `T_therm`, is used to evaluate the bolometric corrections

```fortran
T_eff_for_BC = MAX(T_therm, T_eff_min)   ! previously MAX(T_eff, T_eff_min)
```

The second column of `magnitudes.dat` is therefore the thermalization temperature, with the same floor as before applied.

The `parameters` file has one new parameter:

```
#time (in seconds) after which the thermalization radius calculation starts
min_time_therm_calc     = 1.0d3
```

### New default output files

| File | Contents |
|---|---|
| `T_therm.dat` | time, thermalization temperature (K) |
| `r_therm.dat` | time, thermalization radius (cm) |
| `index_therm.dat` | time, grid index of the thermalization radius |
| `kappa_abs.xg` | absorption opacity profiles |
| `kappa_scat.xg` | electron scattering opacity profiles |
| `kappa_table.xg` | opacity from the tables, before the floor is applied |
| `ne.xg` | electron number density profiles |
