# Rethinking the relationship between age and ability

I used to have a fairly defeatist attitude about age and achievement. Groundbreaking achievements seem to come disproportionately from younger people; what I call the *achievement gap*. Nobel Laureates typically do their prizewinning work in their 20s and 30s. Successful entrepreneurs often start their companies before age 30. Artists make their greatest breakthroughs while they're young (often with some help from "cognitive enhancers").

Based on this, it's easy to conclude that as we get older, our abilities decline, making truly great achievements increasingly unlikely. In other words, the achievement gap is best explained by an *ability gap*. But I've reconsidered based on a simple statistical argument.

The key insight is that even if abilities don't decline with age, we should still expect to see an achievement gap between younger and older people due to selection effects. Let me explain why.

First, it's uncontroversial that achievement depends on both ability and luck. Being in the right place at the right time, having the right collaborators, getting the right break - luck plays a huge role. Second, within any field, the young people who produce the greatest achievements are selected to keep pursuing that field. An academic who does groundbreaking work as a PhD student is more likely to get a postdoc and professorship to continue researching. A successful young entrepreneur has an easier time raising funds for subsequent ventures. An artist who creates a major work early on is more likely to keep getting commissions and exhibitions.

Consequently, people who continue pursuing competitive fields as they age probably had both exceptional ability and exceptional luck in their earlier years. Even if their ability remains strong as they age, their luck will likely run out, resulting in less impressive achievements.

To be clear, this statistical argument doesn't prove that abilities don't decline. But it offers a powerful alternative explanation for the existence of an achievement gap based on simple and realistic assumptions.

It also has an interesting practical implication. If, as I suspect, we are overestimating the ability gap, then middle-aged and older adults who begin pursuing competitive fields have nearly as much potential for groundbreaking achievements - Nobel prize-winning scientific breakthroughs, unicorn companies, and magnificent works of art - as their younger counterparts. The reason this rarely happens, I suspect, has less to do with biological limitations and more to do with risk-aversion and lifestyle creep. Let's face it, PhD students, first-time entrepreneurs, and artists have it tough. How many 30-, or 40-, or 50-somethings would walk away from a decently-paid 9-5 to toil for years sharing a 3-bed flat with two guys they met on Craiglist and eating instant ramen every night for a slim shot at greatness? But those few are are willing to try, I suspect, have nearly as good a shot as anyone.

The rest of this post is dedicated to mathematizing this argument, as well as making two additional points. 1) The achievement gap will be larger in more competitive fields, 2) The achievement gap is usually larger when achievement depends more heavily on luck.

## Formal statement

Consider a two-period model in which a person achieves something with quality $x_i$ for $i \in \{0, 1\}$. The achievements $x_i$ are independently distributed random variable with cumulative distribution functions $F_i$ and expected values $E[x_i] = \mu_i$. The achievement gap is $x_1 - x_0$ and the ability gap is $\mu_1 - \mu_0$. Suppose the observed achievement gap is the average ability gap $\Delta = E[x_1 - x_0] = \mu_1 - \mu_0$. If the observed achievement gap is negative (the period-1 achievement is lower quality on average), then it implies the ability gap is also negative.

But now suppose that only the top $1 - \tau$ fraction of people are selected to achieve something in period 1 based on the quality of their period-0 achievement. Formally, suppose the observed achievement gap is instead the expected achievement gap of the selected individuals $\Delta = E[x_1 - x_0 | x_0 > F_0^{-1}(\tau)] = \mu_1 - \mu_0 - l$ where $l = E[x_0 | x_0 > F_0^{-1}(\tau)] - \mu_0$ is period-0 "luck". Then, a negative observed achievement gap no longer implies a negative ability gap. It simply implies that ability cannot increase by more than $l$.

Two additional conclusions follow from this setup.

First, more competitive fields (i.e., with a higher $\tau$) will have a larger observed achievement gap because $l$ increases with $\tau$.

Second, less obviously, fields in which period-0 achievements depend more heavily on luck will also have a larger observed achievement gap holding ability constant, provided the absolute quality of the period-0 achievement must be higher to advance to period 1. Formally, consider two fields in which period-0 achievement (which we'll now simply denote $x$ going forward to avoid excess subscripts) follows distributions $\rho$ and $\nu$ with cumulative distributions functions $F_\rho$ and $F_\nu$ respectively. $\nu$ is a mean-preserving spread of $\rho$ (i.e., $\nu$ involves more luck). If $F_\rho^{-1}(\tau) < F_\nu^{-1}(\tau)$, then the magnitude of the observed achievement gap is larger under $\nu$.

The condition $F_\rho^{-1}(\tau) < F_\nu^{-1}(\tau)$ will be satisfied for sufficiently large values of $\tau$ under many reasonable distributions, such as when $\rho$ and $\nu$ are normal and $\tau > 0.5$. As an aside, it is easy to show that, in the normal case, the observed achievement gap will be larger under $\nu$ for all values of $\tau \in (0, 1)$ by taking the derivative of the conditional expectation of $x$ with respect to the standard deviation.

Proof.

Because $\nu$ is a mean-preserving spread of $\rho$, $E_{x \sim \rho}[u(x)] \geq E_{x \sim \nu}[u(x)]$ for all (weakly) concave, non-decreasing real-valued functions $u$.

Consider $u(x) = (x - F_\nu^{-1}(\tau)) * 1\{x < F_\nu^{-1}(\tau)\}$ where $1$ is the indicator function. $u$ is concave and non-decreasing.

Then for $\theta \in \{\rho, \nu\}$,

$$
    E_{x \sim \theta}[u(x)]
    = \int_{-\infty}^{\infty} (x - F_\nu^{-1}(\tau)) * 1\{x < F_\nu^{-1}(\tau)\} dF_\theta(x) \\
    =
        \int_{-\infty}^{F_\theta^{-1}(\tau)} x dF_\theta(x)
        + \int_{F_\theta^{-1}{\tau}}^{F_\nu^{-1}(\tau)} x dF_\theta(x)
        - \int_{-\infty}^{F_\nu^{-1}(\tau)} F_\nu^{-1}(\tau) dF_\theta(x)
        \\
    =
        \tau E_{x \sim \theta}[x | x < F_\theta^{-1}(\tau)] \\
        + (F_\theta(F_\nu^{-1}(\tau)) - \tau)E_{x \sim \theta}[x | F_\theta^{-1}(\tau) < x < F_\nu^{-1}(\tau)] \\
        - F_\nu^{-1}(\tau) F_\theta(F_\nu^{-1}(\tau)).
$$

Note that when $\theta = \nu$ the above expression simplifies to

$$
    \tau E_{x \sim \nu}[x | x < F_\nu^{-1}(\tau)]
    - F_\nu^{-1}(\tau) \tau.
$$

Also note that

$$
    \tau E_{x \sim \theta}[x | x < F_\theta^{-1}(\tau)]
    = E_{x \sim \theta}[x] - (1 - \tau)E_{x \sim \theta}[x | x > F_\theta^{-1}(\tau)].
$$

Using $E_{x \sim \rho}[u(x)] \geq E_{x \sim \nu}[u(x)]$ and noting that $E_{x \sim \rho}[x] = E_{x \sim \nu}[x]$ by the definition of a mean-preserving spread, we obtain the following after some gnarly algebra:

$$
    E_{x \sim \nu}[x | x > F_\nu^{-1}(\tau)] - E_{x \sim \rho}[x | x > F_\rho^{-1}(\tau)]
    = l_\nu - l_\rho \\
    \geq \frac{F_\rho(F_\nu^{-1}(\tau)) - \tau}{1 - \tau}
    (F_\nu^{-1}(\tau) - E_{x \sim \rho}[x | F_\rho^{-1}(\tau) < x < F_\nu^{-1}(\tau)])
    \geq 0
$$

where $l_\nu$ and $l_\rho$ are the "luck" required to advance to period 1 under $\nu$ and $\rho$, respectively.

Note that $F_\rho(F_\nu^{-1}(\tau)) - \tau \geq 0$ because $F_\nu^{-1}(\tau) > F_\rho^{-1}(\tau)$ and the cumulative distribution function is non-decreasing.

This result is intuitive. The more luck-dependent the quality of the achievement, the more luck is required to advance to period 1. Therefore, given that the abilities in both fields are the same, the more luck-dependent one will exhibit a larger observed achievement gap.