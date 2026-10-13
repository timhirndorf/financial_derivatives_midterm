# Midterm Results – Draft for the Presentation

**Setup:** European call on a non-dividend-paying stock: $S_0 = 100$, $K = 100$, $T = 1$ year, $\sigma = 20\%$, risk-free rate $r = 3.94\%$ (1-month US Treasury), stock follows a geometric Brownian motion (GBM).

Each section first gives the **results**, then the **intuition (logic)**, then the **mathematical reasoning**.

---

## Part 1 – Option Value

### Results

| Method | Settings | Call price | Error vs. BS | Uncertainty |
|---|---|---|---|---|
| Black-Scholes | closed form | **9.8940** | – | none (exact) |
| Monte Carlo | $M = 10^8$ paths | 9.8959 | +0.0020 | SE = 0.0014, 95% CI [9.8931, 9.8988] |
| Binomial tree (CRR) | $N = 10{,}000$ steps | 9.8938 | −0.0002 | deterministic |
| Finite differences (Crank-Nicolson) | $400 \times 400$ grid | 9.8915 | −0.0025 | deterministic |

**Sanity check:** put prices show the same picture (BS put = 6.0306). Tree and grid have *exactly* the same error for call and put → they satisfy put-call parity $C - P = S_0 - Ke^{-rT}$ exactly, just like Black-Scholes.

All three numerical methods converge to the Black-Scholes price. The Black-Scholes price lies inside the Monte-Carlo 95% confidence interval.

### Why do the methods agree? (logic)

All three methods compute the same thing: **the discounted expected payoff under the risk-neutral measure**. They only differ in *how* they compute the expectation:

- **Monte Carlo** *samples* random future stock prices and averages the payoffs.
- **The tree** *enumerates* all outcomes on a discrete up/down lattice with funny probabilities.
- **The finite-difference grid** solves the Black-Scholes *PDE* on a price × time grid, backwards from the payoff.

Since all three are approximations of the same quantity, they must converge to the same number.

**Math:**

$$C = e^{-rT}\,\mathbb{E}^{\mathbb{Q}}\left[\max(S_T - K, 0)\right], \qquad S_T = S_0\, e^{(r - \frac{\sigma^2}{2})T + \sigma\sqrt{T} Z}$$

By the Feynman-Kac theorem this expectation is the solution of the Black-Scholes PDE $\frac{\partial V}{\partial t} + \frac{1}{2}\sigma^2 S^2 \frac{\partial^2 V}{\partial S^2} + rS\frac{\partial V}{\partial S} - rV = 0$ – so "expectation" (MC, tree) and "PDE" (grid) are two views of the same problem.

### Convergence

| Method | Error shrinks like | Slope on log-log plot | 10× more effort → |
|---|---|---|---|
| Monte Carlo | $1/\sqrt{M}$ | −½ | ~3.2× smaller error |
| Binomial tree | $1/N$ | −1 | 10× smaller error |
| Finite differences (CN) | $1/N^2$ | −2 | 100× smaller error |

*(Show the three log-log plots side by side – a straight line with slope $-k$ means error $\propto N^{-k}$.)*

#### Monte Carlo

**Logic:** each path is one random "draw" of the future. With few paths, a handful of lucky or unlucky paths move the average a lot. Adding paths averages out the noise, but slowly: every new path adds less information than the previous one.

**Math:** the estimator is a sample mean of $M$ independent discounted payoffs $X_i$. By the law of large numbers it converges to the true expectation, and by the central limit theorem its error is approximately normal with standard deviation

$$\text{SE} = \frac{\text{std}(X)}{\sqrt{M}}$$

→ halving the error needs **4×** the paths. Our numbers: SE $\approx 1.0$ at $M = 100$, $\approx 0.014$ at $M = 10^6$, $\approx 0.0014$ at $M = 10^8$.

#### Binomial tree

**Logic:** the tree replaces the continuous stock movement by discrete up/down steps. With more steps, the lattice gets finer and the discrete distribution of $S_T$ approaches the continuous lognormal one. The price **zig-zags** around Black-Scholes, because – depending on whether $N$ is odd or even – the strike $K = 100$ falls on a node or between two nodes, so the payoff kink is captured differently.

**Math:** CRR parameters $u = e^{\sigma\sqrt{\Delta t}}$, $d = 1/u$, $p = \frac{e^{r\Delta t} - d}{u - d}$ match the mean and variance of the GBM over each step. By the central limit theorem the binomial distribution of $\ln S_T$ converges to the normal distribution. The error is $O(\Delta t) = O(1/N)$; the non-smooth payoff (kink at $K$) is what causes the oscillating error term. Our numbers: error ≈ 1.8 at $N = 1$, 0.02 at $N = 100$, 0.002 at $N = 1{,}000$, 0.0002 at $N = 10{,}000$.

#### Finite differences (Crank-Nicolson)

**Logic:** the grid approximates the derivatives in the PDE by differences between neighbouring grid points. Finer grid → better approximation of the derivatives. Crank-Nicolson averages the derivatives over the old and new time step, which cancels the leading error term – that's why it converges faster than the tree.

**Math:** central differences $\frac{V_{i+1} - V_{i-1}}{2\Delta S}$ and $\frac{V_{i+1} - 2V_i + V_{i-1}}{\Delta S^2}$ have a truncation error $O(\Delta S^2)$ (Taylor expansion: the odd terms cancel). Crank-Nicolson is centred in time, so its time error is $O(\Delta t^2)$. Total error $O(\Delta S^2 + \Delta t^2)$ → with $N_S = N_t = N$: error $\propto 1/N^2$. Our numbers: doubling the grid divides the error by exactly 4 (1.22 → 0.26 → 0.062 → 0.015 → 0.0038 → 0.00096 → 0.00024).

### Uncertainty in the numerical results

- **Monte Carlo – statistical error (random):** a different seed gives a different price. We can only give a confidence interval: $\hat{C} \pm 1.96\,\text{SE}$. The error is *unbiased* – on average MC is right.
- **Tree and grid – discretisation error (deterministic):** the same settings always give the same price; there is no randomness, but there is a systematic bias that shrinks as the grid gets finer. Its size is known from the convergence rate, not from a confidence interval.
- **Additional grid error:** truncating the price axis at $S_{max} = 4K$ and the boundary conditions there (negligible in our case).
- **Model uncertainty (bigger than all numerical errors!):** all four numbers assume GBM with constant $\sigma = 20\%$ and constant $r$. A change of $\sigma$ by 1 percentage point changes the price by ~0.38 (vega) – about 200× the numerical error of our MC estimate.

### Which method is best here?

For this simple European option: **Black-Scholes** (exact). Among the numerical methods, **Crank-Nicolson** converges fastest, then the **tree**, then **Monte Carlo**. MC's advantage is flexibility: it handles path-dependent payoffs and many underlyings, where trees and grids become very expensive (the "curse of dimensionality").

---

## Part 2 – Replicating Portfolio (Delta Hedging)

### Setup

We sell the call for its BS price $C_0 = 9.894$, buy $\Delta = N(d_1)$ shares, and put the rest in the bank. At each rebalancing date we adjust to the new delta, financed by the bank account (self-financing). The stock is simulated under the **real-world** measure (drift $\mu$, here $\mu = r = 3.94\%$; results for other $\mu$ below). At expiry we record

$$\varepsilon = \Pi_T - \max(S_T - K, 0)$$

($\varepsilon > 0$: portfolio worth more than the payoff, $\varepsilon < 0$: shortfall).

### Results

**Base case – daily rebalancing (252 dates), 10,000 paths:** mean error = 0.002, std = 0.44 (≈ 4.4% of the option premium).

| Rebalancing dates | 4 | 12 | 52 | 252 | 1000 |
|---|---|---|---|---|---|
| Std of error | 3.22 | 1.94 | 0.95 | 0.44 | 0.22 |
| Mean error | 0.06 | 0.01 | 0.01 | 0.01 | 0.00 |

| Real-world drift μ | 0% | 5% | 10% | 20% |
|---|---|---|---|---|
| Mean error | 0.006 | 0.008 | 0.009 | −0.001 |
| Std of error | 0.44 | 0.44 | 0.43 | 0.39 |

| True volatility (we hedge with 20%) | 10% | 15% | 20% | 25% | 30% |
|---|---|---|---|---|---|
| Mean error | +3.90 | +1.97 | 0.01 | −1.98 | −3.97 |
| Std of error | 1.05 | 0.73 | 0.44 | 1.00 | 1.92 |

### What do the differences depend on?

**Logic:**
The delta hedge makes the portfolio insensitive to *small* stock moves. But the call is **convex** (curved) in $S$ while the stock position is **linear**. Between two rebalancing dates, the stock moves by a finite amount and the straight line of the hedge cannot follow the curve of the option. How big that mismatch is depends on:

1. **How often we rebalance** – the longer between trades, the bigger the moves we miss.
2. **How curved the option is** (gamma) – largest near the strike and close to expiry.
3. **How much the stock actually moves** compared with the 20% volatility we priced and hedged with.

It does **not** depend on the drift $\mu$: the hedge removes the exposure to the *direction* of the stock.

**Math:** over one step, the hedged portfolio (long $\Delta$ shares + cash, minus the option) earns

$$d\varepsilon \approx \frac{1}{2}\,\Gamma_t\,S_t^2\left(\sigma^2\,\Delta t - \left(\frac{\Delta S_t}{S_t}\right)^2\right)$$

- The term in brackets is "variance we were paid for" minus "variance that actually happened". With the correct $\sigma$ its expectation is zero → **mean error ≈ 0**, but it is random in each step → **noise**.
- Summing $N$ independent steps: each step contributes noise of order $\Delta t = T/N$, so the total variance is $N \cdot O(\Delta t^2) = O(1/N)$ → **std of error ∝ $1/\sqrt{N}$**. Our table: 4× more rebalancing → ~2× smaller std.
- The weight $\Gamma_t S_t^2$ is largest when $S_t \approx K$ near expiry → paths that end near the strike have the largest errors (see scatter plot $\varepsilon$ vs. $S_T$).
- $\mu$ does not appear in the formula → drift does not matter (confirmed: mean ≈ 0 for all $\mu$).
- If the true vol $\sigma_{real} \ne \sigma$: the bracket has non-zero expectation $\left(\sigma^2 - \sigma_{real}^2\right)\Delta t$ → a **systematic** profit or loss, approximately $\frac{1}{2}\int \Gamma S^2 (\sigma^2 - \sigma_{real}^2)\,dt$. Our table: about ±2 per 5 vol points (≈ 20% of the premium).

### When does the replication work well, and when not?

**Works well:**
- Frequent rebalancing (continuous hedging → error → 0, the Black-Scholes argument).
- The volatility used for hedging equals the true volatility.
- Paths that end far from the strike (payoff almost linear → easy to hedge with stock).
- The model assumptions hold: GBM, constant $\sigma$ and $r$, no jumps, no transaction costs.

**Does not work well:**
- **Infrequent rebalancing:** quarterly rebalancing → std 3.2, about a third of the premium.
- **Wrong volatility:** the error is no longer noise around zero but a systematic loss (true vol > hedging vol) or gain (true vol < hedging vol). **This is the most important risk** – a delta hedger is effectively betting on realised vs. implied volatility.
- **Stock near the strike close to expiry:** delta jumps between 0 and 1, gamma is huge, every move costs.
- **In reality additionally:** transaction costs (more rebalancing = more costs → trade-off with the $1/\sqrt{N}$ improvement), price jumps (cannot be hedged with delta at all), stochastic volatility.

---

## Part 3 – Greeks

### Results

| | Black-Scholes | Monte Carlo ($10^6$ paths) | MC std. error | Tree ($N = 1000$) | Tree error |
|---|---|---|---|---|---|
| **Delta** | 0.6168 | 0.6167 | 0.0006 | 0.6167 | −0.00003 |
| **Gamma** | 0.01909 | 0.01903 | 0.00011 | 0.01910 | +0.00002 |
| **Vega** | 38.17 | 38.19 | 0.08 | 38.16 | −0.01 |

Vega is per 1.00 of volatility → per volatility point (20% → 21%): **0.38**.
All MC estimates are within 0.5 standard errors of Black-Scholes; the tree is accurate to 4–5 digits for delta and gamma.

### Interpretation of the values

- **Delta = 0.62:** a 1 increase in the stock raises the call by ~0.62; to hedge one call, hold 0.62 shares. It is **not 0.5 at the money**: $d_1 = \frac{(r + \sigma^2/2)T}{\sigma\sqrt{T}} = 0.297 > 0$, because (1) the forward $S_0 e^{rT} \approx 104 > K$, so the option is in the money relative to the forward, and (2) the lognormal distribution is right-skewed. Delta → 0.5 only for short maturities. Also: delta ≠ probability of exercise ($N(d_2) = 0.54$).
- **Gamma = 0.019:** delta changes by 0.019 per 1 move in the stock. Gamma is highest near the strike → this is where hedging is hardest (Part 2).
- **Vega = 38.2:** the option is very sensitive to volatility – a 1 vol-point error changes the price by 0.38, far more than any of our numerical errors.

### Why do the methods differ? (logic)

- **Black-Scholes:** exact derivatives of the formula – no error.
- **Tree:** delta and gamma come *for free* from the first nodes – the tree has already priced the option at the neighbouring stock prices $S_0u$, $S_0d$, $S_0u^2$, … Only vega needs extra trees (σ is not a dimension of the tree). Errors are deterministic and shrink like the tree price, $\propto 1/N$.
- **Monte Carlo:** there is no derivative to read off, so we **bump and reprice**: price at $S_0 \pm h$, compare. The Greeks are much noisier than the price, especially gamma, because we are measuring *differences* of noisy numbers. Using the **same random numbers** for all bumped prices (common random numbers) is essential – then the noise mostly cancels.

### Math

**Monte Carlo** (central differences, bump $h$):

$$\Delta \approx \frac{C(S_0 + h) - C(S_0 - h)}{2h}, \qquad \Gamma \approx \frac{C(S_0 + h) - 2C(S_0) + C(S_0 - h)}{h^2}$$

Two error sources, which pull in opposite directions:
- **Bias** from the finite difference (Taylor expansion): $O(h^2)$ → large $h$ is inaccurate.
- **Variance:** for gamma, only paths that end within $\pm h$ of the strike contribute, and their contribution is divided by $h^2$ → $\text{Var} \propto 1/h$ → small $h$ is noisy.

→ **bias-variance trade-off**: we chose $h = 1$ (1% of $S_0$). Plot "MC gamma vs. bump size" shows this. For fixed $h$ all MC Greeks converge like $1/\sqrt{M}$, but gamma has ~6× the relative error of delta.

**Tree:** delta = slope between the two nodes at step 1, gamma = change of slope between the three nodes at step 2. These are first/second differences on the tree's own grid (spacing $\propto \sigma\sqrt{\Delta t}$, which shrinks as $N$ grows) → error $\propto 1/N$ for all three Greeks (log-log plot, slope −1). Small detail: they are measured at $t = \Delta t$ and $2\Delta t$ instead of $t = 0$, which vanishes for large $N$.

### Comments / conclusion

1. All three methods agree – Black-Scholes is the benchmark and the others converge to it.
2. **Tree:** best numerical method here – very accurate delta and gamma from one tree, cheap.
3. **Monte Carlo:** most flexible, but the Greeks are noisy (gamma worst) and need a careful choice of bump size and common random numbers. Better MC Greek estimators exist (pathwise derivative, likelihood ratio method).
4. Gamma and vega both peak near the strike – this connects to Part 2: the hedging error comes from $\Gamma$ times the volatility mismatch, and vega tells us how expensive a wrong volatility assumption is.
