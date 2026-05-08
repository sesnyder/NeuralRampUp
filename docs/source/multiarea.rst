..  _multiarea:

--------------------
Multi-Area Analyses
--------------------

Up to this point, every analysis in this tutorial has treated the recorded population as a single group. But in many experiments (including the one that we've been using for this tutorial), electrodes are implanted in multiple cortical areas simultaneously. When you have simultaneous recordings from two populations, you can ask a new kind of question: how do these areas interact?

Here, I introduce canonical correlation analysis (CCA), which is a technique  designed to find the dimensions of activity that capture how the two populations are interacting. We'll apply it to simultaneous recordings from primary motor cortex (M1) and dorsal premotor cortex (PMd) during the center-out reaching task to show how the two areas have shared signals about the cued target.

The Communication Subspace
--------------------------

Single-area methods like PCA or FA can tell you about the structure of population activity within each area, but they can't tell you about the relationship *between* areas. Two populations could each have rich, high-dimensional activity and yet share only a narrow channel of communication. Or they could be tightly coupled along many dimensions. 

The idea of a communication subspace, introduced by `Semedo et al. (2019) <https://www.cell.com/neuron/fulltext/S0896-6273(19)30053-4?_returnURL=https%3A%2F%2Flinkinghub.elsevier.com%2Fretrieve%2Fpii%2FS0896627319300534%3Fshowall%3Dtrue>`_, is that brain areas communicate through a low-D subspace. Low-D here is assessed relative to the dimensionality of the two areas being considered. This can be thought of as an information bottleneck: area 1 might have 10 dimensions of meaningful internal activity, but only 3 or 4 of those dimensions are correlated with area 2. The rest are considered "private" to area 1. These private dimensions may still be important for local computations, but they aren't shared with the other area. 

The Semedo 2019 paper also goes on to make another critical point: the communication dimensions are not aligned with the dimensions of largest variance in an area. This means methods like PCA, which find dimensions of the largest shared variance, may miss across area signals entirely. This points to a need for specific techniques to find signals shared across areas. 

Canonical Correlation Analysis (CCA)
------------------------------------

Let's consider neural activity that is simultaneously recorded in two brain areas. That is, for an activity pattern recorded at a
given time point in area 1, there is a co-occurring activity pattern
recorded in area 2. Canonical Correlation Analysis (CCA) identifies an axis in each area such that the activity projections onto those axes are maximally correlated. The first pair of dimensions CCA finds will have the highest correlation. The second pair finds the next most correlated pair, subject to being uncorrelated with the first. And so on.

The result is a set of canonical dimensions that are paired low-dimensional signals, one from each area, that capture the shared structure between the areas. 

The Math
++++++++

Let :math:`X_1 \in \mathbb{R}^{t \times q_1}` be the matrix of area 1 activity and :math:`X_2 \in \mathbb{R}^{t \times q_2}` be the matrix of area 2 activity, both mean-centered. The activity is paired across populations at each of :math:`T` total time steps, and the correlation is taken over these :math:`T` time steps. CCA finds weight vectors :math:`a^*` and :math:`b^*` that maximize:

.. math::

   a^*, b^* = \underset{a, b}{\operatorname{argmax}} \; \operatorname{corr}(X_1 a, \; X_2 b)

Thus, the “canonical dimensions” are the vectors :math:`a^* \in \mathbb{R}^{q_1 \times 1}` in area 1 and :math:`b^* \in \mathbb{R}^{q_2 \times 1}` in area 2 that solve the above optimization. We then project the neural population activity onto these canonical dimensions:

.. math::

   w_1 &= X_1 a^* \\
   w_2 &= X_2 b^*

to obtain the "projection scores" :math:`w_1 \in \mathbb{R}^{T \times 1}` and :math:`w_2 \in \mathbb{R}^{T \times 1}`.

Applying CCA to M1 and PMd
--------------------------

M1 and PMd are densely interconnected and both play central roles in movement generation, but they have distinct functional specializations. Understanding how these areas coordinate their activity to control movements is a fundamental question in motor neuroscience.

For our center-out dataset, the spiking data you have been working with is actually from M1 and PMd. A separate Utah array was implanted in each area. Channels 1-32 and 97-128 are in M1 and and channels 33-96 are in PMd.

1. Start by separating the neural population into M1 and PMd subpopulations. Get new columns for spike information for each subpopulation. This should be formatted as binned spike counts.

2. Get a separate matrix of spike counts that is time bins by neurons for each brain area. For each trial, use the 200 ms before go cue and 500 ms after go cue. Fit CCA to this data (there are commands in both Python and MATLAB to do so) with 4 dimensions.

3. Visualize neural activity projected in the top pair of dimensions you identified. Let the x-axis be the M1 dimension and the y-axis be the PMd dimension. How correlated is activity in this dimension? 

   a. Color each time point by the target the animal is moving towards at that time. Do you see any tuning emerge?
   b. Do the same for the second and third pairs of dimensions. Are they also tuned to target direction? Does this make sense? 

