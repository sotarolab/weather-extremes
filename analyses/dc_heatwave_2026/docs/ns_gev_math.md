# Non-Stationary GEV — Math Reference

## 1. The GEV Distribution

The Generalized Extreme Value distribution models the maximum of a large sample. Its PDF is:

$$f(x; \mu, \sigma, \xi) = \frac{1}{\sigma} \left[1 + \xi \frac{x - \mu}{\sigma}\right]^{-1/\xi - 1} \exp\left(-\left[1 + \xi \frac{x - \mu}{\sigma}\right]^{-1/\xi}\right)$$

defined where $1 + \xi(x - \mu)/\sigma > 0$, with three parameters:

| Parameter | Symbol | Role |
|-----------|--------|------|
| Location | $\mu$ | Shifts the distribution left/right — roughly the "typical" annual maximum |
| Scale | $\sigma > 0$ | Controls spread — year-to-year variability |
| Shape | $\xi$ | Controls tail behaviour — finite upper bound ($\xi < 0$), exponential ($\xi = 0$), heavy tail ($\xi > 0$) |

**scipy sign convention:** `genextreme(c, loc, scale)` uses `c = ξ` with the sign already handled internally. The DC fit gives `c ≈ -0.27` → finite upper bound (Weibull family).

---

## 2. Stationary vs Non-Stationary

**Stationary GEV:** all three parameters are constant in time.

$$x_t \sim \text{GEV}(\mu, \sigma, \xi) \quad \forall t$$

**Non-stationary GEV (NS-GEV):** the location parameter varies with a covariate — here, global mean surface temperature (GMST):

$$\mu(t) = \mu_0 + \alpha \cdot \text{GMST}(t)$$

$$x_t \sim \text{GEV}(\mu(t), \sigma, \xi)$$

$\sigma$ and $\xi$ remain stationary — we assume warming shifts the distribution but doesn't change its spread or tail shape. This is the World Weather Attribution (WWA) standard protocol.

**Physical interpretation of $\alpha$:** for every 1°C of global warming, the location of the DC JJA maximum distribution shifts by $\alpha$ degrees. If $\alpha = 1.4$, then 1.2°C of warming since pre-industrial shifts $\mu$ by $1.4 \times 1.2 = 1.68$°C.

---

## 3. Maximum Likelihood Estimation

Given $n$ observations $\{x_1, \ldots, x_n\}$ paired with GMST values $\{g_1, \ldots, g_n\}$, the log-likelihood is:

$$\ell(\mu_0, \alpha, \sigma, \xi) = \sum_{t=1}^{n} \log f\!\left(x_t \,\middle|\, \mu_0 + \alpha g_t,\, \sigma,\, \xi\right)$$

We maximize $\ell$ — equivalently, minimize the **negative log-likelihood (NLL)**:

$$\hat{\theta} = \arg\min_{\mu_0,\, \alpha,\, \sigma,\, \xi} \left[ -\sum_{t=1}^{n} \log f(x_t \mid \mu_0 + \alpha g_t, \sigma, \xi) \right]$$

**Why log?** The full likelihood is a product of $n$ probabilities, which underflows to zero numerically. The log converts it to a sum, and the argmin is unchanged since log is monotone.

---

## 4. The Log-Transform on $\sigma$

The optimizer (`scipy.optimize.minimize`) searches over all of $\mathbb{R}^4$ by default — it can try negative values of $\sigma$, which are physically impossible and make the GEV undefined.

The fix: optimize $\log \sigma$ instead of $\sigma$ directly.

$$\sigma = e^{\theta_3}, \quad \theta_3 \in \mathbb{R}$$

Since $e^{\theta_3} > 0$ always, the optimizer is free to search anywhere and $\sigma$ stays positive. We recover $\sigma$ at the end with `np.exp(log_sigma)`.

---

## 5. Attribution — PR and FAR

Once the NS-GEV is fit, we compare two counterfactual worlds for a given event threshold $x^*$ (the GFS forecast peak):

| World | GMST | Location |
|-------|------|----------|
| Factual | $g_\text{2024}$ | $\mu_0 + \alpha \cdot g_\text{2024}$ |
| Pre-industrial | $g_\text{PI} = \overline{g_{1880-1900}}$ | $\mu_0 + \alpha \cdot g_\text{PI}$ |

**Exceedance probabilities:**

$$p_1 = P(X > x^* \mid \text{factual}) = 1 - F(x^*; \mu_\text{factual}, \sigma, \xi)$$

$$p_0 = P(X > x^* \mid \text{pre-industrial}) = 1 - F(x^*; \mu_\text{PI}, \sigma, \xi)$$

**Probability Ratio (PR):**

$$\text{PR} = \frac{p_1}{p_0}$$

PR = 1 → no change. PR = 5 → the event is 5× more likely today than pre-industrially.

**Fraction Attributable Risk (FAR):**

$$\text{FAR} = 1 - \frac{1}{\text{PR}} = 1 - \frac{p_0}{p_1}$$

FAR = 0.8 → 80% of the risk of this event is attributable to anthropogenic warming. The algebraic relationship: $\text{PR} = 1/(1 - \text{FAR})$.

---

## 6. Bootstrap Confidence Intervals on PR and FAR

The NS-GEV parameters have sampling uncertainty (only 53 years of data). We propagate that uncertainty to PR and FAR by bootstrapping:

1. Resample $(x_t, g_t)$ pairs with replacement — keep GMST and temperature paired
2. Refit the NS-GEV to the resampled data
3. Recompute PR and FAR from the refit parameters
4. Repeat 500 times; take the 2.5th and 97.5th percentiles as the 95% CI

This captures how much the attribution result would vary if we had observed a different 53-year record drawn from the same true distribution.