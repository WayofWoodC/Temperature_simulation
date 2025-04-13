
# Temperature Simulation with Ornstein-Uhlenbeck Process and Seasonal Trend

This project simulates daily temperature data using a combination of **seasonal modeling** and the **Ornstein-Uhlenbeck (OU) process**. The model is calibrated using 2023 weather data for Baltimore, and its simulation results are compared against the actual observed data from 2024.

---

## 📁 Data

The required input is a CSV file:
```
Baltimore 2023-01-01 to 2024-12-31.csv
```

The file should contain columns:
- `datetime`: daily date string
- `tempmax`: maximum daily temperature
- `tempmin`: minimum daily temperature

The average temperature per day is computed as:

```python
T = (tempmax + tempmin) / 2
```

---

## 📈 Methodology

### 1. Decomposition Model

We assume the observed temperature can be decomposed into a deterministic seasonal component and a stochastic residual component:

$$
T(t) = T_{\text{seasonal}}(t) + T_{\text{residual}}(t)
$$

---

### 2. Seasonal Trend Estimation

The seasonal trend is modeled using a low-order Fourier series:

$$
T_{\text{seasonal}}(t) = a_0 + a_1 \sin\left(\frac{2\pi t}{365}\right) + a_2 \cos\left(\frac{2\pi t}{365}\right)
$$

The parameters \(a_0, a_1, a_2\) are estimated using least squares regression.

---

### 3. Ornstein-Uhlenbeck (OU) Model for Residuals

The residuals are modeled as a discretized OU process:

$$
S_{i+1} = S_i e^{-\lambda \delta} + \mu (1 - e^{-\lambda \delta}) + \sigma \sqrt{\frac{1 - e^{-2\lambda \delta}}{2\lambda}} \cdot Z_i
$$

This is equivalent to a linear form for estimation via regression:

$$
S_{i+1} = a S_i + b + \epsilon_i
$$

Where:

- \( a = e^{-\lambda \delta} \)
- \( b = \mu (1 - e^{-\lambda \delta}) \)
- \( \epsilon_i \sim \mathcal{N}(0, \text{sd}(\epsilon)^2) \)

and:

$$
\text{sd}(\epsilon) = \sigma \sqrt{\frac{1 - e^{-2\lambda \delta}}{2\lambda}}
$$

---

### 4. Parameter Estimation (via Least Squares)

From the estimated regression coefficients \(a\), \(b\), and the standard deviation of residuals \(\text{sd}(\epsilon)\), the OU parameters are recovered using:

$$
\lambda = -\frac{\ln a}{\delta}
$$

$$
\mu = \frac{b}{1 - a}
$$

$$
\sigma = \text{sd}(\epsilon) \cdot \sqrt{\frac{-2 \ln a}{\delta (1 - a^2)}}
$$

---

### 5. Simulation

After estimating parameters, we simulate the residuals using the OU process and combine them with the seasonal trend:

$$
T_{\text{sim}}(t) = T_{\text{seasonal}}(t) + T_{\text{resid,sim}}(t)
$$

---

### 6. Evaluation Metrics

To evaluate model performance on the test data, we use:

- **Mean Bias Error (MBE)**
- **Root Mean Square Error (RMSE)**
- **Mean Absolute Error (MAE)**
- **R-squared (R²)**

---

## 📊 Output

- Estimated OU parameters
- Plots comparing observed and simulated temperatures
- Quantitative error metrics

---

## 🧠 Notes

- The model assumes Δt = 1 day.
- Seasonal component is annual (365-day periodicity).
- Simulation focuses on one-year ahead forecasting for validation.
