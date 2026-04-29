..  _startinganalysis:

-----------------
Analysis workflow
-----------------

Below, I've walked through my process for getting started with an analysis. If you haven't done an independent data analysis project before, I recommend reading this through once now and then coming back again after you've worked through the rest of the tutorial when you will have more context for some of the issues and ideas here.

Start With an Outline
---------------------

Before you write a single line of code, you should have a plan. This might sound obvious, but I think it is one of the most common places where new (and experienced) researchers run into trouble. While I understand that it is tempting to dive straight into coding, taking the time to think through what you need to do first (and writing it down!) will pay dividends down the line as you avoid messy scripts and confusing results.

This module walks through a practical framework for planning, validating, and documenting your analyses.

The most important thing you can do before coding is write out your analysis plan in plain language. This is your guiding star. It doesn't need to be long or formal — even a bulleted list in a notebook or a comment block at the top of a script is fine. The key here is to force yourself to think through the full pipeline before you start building it.

Your outline should address all of the following:

What is your question?
^^^^^^^^^^^^^^^^^^^^^^

Be as specific as possible. Our starting question is "Does neural activity change with reach direction?" However, some key analysis components are missing from this framing. First, "change" is vague. How are we going to measure that? Second, what neural activity? Are we using our entire population we recorded or single neurons? Third, this glosses over what data is needed to address this question. Are we interest

"Do individual neurons in PMd show cosine-like tuning to reach direction during the peri-movement period?" is better. A well-defined question constrains the rest of your analysis decisions.

### What data do you need?

Think about which parts of the dataset are relevant. Do you need all trials or only successful ones? Which neurons? What time windows? Do you need behavioral data alongside the neural data? Being explicit here prevents you from loading and processing more data than necessary, and helps you catch cases where you're missing something critical.

### What structure does the data need to be in?

Most analyses require your data to be in a specific format: a trials-by-neurons matrix, a time series aligned to an event, a set of labeled training and testing splits. Think about what your analysis function actually expects as input, and plan the transformation from raw data to that format.

### How are you changing the data?

Every step between raw data and your final result is a transformation: filtering trials, binning spikes, z-scoring firing rates, averaging across conditions. Write down each transformation. This serves two purposes: it makes your pipeline explicit, and it gives you natural checkpoints for validation (more on this below).

### What does the output look like?

What does your analysis actually produce? A single number (e.g., classification accuracy)? A matrix of coefficients? A set of decoded trajectories? Knowing the expected shape and type of your output helps you catch bugs early.

### How do you want to plot the results?

Thinking about your figures early is surprisingly useful. It forces you to consider what comparisons you're making, what the axes should be, and what a reader (or your PI) needs to see to understand the result. Sketching out your target figure — even by hand — is a great habit.

### What should the results look like if your hypothesis is right or wrong?

This is the most important question, and the one people skip most often. Before you look at the output, you should have a prediction. If neurons are tuned to reach direction, what should your tuning curve look like? If your decoder is working correctly, how low should the angular error be? If you can't articulate what the "right" answer looks like, you probably don't understand the analysis well enough yet.

Having a prediction for the "wrong" answer is equally valuable. If your classifier is performing at chance, that tells you something. If it's performing way above what's reasonable, that might mean you have a data leakage problem rather than a genuine result.

## Validate Your Code

Writing code that runs without errors is necessary but nowhere near sufficient. The hardest bugs in data analysis aren't crashes — they're silent errors that produce plausible-looking but wrong results. Validation needs to be an active, ongoing part of your workflow.

### Test on Known Quantities

Whenever possible, test your analysis on data where you already know the answer. This could be:

- **Synthetic data** you generate yourself. For example, if you're fitting a cosine tuning model, generate fake spike counts from a known cosine function and verify that your fitting code recovers the correct parameters.
- **Published results.** If you're implementing a known algorithm, find a paper or textbook that reports results on a specific dataset and check that you can reproduce them.
- **Sanity checks on real data.** Does the neuron with the highest firing rate match what you see in the raw raster? Does the trial count per condition add up to what you expect?

### Integrate Validation Into Your Outline

Go back to your analysis outline and, at each step, add a validation check. For example:

> **Step:** Bin spikes into 50ms windows around movement onset.
> **Check:** For a single trial and neuron, manually count spikes in one bin from the raw timestamps. Does it match the binned value?

This doesn't need to be a formal unit test (though those are great too). Even a quick visual check or print statement at each stage can catch problems early.

### Just Because It Ran Does Not Mean It Did What You Think

This is worth repeating. MATLAB and Python will happily execute code that is mathematically or logically wrong without any error message. Common silent failures include:

- Broadcasting or implicit expansion producing the wrong matrix dimensions
- Averaging over the wrong axis
- Off-by-one errors in time alignment
- Using the wrong variable (especially in a notebook where old variables linger in memory)

Get in the habit of checking intermediate results, not just the final output.

### Investigate Outliers

If a few data points look strange — a trial with an absurdly high firing rate, a decoded trajectory that goes to infinity, a condition with very few trials — don't just ignore them. Track down the source. Sometimes outliers reveal a bug in your code. Sometimes they reveal something interesting about the data. Either way, you need to understand them before deciding what to do with them.

### Are Things on the Scale You Expected?

This is a quick but powerful sanity check. If you're computing firing rates, are they in a reasonable range (e.g., 5–100 Hz for cortical neurons, depending on the area)? If you're computing classification accuracy, is it between chance and 100%? If you're decoding velocity, are the units physically plausible? A result that's off by orders of magnitude almost always means something went wrong upstream.

## Code Style

Clean code isn't just about aesthetics — it directly affects your ability to debug, collaborate, and return to an analysis months later.

### Follow a Style Guide

Consistent formatting makes code dramatically easier to read. For Python, [PEP 8](https://peps.python.org/pep-0008/) is the standard. For MATLAB, MathWorks publishes [coding conventions](https://www.mathworks.com/help/matlab/matlab_prog/matlab-code-conventions.html) that are worth following. You don't need to memorize every rule — just pick a style and be consistent.

Key habits: use descriptive variable names (`firing_rates` instead of `fr`), keep functions short and focused, and avoid deeply nested logic.

### Write Modular Functions

Resist the temptation to write one giant script that does everything. Instead, break your analysis into small, reusable functions, each with a clear input, output, and purpose. For example:

- A function that loads and filters trials
- A function that bins spikes into a given time window
- A function that fits a tuning curve to a single neuron
- A function that plots a PSTH

This makes your code easier to test (you can validate each function independently), easier to reuse, and easier for someone else to understand.

## Reproducibility

Your analysis needs to be reproducible — by you in six months, by your labmate next week, and by a reviewer who wants to verify your results.

### Comment Your Code

Comments should explain *why*, not just *what*. The code itself tells you what's happening; the comments should tell you the reasoning behind each decision. A good starting point is to paste in the analysis outline you wrote earlier and fill in the code beneath each step. This gives your script a natural narrative structure.

```python
# Bin spikes into 50ms windows from 100ms before to 500ms after movement onset.
# We use this window because it captures the peri-movement planning activity
# described in Churchland et al., 2006.
bin_edges = np.arange(move_onset - 100, move_onset + 500, 50)
```

### Pay Attention to Random State

Many analyses involve randomness: train/test splits, random initializations, bootstrap resampling. If you don't control the random seed, you'll get slightly different results every time you run the code, which makes debugging painful and exact replication impossible.

Always set the random seed at the top of your script or function:

````{tab} Python
```python
import numpy as np
np.random.seed(42)

# If using scikit-learn, also set random_state in function calls
from sklearn.model_selection import train_test_split
train, test = train_test_split(data, test_size=0.2, random_state=42)
```
````

````{tab} MATLAB
```matlab
rng(42);

% Now any random operations (randperm, randn, etc.) are reproducible
idx = randperm(nTrials, nTest);
```
````

Pick any seed value you like — the specific number doesn't matter, only that it's consistent. Document it in your code so others can reproduce your exact results.

---

## Practice Problems

These exercises are designed to reinforce the habits described in this module. They're intentionally simple — the goal is to practice the *process*, not the analysis.

### Problem 2.1: Write an Analysis Outline

Pick one of the following questions and write out a complete analysis outline following the framework above (question, data needed, data structure, transformations, output, plot, expected results):

- *Do neurons fire more during movement than during a baseline hold period?*
- *Can you predict reach direction from the population firing rate on a single trial?*

You don't need to write any code — just the plan.

### Problem 2.2: Debug a Broken Analysis

<!-- TODO: Include a short script with 2-3 intentional bugs (e.g., wrong axis
for averaging, off-by-one error, missing trial filter). Task: find and fix
the bugs using the validation strategies from this module. -->

*A starter script for this problem will be provided alongside the tutorial dataset.*

### Problem 2.3: Refactor a Messy Script

<!-- TODO: Provide a single long script that works correctly but has poor
style: no comments, magic numbers, non-descriptive variable names,
everything in one block. Task: refactor into modular functions with
comments and meaningful names. -->

*A starter script for this problem will be provided alongside the tutorial dataset.*

---

**Next up:** [Module 3: The Data Structure →](module_03_data_structure.md)

