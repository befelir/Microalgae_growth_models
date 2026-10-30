# Parameter estimation of a microalgae growth model that predicts biomass concentration at steady state for acclimated cultures as a function of light intensity and temperature

Jupyter notebook that calibrates a mechanistic model of microalgal growth in a continuous culture, using biomass measurements taken at different light intensities and temperatures. The model combines a Haldane-type light response, a cardinal temperature model with an attenuation term for acclimation, and a Lambert–Beer light-attenuation law whose absorption coefficient depends on pigment content. Parameters are estimated with [pyFOOMB](https://github.com/MicroPhen/pyFOOMB).

## Model

Biomass concentration $C_x$ in a continuous culture with dilution rate $D = 1/\tau$ ($\tau = 1.3$ d), averaged over the culture depth $L$:

$$\frac{dC_x}{dt} = \frac{1}{L}\int_0^L \left[\mu(I_z,T) - m_e\right] C_x \, dz \;-\; D\,C_x$$

- **Light (Haldane-type):** $\mu_I(I_z) = \mu_{max}\,\dfrac{I_z}{I_z + K_I\left(1 - I_z/I_{opt}\right)^2}$
- **Temperature (cardinal temperature model):** $\varphi(T)$, with an exponent $n$ that attenuates its effect to account for acclimation: $\mu(I_z,T) = \mu_I(I_z)\,\varphi(T)^{n}$
- **Light attenuation (Lambert–Beer):** $I_z = I_0\,e^{-k_a C_x z}$, where $k_a = k_{a1}\left(\dfrac{a}{b\,T^{k} + I_0} + k_2\right)$

The expression for $k_a$ is empirical, fitted to pigment measurements: the pigment content changes with acclimation, which changes the optical properties of the culture. $a$, $b$, $k$ and $k_2$ were fitted separately and are fixed in this notebook.

Parameters whose names end in `n` (for example `mumaxn`) are dimensionless multipliers applied to a nominal value, and these multipliers are what the optimiser estimates. The full equations are written out at the top of the notebook.

## Data

`data/cx_measurements.csv` contains the biomass measurements used for the fit.

| Column | Meaning |
|---|---|
| `time_d` | Time of sampling (days) |
| `I` | Light intensity (µmol photons m⁻² s⁻¹) |
| `T` | Temperature (°C) |
| `Cx` | Biomass concentration (g m⁻³) |
| `Cx_sd` | Standard deviation of `Cx` |

Design: 4 light levels (150, 300, 600, 900 µmol photons m⁻² s⁻¹) × 4 temperatures (18, 24, 30, 36°C), 10 days per condition, sampled on day 5 of each step.

The measurements come from the author's PhD thesis: Felices Rando B. (2025). *Investigating microalgae behaviour via experimentation in micro-scale photobioreactors*. PhD thesis, University of Padova. https://hdl.handle.net/20.500.14242/195901. The thesis is open access; the repository lists it as "All rights reserved".

`data/cx_measurements.csv` reproduces the values from the thesis so that the analysis in this repository can be repeated. If you reuse the data, please cite the thesis.

## Installation

pyFOOMB depends on `assimulo` and `pygmo`, which are easiest to install from conda-forge.

```
conda env create -f environment.yml
conda activate pyfoomb-env
```

Then open the notebook in Jupyter or VS Code and select the `pyfoomb-env` kernel.

## Usage

Open `parameter_estimation.ipynb` and run the cells in order.

The global estimation takes about 5.6 hours on 16 cores. The outputs saved in the notebook come from that original run; the code was tidied afterwards for readability and was not rerun. To test the workflow quickly, reduce the number of evolutions or islands in the call to `estimate_parallel`.

## Results

Estimated multipliers and uncertainties, as printed in the notebook:

| Parameter | Estimate | Uncertainty |
|---|---|---|
| `mumaxn` | 0.987 | ± 4.3 % |
| `KIn` | 0.900 | ± 18.8 % |
| `Ioptn` | 1.01 | ± 8.5 % |
| `ka1n` | 0.847 | ± 9.8 % |
| `nn` | 0.102 | ± 42.4 % |
| `Tmaxn` | 0.863 | ± 2.7 % |
| `Toptn` | 1.07 | ± 10.7 % |

## Limitations

- 7 parameters are estimated from 16 data points (3 replicates per condition), so some parameters are poorly constrained. The attenuation exponent `nn` is the most uncertain.
- The estimated $T_{max}$ lies close to the highest temperature tested (36°C), so the upper end of the temperature response is not well constrained by the data.
- The empirical expression for $k_a$ is only valid within the range of the pigment measurements it was fitted to (roughly 18–36°C and the light levels used here).

## References

- Hemmerich J, Tenhaef N, Wiechert W, Noack S. pyFOOMB: Python framework for object oriented modelling of bioprocesses. *Engineering in Life Sciences*.
- Bernard O, Rémond B (2012). Validation of a simple model accounting for light and temperature effect on microalgal growth. *Bioresource Technology* 123, 520–527.


## Author

Beatriz Felices Rando

## License

Code released under the MIT License (see `LICENSE`).
