---
title: Deriving the Black Scholes Pricing Formula
author: Domenico Sauta
date: 2021-10-10 13:36:00 +0800
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

Which can be thought of the sum of a deterministic drift term and a stochastic noise term. Note that if we were working in $P$-measure, the $r$ term (risk-free rate) would be replaced with $\mu$ (expected return). $B_t$ is standard Brownian motion or a Wiener process, with $dB_t$ being normally distributed with mean zero and variance $dt$.

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

## Deriving the Price of a European Call

At expiry $T$, the pay-off function of a call option is defined as follows:

$$c_T = \text{max}(S_T - K, 0)$$

Where $K$ is the strike price of the option. What we require is the value of the option at time $t$, prior to expiry. We can arive at an analytic function for $c_t$ by employing the conditional expectation of $c_T$ in $Q$-measure and discounting back to the present time.

$$c_t = e^{-r\tau} E_t^Q (c_T)$$

To compute this, we will need to make use of the integral definition of expectation.

$$E[X] = \int_\mathbb{R} x f(x) dx$$

$$\therefore c_t = e^{-r\tau}\int_{-\infty}^\infty \text{max}(S_T - K, 0)\frac{1}{\sqrt{2\pi\tau}} e^{- \frac{B_\tau^2}{2\tau}} dB_\tau$$

However, before proceeding we will need to deal with that nasty max function. We can do this by computing for which values of $B_\tau$ will $S_T - K$ drop below 0:

$$S_T - K > 0$$

$$ S_t e^{(r - \frac{1}{2}\sigma^2)\tau + \sigma B_\tau} - K > 0 $$

After some rearranging and cosmetic adjustments:

$$ B_\tau > - \frac{\ln(\frac{S_t}{K}) + (r - \frac{\sigma^2}{2})\tau}{\sigma \sqrt{\tau}} \sqrt{\tau} $$

Which we will define as:

$$B_\tau > -d_2 \sqrt{\tau} $$

Now, our expectation can be simplified to the following:

$$ E_t^Q (c_T) = \int_{-d_2\sqrt{\tau}}^\infty S_t e^{(r - \frac{\sigma^2}{2})\tau + \sigma B_\tau} \frac{1}{\sqrt{2\pi\tau}} e^{- \frac{B_\tau^2}{2\tau}} dB_\tau - K \int_{-d_2\sqrt{\tau}}^\infty \frac{1}{\sqrt{2\pi\tau}} e^{- \frac{B_\tau^2}{2\tau}} dB_\tau$$

Which we will break up into 2 problems as follows:

$$:= \text{I} - \text{II}$$

Dealing with $\text{I}$ first:

$$ \text{I} = S_t e^{(r - \frac{\sigma^2}{2})\tau} \int_{-d^2\sqrt{\tau}}^\infty \frac{e^{\sigma B_\tau}e^{\frac{-B_\tau^2}{2\tau}}}{\sqrt{2\pi\tau}} dB_\tau $$

Completing the square so we can perform intergration:

$$ = S_t e^{(r - \frac{\sigma^2}{2})\tau} e^{\frac{\sigma^2 \tau}{2}} \int_{-d_2\sqrt{\tau}}^\infty \frac{e^{\frac{(B_\tau - t\sigma)^2}{2t}}}{ \sqrt{2\pi\tau}} dB_\tau $$

We are now ready to perform a substitution. Let $y = -\frac{B_\tau - t\sigma}{\sqrt{t}}$.

When $B_\tau = \infty: y = -\infty$, and when $B_\tau = -d_2\sqrt{\tau}: y = d_2 + \sqrt{\tau}\sigma := d_1$. Substituting and flipping bounds:

$$=S_t e^{r\tau} \int_{-\infty}^{d_1} \frac{1}{\sqrt{2\pi}} e^{-\frac{y^2}{2}} dy$$

Which is the definition of the cumulative normal density function $N(X)$.

$$\therefore \text{I} = S_t e^{r\tau}N(d_1)$$

Shifting our attention to take a look at $\text{II}$:

$$\text{II} = K \int_{-d_2\sqrt{\tau}}^\infty \frac{1}{\sqrt{2\pi\tau}} e^{- \frac{B_\tau^2}{2\tau}} dB_\tau$$

Which can be dealt with using the substitution $y = -\frac{B_\tau}{\sqrt{\tau}}$. Substituting and swapping bounds we arive at:

$$ = K\int_{-\infty}^{d_2} \frac{1}{\sqrt{2\pi}} e^{-\frac{y^2}{2}} dy = K\cdot N(d_2)$$

Finally, plugging all of this back into our initial equation we get:

$$c_t = e^{-r\tau} (S_t e^{r\tau}N(d_1) - K\cdot N(d_2))$$

$$\therefore \fbox{$c_t = S_t N(d_1) - Ke^{-r\tau}N(d_2)$}$$

Where $d_1 = \frac{\ln(\frac{S_t}{K}) + (r + \frac{\sigma^2}{2})\tau}{\sigma \sqrt{\tau}}$ and $d_2 = \frac{\ln(\frac{S_t}{K}) + (r - \frac{\sigma^2}{2})\tau}{\sigma \sqrt{\tau}}$.

## Corresponding Put Option Price from Put-Call Parity  

We know that at terminal time $T$, the following relationship must hold by the no-arbitrage principle:

$$ c_T - p_T = S_T - K $$

In order to discount this back to the present time $t$, we must once again take the discounted conditional expectation in $Q$-measure:

$$ e^{-r\tau} E_t^Q(c_T - p_T) = e^{-r\tau}E_t^Q(S_T) - Ke^{-r\tau} $$

$$ c_t - p_t =  e^{-r\tau}E_t^Q(S_t e^{(r - \frac{1}{2}\sigma^2)\tau + \sigma B_\tau}) - Ke^{-r\tau} $$

$$ = S_t - Ke^{-r\tau} $$

Rearranging for $p_t$:

$$p_t = Ke^{-r\tau}(1 - N(d_2)) - S_t(1 - N(d_1))$$

But, by the symmetry of the normal cumulative density function:

$$N(-x) + N(x) = 1 $$

$$\therefore 1 - N(x) = N(-x)$$

Therefore we can simplify the expression to the following:

$$\therefore \fbox{$p_t = Ke^{-r\tau} N(-d_2) - S_tN(-d_1)$}$$
