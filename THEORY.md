# Theory and mathematics

## 1. Problem formulation

Cars arrive at an expressway toll plaza at random. The plaza has $c$ booths open, and each booth serves one car at a time from a single first-come-first-served line. At peak time the operator must decide how many booths to open.

**Question:** how many booths must be open at peak time to keep the average waiting time in the queue under two minutes?

**Quantities of interest:**

- the traffic intensity $\rho$, which says whether the queue can reach a steady state at all;
- the mean waiting time in the queue, $W_q$, in minutes (time from arrival until service starts, not including service), with a 95% confidence interval;
- the smallest $c$ for which $W_q$ is under the two-minute target;
- how sensitive that answer is to the arrival rate $\lambda$ and the service rate $\mu$.

## 2. Assumptions

1. **Arrivals form a Poisson process with a constant rate $\lambda$ during the peak window.** Peak traffic is the sum of many independent drivers, which is the situation in which Poisson arrivals are a good model. Real traffic can arrive in platoons, so $\lambda$ should be measured over a short window. The rush-hour extension relaxes constancy by allowing $\lambda(t)$ to change.
2. **Service times are independent and exponential with rate $\mu$.** This is the simplest memoryless model and gives exact formulas to verify against. Real service times, for example with electronic tags, are usually less variable than exponential, so waits here are likely to be somewhat pessimistic.
3. **One shared first-come-first-served line, with identical booths.** This is the standard M/M/c structure. Real drivers choose lanes and may switch, which would make waits somewhat longer than in a single shared line.
4. **No limit on queue length, and no car balks or leaves.** Reasonable while queues are short compared with the road space available.
5. **A steady state requires $\rho < 1$.** Long-run averages exist only then, so the core experiment reports averages only for such $c$ and treats $\rho \ge 1$ as unstable.
6. **The system starts empty, and the first 10% of each run is discarded** so that start-up effects do not bias the averages.
7. **The inputs $\lambda = 6$ cars/min and $\mu = 2.5$ cars/min per booth are illustrative.** They stand in for traffic counts and transaction timings that would be measured at a real plaza.

## 3. Model equations

The plaza is an M/M/c queue (Kendall notation: Markovian arrivals, Markovian service, $c$ servers). Its offered load and traffic intensity are

$$
a = \frac{\lambda}{\mu}, \qquad \rho = \frac{a}{c} = \frac{\lambda}{c\mu}.
$$

If $\rho \ge 1$, cars arrive at least as fast as the booths can serve them and the queue grows without bound. If $\rho < 1$ the queue settles to a steady state.

**Exact mean wait, one booth (M/M/1):**

$$
W_q = \frac{\rho}{\mu - \lambda}.
$$

**Exact mean wait, $c$ booths (Erlang C):** the probability that an arriving car has to wait is

$$
P_{\text{wait}} = \frac{\dfrac{a^{c}}{c!}\,\dfrac{1}{1-\rho}}{\displaystyle\sum_{k=0}^{c-1}\frac{a^{k}}{k!} + \frac{a^{c}}{c!}\,\dfrac{1}{1-\rho}}, \qquad W_q = \frac{P_{\text{wait}}}{c\mu - \lambda}.
$$

For $c = 1$ this gives $P_{\text{wait}} = \rho$ and reduces to the M/M/1 result.

**Random variates (inverse transform):** if $U$ is uniform on $(0,1)$, then an exponential variate with rate $r$ is

$$
X = -\frac{\ln U}{r}.
$$

Gaps between arrivals use $r = \lambda$ and service times use $r = \mu$.

**Confidence interval:** with $R$ independent replications giving means $\bar{x}_1, \dots, \bar{x}_R$, sample mean $\bar{x}$ and sample standard deviation $s$,

$$
\bar{x} \pm t_{0.975,\,R-1}\,\frac{s}{\sqrt{R}}.
$$

**Time-varying arrivals (thinning):** to generate a Poisson process with rate $\lambda(t) \le \lambda_{\max}$, generate candidates at rate $\lambda_{\max}$ and keep a candidate at time $t$ with probability $\lambda(t)/\lambda_{\max}$ [4].

**Sensitivity (elasticity):** for a parameter $p$,

$$
E_p = \frac{\partial W_q / W_q}{\partial p / p}.
$$

Because $W_q = \dfrac{1}{\mu}\,f\!\left(\dfrac{\lambda}{\mu}\right)$ with $f(a) = P_{\text{wait}}(a,c)/(c-a)$, writing $\varepsilon = a f'(a)/f(a)$ gives $E_\lambda = \varepsilon$ and $E_\mu = -1 - \varepsilon$. So when raising $\lambda$ lengthens the wait ($\varepsilon > 0$), $\mu$ has the larger effect, by exactly 1.

| Symbol | Meaning | Units | Value / range | Source |
|---|---|---|---|---|
| $\lambda$ | Arrival rate | cars/min | 6 (demo slider 0.5–12) | Illustrative input |
| $\mu$ | Service rate of one booth | cars/min | 2.5 (demo slider 0.5–6) | Illustrative input |
| $c$ | Number of open booths | booths | 1–5 in the notebook, 1–8 in the demo | Decision variable |
| $a$ | Offered load $\lambda/\mu$ | booths | 2.4 at the inputs above | Derived |
| $\rho$ | Traffic intensity $\lambda/(c\mu)$ | dimensionless | 0.8 at $c = 3$; stable only if below 1 | Derived [1] |
| $P_{\text{wait}}$ | Probability an arriving car must wait | dimensionless | 0.647 at $c = 3$ | Erlang C [1], [5] |
| $W_q$ | Mean wait in the queue | min | 0.4315 at $c = 3$ | Erlang C [1], [5] |
| Target | Acceptable mean wait | min | 2 | Project requirement |
| $\lambda(t)$ | Time-varying arrival rate (rush-hour extension) | cars/min | 3, then 6, then 2, in three 60-min phases | Illustrative input |
| $\lambda_{\max}$ | Maximum rate used for thinning | cars/min | 6 | Equal to $\max \lambda(t)$ |
| $R$ | Replications per estimate | — | 50 in the experiment, 30–60 in checks | Chosen (see section 4) |
| $n$ | Cars per replication | cars | 100,000 in the experiment, 60,000 in checks | Chosen (see section 4) |
| $U$ | Uniform random number | — | on (0, 1) | NumPy generator [6] |
| $t_{0.975,R-1}$ | Student t quantile | — | about 2.01 for $R = 50$ | SciPy [7] |
| $E_p$ | Elasticity of $W_q$ with respect to $p$ | dimensionless | $E_\lambda \approx +6.0$, $E_\mu \approx -7.0$ at $c = 3$ | Derived |

## 4. Numerical method

**Next-event time advance.** The simulator keeps an event list ordered by time and jumps directly from one event to the next [2], [3]. There are two kinds of event:

- *Arrival:* schedule the next arrival. If a booth is idle, the car starts service immediately (wait = 0) and a departure is scheduled; otherwise it joins the first-come-first-served queue.
- *Departure:* if cars are waiting, the first one starts service (its wait is the current time minus its arrival time) and a new departure is scheduled; otherwise the booth becomes idle. On a tie in time, departures are processed first.

**Accuracy.** There is no time step and so no discretisation error: event times are exact. The only error is statistical, from using a finite number of random samples. By the central limit theorem it shrinks in proportion to $1/\sqrt{n}$ for $n$ observations, and in proportion to $1/\sqrt{R}$ for the confidence interval over $R$ replications (confirmed in section 5).

**Stability.** The method itself never fails, but results are meaningful only if $\rho < 1$. For $\rho \ge 1$ the simulated wait keeps growing with run length instead of settling, which the notebook shows directly (section 5).

**Choosing the run length and number of replications.**

- *Warm-up:* the first 10% of cars in each run are discarded. For the main experiment this is 10,000 cars, comfortably longer than the time the queue needs to forget its empty start at $\rho = 0.8$.
- *Replications and length:* the main experiment uses $R = 50$ runs of $n = 100{,}000$ cars. At $c = 3$ this gives a 95% interval of about $\pm 0.006$ min, far narrower than the gap to the two-minute target, so the decision is not sensitive to this choice. Verification checks use $R = 30$ to $40$ runs of 60,000 cars (the error-versus-sample-size study uses 60 runs at each $n$).
- *Random streams:* independent replications use separate streams from NumPy's `SeedSequence.spawn`, and every result uses a fixed seed so that it is reproducible.
- *Rush-hour extension:* this is a finite-horizon (180-minute) simulation started empty, with no steady state, so it is averaged over 200 replications and no warm-up is discarded.

## 5. Verification

Every row below is an `assert` in the notebook (or, for the demo, the **Run check** button), so a disagreement stops the run.

| Check | Expected | Obtained | Pass? |
|---|---|---|---|
| Erlang C with $c = 1$ equals the M/M/1 formula (notebook section 2) | equal | difference below $10^{-12}$ | yes |
| M/M/1, $\lambda = 2$, $\mu = 2.5$: mean wait (section 4a) | 1.6000 min | 1.5986 ± 0.0267 min (0.09% error) | yes |
| M/M/3, $\lambda = 6$, $\mu = 2.5$ (section 4b) | 0.4315 min | 0.4314 ± 0.0072 min (0.01%) | yes |
| M/M/4, $\lambda = 6$, $\mu = 2.5$ (section 4b) | 0.0718 min | 0.0716 ± 0.0010 min (0.26%) | yes |
| M/M/5, $\lambda = 6$, $\mu = 2.5$ (section 4b) | 0.0175 min | 0.0176 ± 0.0003 min (0.54%) | yes |
| M/M/5, $\lambda = 4$, $\mu = 1$: a differently shaped system (section 4b) | 0.5541 min | 0.5501 ± 0.0112 min (0.73%) | yes |
| Convergence rate: log–log slope of error against $n$ (section 8) | $-0.5$ | $-0.47$ | yes |
| Limiting case $\rho \ge 1$: wait grows in proportion to run length (section 5) | about 10× when the run is 10× longer | $c = 1$: 650 → 6,441 min; $c = 2$: 92 → 922 min | yes |
| Thinning: expected cars per morning (section 7) | 660 | 660.4 | yes |
| Thinning: expected cars in the rush hour (section 7) | 360 | 360.5 | yes |
| Thinning: variance of the count divided by its mean (a Poisson property) | about 1 | 0.985 | yes |
| Thinning with a constant rate through the queue, $c = 3$ (section 7) | 0.4315 min | 0.4405 min (2.1%) | yes |
| Sensitivity: simulated vs exact elasticity, $\lambda$ and $\mu$ (section 9) | +6.27 and −7.52 | +6.19 and −7.48 | yes |
| Elasticity identity $E_\mu = -1 - E_\lambda$ (exact local values) | −1 | −1.0000 | yes |
| Web demo, **Run check** at $\lambda = 6$, $\mu = 2.5$, $c = 3$ | 0.4315 min | 0.4372 ± 0.0103 min (1.3%) | yes |

The acceptance tolerance for each simulation-versus-theory `assert` is a 5% relative error. The observed errors are all under 2.2%, and wherever a confidence interval is reported the exact value lies inside it.

![Verification plot](figures/verification.png)

## 6. Concepts applied

| Concept | Where it is used in this project |
|---|---|
| Next-event time-advance discrete-event simulation | The event-list simulator, notebook section 3, and the same engine in the web demo |
| Inverse-transform random variate generation | Exponential arrival gaps and service times (section 3 above; notebook section 3) |
| Monte Carlo replications and confidence intervals | 95% interval on every simulated mean (notebook sections 4, 5, 7, 8 and 9) |
| Verification against analytic results | M/M/1 and Erlang C asserts (notebook section 4); web demo **Run check** |
| Steady state and the stability condition $\rho < 1$ | Choosing which settings can be averaged; unstable $c = 1, 2$ shown in notebook section 5 |
| Non-homogeneous Poisson process by thinning | Rush-hour arrivals (notebook section 7) |
| Law of large numbers and $1/\sqrt{n}$ convergence | Log–log error plot (notebook section 8) |
| Sensitivity analysis with common random numbers | Same seed for every perturbed case; elasticities of $W_q$ with respect to $\lambda$ and $\mu$ (notebook section 9) |

## 7. References

See [REFERENCES.md](REFERENCES.md).
