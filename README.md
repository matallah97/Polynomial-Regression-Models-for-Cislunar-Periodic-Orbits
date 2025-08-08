# Polynomial Regression Models for Cislunar Periodic Orbits

A comprehensive dataset of polynomial coefficients for representing periodic solutions in the Earth-Moon system.

---

## 1. Overview

These MAT-files contain the polynomial coefficients for Polynomial Regression Models (PRMs) that represent the initial states of several periodic orbit families in the Earth-Moon Circular Restricted Three-Body Problem (CR3BP).

The models are trained using data from the renowned **JPL Three-Body Periodic Orbit Catalog**. They allow for the rapid and accurate computation of an initial state vector for a given orbit within a family, eliminating the need to consult a large lookup table.

> #### System Parameters
>
> **Mass Parameter ($\mu$):** 0.012150585609624

---

## 2. Model Structure

For each orbit family, the parameter domain (defined by a parameter $\kappa$) is divided into **10 distinct regions**. The initial state data within each region is then fit to a **sixth-order polynomial**.

The data is organized into individual MAT-files, one for each orbit family.

#### Orbit Families Included:

* Lyapunov Orbits near L1 (`LO1_coeffs.mat`)
* Lyapunov Orbits near L2 (`LO2_coeffs.mat`)
* Halo Orbits near L1 (`HO1_coeffs.mat`)
* Halo Orbits near L2 (`HO2_coeffs.mat`)
* Vertical Orbits near L1 (`VO1_coeffs.mat`)
* Vertical Orbits near L2 (`VO2_coeffs.mat`)
* Distant Retrograde Orbits (`DRO_coeffs.mat`)

---

## 3. Data Structure in the MAT-file

Each MAT-file contains the following arrays:

**kappaLim**: An 11x1 array defining the boundaries of the 10 regions for the family-specific parameter $\kappa$. `kappaLim(i)` is the lower bound of region i, and `kappaLim(i+1)` is the upper bound.

**evalPt_vec**: A 10x1 array containing the operating point (average $\kappa$ value) for each of the 10 regions.

**xPoly**: A 10x7 array where each row contains the polynomial coefficients $[C_0, C_1, ..., C_6]$ for the initial x-component in the corresponding region.

**dyPoly**: A 10x7 array where each row contains the polynomial coefficients $[C_0, C_1, ..., C_6]$ for the initial vy-component ($\dot{y}$) in the corresponding region.

**zPoly** (if applicable): A 10x7 array for the initial z-component. This field exists for non-planar orbits (Halo, Vertical).

**dzPoly** (if applicable): A 10x7 array for the initial vz-component ($\dot{z}$). This field exists for non-planar orbits (Halo, Vertical).

> **Note:** These models represent the initial state at a specific crossing plane (e.g., the x-z plane where y=0). The state components not included in the model (e.g., y, $\dot{x}$, $\dot{z}$ for planar orbits) are assumed to be zero at this initial point.

---

## 4. How to Use the Data (Example in MATLAB)

Here is a simple workflow to compute the initial state vector $x_0$ for a given orbit parameter $\kappa$.

```matlab
% 1. Load the data file
load('HO1_coeffs.mat'); % Example for Halo L1

% 2. Define the desired orbit parameter
kappa = 0.3; % Example parameter value

% 3. Find the correct region for the given kappa
regionIndex = find(kappa >= kappaLim(1:end-1) & kappa < kappaLim(2:end));

if isempty(regionIndex)
    error('Kappa value is outside the defined domain.');
end

kappa_hat = evalPt_vec(regionIndex);
dKappa = kappa - kappa_hat;

% 4. Extract the coefficient rows for that region
coeffs_x  = xPoly(regionIndex, :);
coeffs_dy = dyPoly(regionIndex, :);
coeffs_z  = zPoly(regionIndex, :); % Assuming Halo orbit

% 5. Evaluate the polynomials to get the initial state values
% NOTE: The coefficients are stored [C0, ..., C6]. MATLAB's polyval
% requires coefficients from highest power to lowest, so we must flip them.
x_val  = polyval(fliplr(coeffs_x),  dKappa);
dy_val = polyval(fliplr(coeffs_dy), dKappa);
z_val  = polyval(fliplr(coeffs_z),  dKappa);

% 6. Construct the full initial state vector
% For Halo orbits starting on the x-z plane, y = x_dot = z_dot = 0.
x0 = [x_val; 0; z_val; 0; dy_val; 0];

disp('Computed Initial State Vector:');
disp(x0);

```
