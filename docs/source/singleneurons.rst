..  _singleneurons:

----------------------
Single Neuron Analyses
----------------------

Ok, so now with our data cleaned we can start to analyze neural activity. To start, we're going to take a look at what individual neurons are doing. Population-level analyses are powerful, but they're built on an understanding of what individual neurons are doing. This module covers the fundamental single-neuron tools like peristimulus time histograms (PSTHs) and directional tuning curves. These are the building blocks you'll use to develop intuition about your data, and they're some of the most effective ways to communicate what neural activity looks like. 

Peristimulus time histograms (PSTHs)
++++++++++++++++++++++++++++++++++++

A peristimulus time histogram (PSTH) captures how a neuron's firing rate changes over time relative to a behavioral event. This plot is made by first aligning spikes across many trials to the same event like a go cue or movement onset and then averaging. This can reveal the reliable temporal structure of the neuron's response that may be hard to see on any single trial due to the inherent variability of spiking. The tradeoff is that by averaging, we collapse information across trials so we lose information about trial-to-trial variability and any changes in the response over the course of a session.

1. We'll start with the existing function you have to make a raster plot. Now, chose one channel. Write a new modified raster plot function where each row of the raster plot is the activity of this one channel for a single trial. Additionally, make a separate subplot for each target direction. Align each trial to the go cue and plot 400 ms before the go cue to the 500 ms after the go cue. Add a vertical line to each subplot to mark the go cue time. Plot each separate subplot in a 3x3 arrangement such that the subplot location corresponds to the target direction.

2. Now, instead of a raster plot to each target, plot the mean firing across trials for each target as a function of time. Use a 50 ms bin size and plot the result as a bar graph. Make this plot for a few different channels. What does this plot tell you about the neuron's average response to the stimulus? Are there any time points where the neuron appears to increase or decrease its firing rate relative to baseline? Does the neuron fire the same for each target direction? 

Tuning Curves
+++++++++++++

A tuning curve describes how a neuron's firing rate varies with a behavioral parameter like movement direction. Georgopoulos and colleagues showed in 1982 that most directionally selective cells in monkey motor cortex were well described by a cosine function of movement direction `in his 1982 paper. <https://pubmed.ncbi.nlm.nih.gov/7143039/>`_ The cosine model is a simplification, but it provides a compact summary of directional selectivity and is often a standard starting point for population-level decoding. Additionally, many BCI decoders are built upon directional tuning.

The PSTH shows you how firing rate changes over time. A tuning curve shows you how firing rate changes relative to target direction. We will collapse information in the time dimension by averaging firing rate within a window of interest, and plots that average as a function of direction.

Here, we model each neuron's firing rate :math:`f` as a cosine function of the movement direction :math:`\theta_m`:

.. math::

   f = b_0 + k \cos(\theta_{pd} - \theta_m)

where:

- :math:`b_0` is the **baseline firing rate**: the neuron's average activity across all directions
- :math:`k` is the **modulation depth**: how much the firing rate varies with direction
- :math:`\theta_{pd}` is the **preferred direction**: the direction that produces the highest firing rate

With this simple model we're able to capture a good chunk of the variance in real neural data. 

Fitting the Model
------------------

1. To fit a tuning curve, we need a single firing rate value for each of the 8 target directions. The standard approach is to average the firing rate across all trials to each target. To start, get the average firing rate across each successful trial for one target direction for a specific neuron. Use the 50 ms before go cue and 300 ms after go cue for each trial. Repeat for each of the target directions so you have 8 mean firing rates (one per target). Plot the average firing rate on the y-axis with target direction on the x-axis. This plot is the start of the tuning curve. Make this plot for a few different neurons.

To go beyond a plot and actually quantify the tuning, we fit the cosine model to these data. The cosine tuning equation as written above isn't directly solvable with standard regression because the preferred direction :math:`\theta_{pd}` appears inside the cosine. But a trigonometric identity lets us rewrite it into a form that is.

Apply the cosine difference identity:

.. math::

   \cos(\theta_{pd} - \theta_m) = \cos(\theta_{pd})\cos(\theta_m) + \sin(\theta_{pd})\sin(\theta_m)

Substituting back into the tuning model and letting :math:`b_x = k\cos(\theta_{pd})` and :math:`b_y = k\sin(\theta_{pd})`:

.. math::

   f = b_0 + b_x \cos(\theta_m) + b_y \sin(\theta_m)

This is now a linear equation in three unknowns: :math:`b_0`, :math:`b_x`, and :math:`b_y`. Since we have 8 target directions, each with an average firing rate, we can write the full system in matrix form:

.. math::

   \mathbf{f} = X \boldsymbol{\beta}

where :math:`\mathbf{f}` is the :math:`8 \times 1` vector of mean firing rates (one per target direction), :math:`\boldsymbol{\beta} = [b_0,\; b_x,\; b_y]^T`, and :math:`X` is the :math:`8 \times 3` design matrix built from the 8 target angles:

.. math::

   X = \begin{bmatrix}
   1 & \cos(0°) & \sin(0°) \\
   1 & \cos(45°) & \sin(45°) \\
   1 & \cos(90°) & \sin(90°) \\
   \vdots & \vdots & \vdots \\
   1 & \cos(315°) & \sin(315°)
   \end{bmatrix}

The ordinary least squares solution minimizes the sum of squared errors between the model predictions and the observed mean firing rates. It has a closed-form solution:

.. math::

   \hat{\boldsymbol{\beta}} = (X^T X)^{-1} X^T \mathbf{f}

In code, this entire operation is a single call — ``np.linalg.lstsq(X, f)`` in Python. Once you have :math:`\hat{\boldsymbol{\beta}} = [\hat{b}_0,\; \hat{b}_x,\; \hat{b}_y]^T`, you recover the original tuning parameters:

.. math::

   k = \sqrt{b_x^2 + b_y^2}, \qquad \theta_{pd} = \text{atan2}(b_y,\; b_x)

3. Now, write a function to fit a tuning curve to your target averaged neural activity. Plot the curve on top of the target averages. How well does the curve match the individual datapoints?
   
    a. Compute the preferred direction of the tuning curve. Add a vertical line to your plot at the preferred direction. Where on the tuning curve does the preferred direction lie?
    
    b. Make this plot for a few different neurons. Are some fits better than others?

4. Now, we want to plot the tuning across the entire population. Start by fitting a tuning curve to each neuron. On a single rose compass plot, plot the preferred direction of each neuron, scaled by the modulation depth. That is, for each neuron there should be a line from the origin in the preferred direction. The length of the line is the modulation depth of that neuron.

    a. Make a separate plot where each preferred direction has unit length.

These compass plots can tell you similar things about the population but may emphasize different characteristics. The scaled plot shows you which directions have strongly tuned neurons. A long line means that neuron modulates its firing rate substantially with direction. The unit-length plot strips away modulation depth and shows only the directional coverage. This is useful for spotting biases in your population. If the lines are roughly evenly distributed around the circle, the population has good coverage of all directions. If they cluster in one region, the population will be better at discriminating movements in that part of the workspace and worse elsewhere. (this can be important for BCI decoding!)

Together with the single-neuron tuning curve plots, these population summaries give you a picture of how direction information is distributed across your recording. Some neurons will be sharply tuned with clear cosine fits and others will typically be noisy or flat. The compass plots let you see at a glance how many neurons are contributing meaningful directional information and whether the population as a whole covers the full range of movement directions.

