---
title: Deriving the Black Scholes Pricing Formula
author: Domenico Sauta
date: 2021-09-11 13:36:00 +0800
categories: [Options Pricing]
tags: [options, trading]
math: true
mermaid: true
---

The Black-Scholes options pricing formula (Black & Scholes, 1973) is one of the most profound results in financial derivative pricing history. In today's post, I am going to demonstrate a way to derive the price of a European call option using risk-neutral conditional expectation in $Q$-measure.

The Black-Scholes formula is a solution to the following partial differential equation:

$$
\frac{\partial c}{\partial t} + \frac{1}{2} \sigma^2 S^2 \frac{\partial^2 c}{\partial S^2} + r S \frac{\partial c}{\partial S} - rc = 0
$$

Which is known as the *Black-Scholes Equation*. This can be derived by constructing a Delta-Hedged option portfolio, but that is a problem for another post.

---

## Stochastic Process for Stock Price

In Black and Scholes (1973), stock price is modeled using the following stochastic process:

$$ dS_t = r S_t dt + \sigma S_t dB_t $$

Which can be thought of the sum of a deterministic drift term and a stochastic noise term. Note that if we were working in $P$-measure, the $r$ term (risk-free rate) would be replaced with $\mu$ (expected return). $B_t$ is standard Brownian motion or a Wiener process, with $dB_t$ being normally distributed with mean zero and variance dt.

Let us commence converting from differential form to a slightly more useful integral form. First, we will make use of a key result from Ito's calculus for expressing the infinitessimal of any stochastic process $X_t(S_t, t)$:

$$
dX_t = \partial_S X_t dS_t + \partial_t X_t dS_t + \frac{1}{2} \partial^2_S X_t (dS_t)^2
$$

$$ (dS_t)^2 = (r S_t dt)^2 + 2(r S_t dt)(\sigma S_t dB_t) + (\sigma S_t dB_t)^2 $$

However, the first and second term vanish as $dt^2$ and $dtdB_t$ are sufficiently small. By Ito's Lemma, the final term becomes:

$$= \sigma^2 S_t^2 dt $$

Let our stochastic process $X_t = \ln S_t$:

$$d(\ln S_t) = \frac{1}{S_t} dS_t + 0 - \frac{1}{2} \frac{1}{S_t^2} \sigma^2 S_t^2 dt$$

Substituting and cancelling:

$$ = (r - \frac{1}{2}\sigma^2)dt + \sigma dB_t $$

Integrating both sides from $t$ to $T$ and letting $\tau \equiv T - t$:

$$\int_t^T d\ln S_u = \int_t^T (r- \frac{1}{2} \sigma^2) du + \int_t^T \sigma dB_u$$

$$\ln(\frac{S_T}{S_t}) = (r - \frac{1}{2}\sigma^2)\tau + \sigma dB_\tau$$

Exponentiating and rearranging:

$$\fbox{$S_T = S_t e^{(r - \frac{1}{2}\sigma^2)\tau + \sigma B_\tau} $}$$
