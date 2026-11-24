# User guide: Toll Plaza Queue Simulation web demo

**Open the demo:** [https://YOUR-GITHUB-USER.github.io/toll-plaza-queue-sim/](https://YOUR-GITHUB-USER.github.io/toll-plaza-queue-sim/). It runs entirely in your browser (current Chrome, Edge or Firefox; Safari 16 or later also works). Nothing is installed or uploaded.

You can also open it offline: download this repository and double-click `docs/index.html`.

![Labelled screenshot of the demo](figures/demo-screenshot-labelled.png)

The numbers in the screenshot mark the parts of the page: **1** sliders, **2** buttons, **3** animation, **4** live statistics, **5** chart, **6** check against theory.

## Controls

| Control | What it does | Range | Default |
|---|---|---|---|
| Arrival rate λ | Average number of cars arriving per minute. Takes effect immediately. | 0.5 – 12 cars/min | 6.0 |
| Service rate µ per booth | Average number of cars one booth can serve per minute. Takes effect immediately. | 0.5 – 6 cars/min | 2.5 |
| Open booths c | Number of booths open. Changing it restarts the simulation. | 1 – 8 | 3 |
| Animation speed | How fast simulated time runs, in simulated minutes per real second. Changes only the speed of the animation, not the results. | 0.1 – 10 | 0.5 |
| Start / Pause | Runs or pauses the simulation | — | — |
| Reset | Clears the run and applies the current settings | — | — |
| Load peak-hour example | Sets λ = 6, µ = 2.5, c = 3 and resets | — | — |
| Run check | Runs 30 simulations of 60,000 cars each with the current settings and compares the average wait with the exact theoretical value | — | — |

## Reading the outputs

- **Animation (3):** cars drive in from the left, wait in line, are served at a booth and leave on the right. The line is a single first-come-first-served queue, drawn in rows for clarity. A car's colour shows how long it waited: green for no wait, through yellow, to red for two minutes or more. A booth light is green when the booth is idle and red when it is serving. If the line is too long to draw, a red note counts the extra cars.
- **Live statistics (4):**
  - *Simulated time* is in minutes; *cars waiting now* is the current queue length; *cars served* counts cars that have started service.
  - *Average wait so far* is the mean wait of all cars served, in minutes. It is a single random run, so it is noisy, and it starts too low because the plaza begins empty. It takes a few hundred cars to settle.
  - *Longest wait so far* is the largest single wait, in minutes.
  - *Traffic intensity ρ = λ/(cµ)* is the share of booth capacity in use. If ρ is 1 or more, a red warning appears: cars arrive at least as fast as the booths can serve them, so the line grows without bound.
  - *Exact long-run average wait* is the Erlang C value for the current settings, in minutes. It has no uncertainty because it is a formula, and it is shown only when ρ < 1.
  - *Fewest booths for wait < 2 min* is the smallest number of booths for which the exact average wait is under two minutes.
- **Chart (5):** the blue line is the running average wait. The grey dashed line is the exact long-run value and the red dotted line is the two-minute target. A noisy blue line that drifts towards the grey line is expected.
- **Check (6):** the output of **Run check** is a table with the simulated mean wait and its 95% confidence interval (for example 0.44 ± 0.01 min), the exact value, the relative error, and PASS if the error is under 5%. The ± shows the uncertainty from randomness: the interval is expected to contain the true long-run average in about 95% of checks.

## Try this

1. Press **Load peak-hour example**, set the animation speed to about 5 and press **Start**. After a few hundred cars the blue line drifts towards the grey line at about 0.43 minutes, well below the red two-minute target. The exact reading and the "fewest booths" box say 3.
2. Set **Open booths** to 2 and press **Start**. The warning appears (ρ = 1.2), the queue grows steadily and the average wait keeps climbing instead of settling. With 1 or 2 booths no steady state exists.
3. Return to the example and raise **Arrival rate λ** to 7.3 while it runs. The exact wait jumps to about 4.75 minutes, and the "fewest booths" box changes to 4. Then set booths to 4 and the wait stays low.
4. Lower **Service rate µ** to 2.2 with 3 booths (ρ = 0.91). The exact wait rises to about 1.39 minutes. A small change in µ has a large effect on the wait, which is the sensitivity result from the notebook.

## Check against the notebook

With the default settings (λ = 6, µ = 2.5, c = 3), press **Run check**. The demo should give a mean wait of about **0.43 min**, anywhere in 0.41 to 0.45 (within 5% of the exact 0.4315), and show **PASS**. This matches the notebook result in section 4b (verification) and section 5 (the experiment). The two will differ in the third decimal place because they use different random seeds, but both agree with theory within their confidence intervals.

## Limitations

- The animation is one random run and shows no confidence interval. The statistics are noisy early on and are biased low until the queue has warmed up. Use **Run check** for a proper comparison.
- The demo covers the constant-arrival-rate model only. The rush-hour extension, the error-versus-sample-size plot and the sensitivity study are in the notebook.
- The demo draws the shared line as rows in front of the booths. The cars are actually served in strict first-come-first-served order, regardless of row.
- Changing µ while running restarts the remaining service time of cars currently at booths. This does not affect long-run results, because service times are memoryless.
- **Run check** is reliable for ρ up to about 0.93 (errors stayed under 4% in testing). At ρ around 0.97 or higher the waits are so variable that 30 runs of 60,000 cars can miss by more than 5% and report FAIL even though the simulator is correct. When ρ is 1 or more there is no steady state, so the check is not available.
- Results are only as good as the inputs: λ = 6 and µ = 2.5 are illustrative values, not measurements of a real plaza.
