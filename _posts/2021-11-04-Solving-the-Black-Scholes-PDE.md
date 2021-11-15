---
title: Solving the Black Scholes PDE
author: Domenico Sauta
date: 2021-11-04 13:36:00 +0800
categories: [Options Pricing]
tags: [pde, options]
math: true
mermaid: true

---

In this post, I intend to step through the Black Scholes (1973) options pricing model derivation from start to finish, in a complete and accessible way. In a previous post, I explored a way to derive the pricing model using stochastic calculus and risk neutral expectation. This time I will take a more 'applied mathematics approach' by deriving the Black Scholes PDE, transforming it into a more recognizable heat equation form, and solving it using Green's functions.

---

# The Black Scholes Equation

As in previous posts, we will use the following stochastic model for stock price:

$$\frac{dS_t}{S_t} = rdt + \sigma dB_t$$

With $dS_t^2 = S_t^2 \sigma^2 dt$ (a result of Ito's Lemma).

We are interested in constructing a risk-free delta hedged portfolio:

$$\pi = c_t - \Delta S$$

Where:
- The portfolio consists of one option $c_t$, and $\Delta$ stock
- $\Delta$ is equal to $\frac{\partial c_t}{\partial S_t}$
- $\pi$ is the value of the portfolio

$$d \pi = dc_t - \frac{\partial c_t}{\partial S_t} dS_t $$

However, this portfolio is risk free, therefore must earn the risk free rate $r$:

$$ d \pi  = r \pi dt = r(c_t - \frac{\partial c_t}{\partial S_t}S_t)dt $$

$$\therefore dc_t - \frac{\partial c_t}{\partial S_t} dS_t = r(c_t - \frac{\partial c_t}{\partial S_t}S_t)dt$$

Let us now construct an algebraic expression for the infinitessimal $dc_t$. We know that $c_t(S_t, t)$ is a function of $S_t$ and $t$, therefore we can employ a Taylor expansion:

$$dc_t = \frac{\partial c_t}{\partial S_t} dS_t + \frac{\partial c_t}{\partial t} dt + \frac{1}{2} \frac{\partial^2 c_t}{\partial S_t^2}dS_t^2$$

Substituting and cleaning up...

$$dc_t = (\frac{\partial c_t}{\partial t} + rS_t \frac{\partial c_t}{\partial S_t} + \frac{1}{2} \sigma^2 S_t^2 \frac{\partial^2 c_t}{\partial S_t^2})dt + \sigma S_t \frac{\partial c_t}{\partial S_t} dB_t$$

Which we can then substitute into our equation for $d\pi$:

$$(\frac{\partial c_t}{\partial t} + rS_t \frac{\partial c_t}{\partial S_t} + \frac{1}{2} \sigma^2 S_t^2 \frac{\partial^2 c_t}{\partial S_t^2})dt + \sigma S_t \frac{\partial c_t}{\partial S_t} dB_t - \frac{\partial c_t}{\partial S_t} (S_t r dt + S_t \sigma dB_t) = r(c_t - \frac{\partial c_t}{\partial S_t}S_t)dt$$

Cancelling and rearranging we arrive at the Black Scholes Equation:

$$\frac{\partial c_t}{\partial S_t} + \frac{1}{2} \sigma^2 S_t^2 \frac{\partial^2 c_t}{\partial S_t^2} + rS_t \frac{\partial c_t}{\partial S_t} - r c_t = 0$$

With initial conditions:
- $ c(S_T, T) = \max(S_T - K, 0) $

# Two Step Transformation

We are now ready to transform the Black Scholes PDE into a more workable heath equation. Firstly, it would be nice if we could deal with those pesky $r$ terms. In order to this, we will employ a forward method transformation. Our goal in doing this is to bring the reference point forward to $T$:

$$
\begin{cases}
c^*(S^*, T) = e^{r\tau}\max(S - K, 0) \\
S^* = Se^{r\tau}
\end{cases}
$$

Where:
- $\tau = T - t$

We now need to construct the respective partial derivatives in terms of the transformed variables:

$$\frac{\partial c}{\partial t} = \frac{\partial c}{\partial \tau} \frac{\partial \tau}{\partial t} = -\frac{\partial c}{\partial \tau}$$

Employing the product rule and the multi-variate chain rule:

$$= -(-re^{-r\tau} c^*(S^*, t) + e^{-r\tau}(\frac{\partial c^*}{\partial S^*}\frac{\partial S^*}{\partial \tau} + \frac{\partial c^*}{\partial t} \frac{\partial t}{\partial \tau}))$$

Which, recalling the definition of the $c$ transform we previously did:

$$ = rc - (\frac{\partial c^*}{\partial S^*}rS^* - \frac{\partial c^*}{\partial t})e^{-r\tau}$$

Let us now shift our attention to the next term:

$$\frac{\partial c}{\partial S} = \frac{\partial c}{\partial S^*} \frac{\partial S^*}{\partial S}$$

$$= \frac{\partial c^*}{\partial s^*} \frac{\partial c}{\partial c^*} \frac{\partial S^*}{\partial S} = \frac{\partial c}{\partial S^*} e^{-r\tau} e^{r\tau}$$

$$= \frac{\partial c}{\partial s^*}$$

And finally, the second partial derivative term...

$$\frac{\partial^2 c}{\partial S^2} = \frac{\partial}{\partial S}(\frac{\partial c}{\partial s^*})$$

$$= \frac{\partial S^*}{\partial S} \frac{\partial}{\partial S^*}(\frac{\partial c}{\partial S^*}) = e^{r\tau} \frac{\partial^2 c^*}{\partial S^{*2}}$$

Substituting and doing some light cosmetic adjustments we arrive at:

$$\frac{\partial c^*}{\partial t} + \frac{1}{2}\sigma^2 (S^*)^2 \frac{\partial^2 c^*}{\partial S^{*2}} = 0 $$

With initial conditions:

$$ c^*(S^*, T) = \max(S^* - K, 0) $$

We are now ready to apply the second transformation (hence the name 'two-step transformation')! This next transformation is called a price-moneyness transformation:

$$
\begin{cases}
S^* = Ke^{\frac{1}{2}\sigma^2 \tau + x} \\
c^*(S^*, t) = Kf(x, \tau)
\end{cases}
$$

Let us start with $\frac{\partial c^*}{\partial t}$:

$$ \frac{\partial c^*}{\partial t} = \frac{\partial c^*}{\partial \tau}\frac{\partial \tau}{\partial t} $$

$$= - \frac{\partial c^*}{\partial \tau} = -K (\frac{\partial f}{\partial \tau} + \frac{\partial f}{\partial x} \frac{\partial x}{\partial \tau})$$

But we know that $\frac{\partial x}{\partial \tau} = - \frac{1}{2} \sigma^2$, therefore:

$$ \frac{\partial c^*}{\partial t} = -K (\frac{\partial f}{\partial \tau} - \frac{1}{2} \sigma^2 \frac{\partial f}{\partial x}) $$

Shifting our attention to $\frac{\partial c^{\*}}{\partial S^{\*}}$:

$$\frac{\partial c^*}{\partial S^*} = K(\frac{\partial f}{\partial x}\frac{\partial x}{\partial S^*})$$

$$= \frac{K}{S^*} \frac{\partial f}{\partial x} $$

With another application of the partial derivative we get...

$$\frac{\partial^2 c^*}{\partial S^{*2}} = \frac{\partial}{\partial S^*} (\frac{K}{S^*} \frac{\partial f}{\partial x})$$

Applying the product rule:

$$= - \frac{K}{S^{*2}} \frac{\partial f}{\partial x} + \frac{K}{S^*}\frac{\partial}{\partial S^*} \frac{\partial f}{\partial x}$$

If we notice that $\frac{\partial}{\partial S^{\*}} = \frac{\partial x}{\partial S^{\*}} \frac{\partial}{\partial x}$, we can simplify the expression to:

$$\frac{\partial^2 c^*}{\partial S^{*2}} =-\frac{K}{S^{*2}}(\frac{\partial f}{\partial x} - \frac{\partial^2 f}{\partial x^2})$$

We are now ready to substitute in! If we know that:

$$\frac{\partial c^*}{\partial t} + \frac{1}{2}\sigma^2 (S^*)^2 \frac{\partial^2 c^*}{\partial S^{*2}} = 0 $$

Then by substitution, the following is equivalent:

$$-K (\frac{\partial f}{\partial \tau} - \frac{1}{2} \sigma^2 \frac{\partial f}{\partial x})-\frac{1}{2}\sigma^2 (S^*)^2 \frac{K}{S^{*2}}(\frac{\partial f}{\partial x} - \frac{\partial^2 f}{\partial x^2})$$

$$ \therefore \frac{\partial f}{\partial \tau} = \frac{1}{2} \sigma^2 \frac{\partial^2 f}{\partial x^2}$$

The initial conditions must also be adjusted:

$$ c^*(S^*, T) = \max(S^* - K, 0) = kf(x, \tau)$$

$$\therefore f(x, \tau) = \frac{1}{K} \max(S^* - K, 0) = \frac{1}{K} \max(Ke^{\frac{1}{2}\sigma^2 \tau + x} - K, 0)$$

$$\therefore f(x, 0) = \max(e^x- 1, 0)$$

With this, we are ready to solve the heat equation.

# Solving the Heat Equation with Green's Functions

In a previous post, I showed that we can express solutions to the heat equation as the following:

$$u(x,t) = \int_{-\infty}^\infty f(x_0)G(x,t;x_0)dx_0$$

Where $f(x_0)$ is the initial condition function. It was also shown for our initial value problem we will make use of the following Green's Function:

$$G(x,t;x_0) = \frac{1}{\sqrt{2\pi\sigma^2t}}e^{-\frac{(x-x_0)^2}{2\sigma^2t}}$$

Substituting our initial conditions we have:

$$u(x,t) = \int_{-\infty}^\infty \max(e^{x_0} -1,0) \frac{1}{\sqrt{2\pi\sigma^2t}}e^{-\frac{(x-x_0)^2}{2\sigma^2t}}dx_0$$

We can deal with the nasty bounds by adjusting our bounds:

$$= \int_{0}^\infty (e^{x_0} -1) \frac{1}{\sqrt{2\pi\sigma^2t}}e^{-\frac{(x-x_0)^2}{2\sigma^2t}}dx_0$$

Letting $u = \frac{x - x_0}{\sqrt{\sigma^2 t}}$, and adjusting our bounds we now have:

$$\int_{-\infty}^{\frac{x}{\sqrt{\sigma^2t}}}(e^{x - \sqrt{\sigma^2 t}u} - 1) \frac{1}{\sqrt{2 \pi}} e^{-\frac{u^2}{2}}du$$

We now need to complete the square for the first term of the integral, while noticing that the second term is simply the standard cumulative normal distribution.

$$e^x \int_{-\infty}^{\frac{x}{\sqrt{\sigma^2t}}}\frac{1}{\sqrt{2 \pi}} e^{x - \sqrt{\sigma^2 t}u} e^{-\frac{u^2}{2}}du - N(\frac{x}{\sqrt{\sigma^2 t}})$$

Focusing in on the first term, after completing the square we have:

$$e^{x + \frac{\sigma^2 t}{2}} \int_{-\infty}^{\frac{x}{\sqrt{\sigma^2t}}}\frac{1}{\sqrt{2 \pi}} e^{- \frac{(u + \sqrt{\sigma^2 t})^2}{2}}du$$

Let us not perform another substitution. Let $\gamma = u + \sqrt{\sigma^2 t}$...

$$e^{x + \frac{\sigma^2 t}{2}} \int_{-\infty}^{\frac{x}{\sqrt{\sigma^2t}} + \sqrt{\sigma^2 t}}\frac{1}{\sqrt{2 \pi}} e^{- \frac{\gamma^2}{2}}du$$

$$= e^{x + \frac{\sigma^2 t}{2}} N(\frac{x}{\sqrt{\sigma^2t}} + \sqrt{\sigma^2 t})$$

$$\therefore f(x,\tau) = e^{x + \frac{\sigma^2 t}{2}} N(\frac{x}{\sqrt{\sigma^2t}} + \sqrt{\sigma^2 t}) - N(\frac{x}{\sqrt{\sigma^2 t}})$$

We know that $S = Ke^{\frac{1}{2}\sigma^2 \tau + x}$...

$$\therefore x = \ln \frac{S}{K} + (r - \frac{\sigma^2}{2})\tau$$

Let us define $d_2 = \frac{x}{\sqrt{\sigma^2t}}$ and $d_1 = d_2 + \sigma \sqrt{\tau}$

$$\therefore f(x, \tau) = \frac{S}{K} e^{r\tau} N(d_1) - N(d_2)$$

However, we know that $c_t = Ke^{-r\tau} f(x, \tau)$...

$$\therefore c_t = S_t N(d_1) - Ke^{-r\tau} N(d_2)$$

Which is the solution to the Black Scholes Equation.
