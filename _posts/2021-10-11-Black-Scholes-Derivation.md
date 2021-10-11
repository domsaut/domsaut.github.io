---
title: Deriving the Black Scholes Pricing Formula
author: Domenico Sauta
date: 2021-09-11 13:36:00 +0800
categories: [Options Pricing]
tags: [options, trading]
math: true
mermaid: true
---

The Black-Scholes options pricing formula (Black, Scholes 1973) is one of the most profound results in financial derivative pricing history. In today's post, I am going to demonstrate a way to derive the price of a European call option using risk-neutral conditional expectation in $Q$-measure.

The Black-Scholes formula is a solution to the following partial differential equation:

$$
\frac{\partial c}{\partial t} + \frac{1}{2} \sigma^2 S^2 \frac{\partial^2 c}{\partial S^2} + r S \frac{\partial c}{\partial S} - rc = 0
$$

Which is known as the *Black-Scholes Equation*. This can be derived by constructing a Delta-Hedged option portfolio, but that is a problem for another post.

---
