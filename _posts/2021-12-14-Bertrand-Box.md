---
title: Bertrand's Box Paradox
author: Domenico Sauta
date: 2021-12-14 11:36:00 +0800
categories: [Probability]
tags: [probability, python]
math: true
mermaid: true
---

Have you ever solved a somewhat trivial problem, but felt like the mathematics deceived you? In today's blog post I plan on walking through a rather elementary probability question whose solution employs an application of Bayes Law. I will also include a simple simulation as an additional unrigorous verification of my calculated solution to this problem.

---

## Bertrand's Box Paradox

The Bertrand Box Paradox was first presented by Joseph Bertrand in 1889. The version of the problem we will be considering today is defined as follows:

"There are three boxes on a table: The first box contains 2 quarters, the second box contains 2 nickels, and the third box contains 1 quarter and 1 nickel. One chooses a box at random, then chooses a coin from that box at random. Provided the chosen coin is a quarter, what's the probability of the other coin being a quarter?"

To solve this problem, we are looking for the probability of two quarters, conditioning on the fact that the first seen coin is a quarter. Expressed mathematically:

$$P(\text{QQ | see Q})$$

Where:
- $P(QQ) = P(NN) = P(QN) = \frac{1}{3}$

## The Naive Approach

At first, it may be tempting to change the sample space to work out our probability. We know that we have observed a $Q$ on the first choice, therefore for our second choice, it may seem tempting to consider an updated sample space of {$QQ, QN$}. If this is the case, provided we observed a $Q$ on the first throw, we now have a sample space of {$Q, N$}, which are equally likely. This would suggest that the probability is in fact $\frac{1}{2}$, which is the incorrect solution. The assumption that they are equally likely is where one may make the critical mistake. Recall that initially, we have the selection of each box as equally likely, but the probability of $QQ$ giving a $Q$ is $1$, and the probability of $QN$ giving a $Q$ is $0.5$, so we must consider the conditional probability to approach this correctly.

## Bayes Theorem and the Law of Total Probability

This problem requires an application of Bayes Theorem, as defined below:

$$P(A|B) = \frac{P(B|A)P(A)}{P(B)}$$

In our case we have the following:

$$P(\text{QQ | see Q}) = \frac{P(\text{QQ}) P(\text{see Q | QQ})}{P(\text{see Q})}$$

It is then convenient to expand the denominator out using the Law of Total Probability. The Law of Total Probability is a very useful theorem that states that for a set of pairwise disjoint events {$B_n : n = 1,2,3,...$} whose union makes up the whole sample space, any event $A$ of the same probability space can be expressed as the following[$^1$][1]:

$$P(A) = \sum_n P(A|B_n)P(B_n)$$

We can make use of this law to express $P(\text{see Q})$ as the following within our original expression:

$$\frac{P(\text{QQ}) P(\text{see Q | QQ})}{P(\text{see Q | QQ})P(\text{QQ}) + P(\text{see Q | NN})P(\text{NN}) + P(\text{see Q | QN})P(\text{QN})}$$

Substituting we have the following:

$$ = \frac{\frac{1}{3} \times 1}{1 \times \frac{1}{3} + 0 \times \frac{1}{3} + \frac{1}{2} \times \frac{1}{3}} $$

$$ = \frac{2}{3} $$

Which is indeed, not $\frac{1}{2}$.

## Python Simulation

Sometimes, I find it useful to verify my solution using a different method. Let us explore a simple way of simulating this problem in Python.

Let us define our sample space using a 2d array.

```python
sample_space = [
    ['Q', 'Q'],
    ['Q', 'N'],
    ['N', 'N']
]
```

Let us construct our experiment

```python
def experiment() -> str:
    # random.choice samples from a discrete uniform distribution
    box = random.choice(3)
    choice_in_box = random.choice(2)
    if sample_space[box][choice_in_box] == 'Q':
        # note that 1 - choice_in_box is just selecting the remaining coin
        return sample_space[box][1 - choice_in_box]

def run_many_experiments(num_experiments):
    results = {'Q': 0, 'N':0}
    for _ in range(num_experiments):
        output = experiment()
        if output:
            results[output] += 1
    return results
```

Finally, let us visualise the results of many trials

```python
X = np.logspace(1,5, dtype = int, num = 50)
Y = np.zeros(50)

for i in range(len(X)):
    result = run_many_experiments(X[i])
    prob = result['Q'] / sum(result.values())
    Y[i] = prob

plt.plot(X,Y,'.-', color = 'orange')
plt.xscale('log')
plt.xlabel('Number of trials')
plt.ylabel('Probability')
plt.title('P(QQ| see Q on first)')  
```

Which produces the following:

![plot](/assets/2021-12-14/plot.png)

Which does indeed also converge to $\frac{2}{3}$ as expected.


[1]: <https://en.wikipedia.org/wiki/Law_of_total_probability> "Wiki Law of Total Probability"
