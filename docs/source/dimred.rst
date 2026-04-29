..  _dimred:

------------------------
Dimensionality Reduction
------------------------

Ok now we are getting to the meat and potatoes. Dimensionality reduction builds on the idea that the brain uses populations of neurons whose activity is coordinated and correlated. It is the set of tools that lets us look at that coordinated population activity directly.

Why Dimensionality Reduction?
-----------------------------

Let's start by considering our full neural space. The neural activity in this dataset was recorded from 2 separate 64 channel arrays (128 channels in total. When we record from 128 channels simultaneously, each time point of neural activity lives in a 128-dimensional space. One axis captures the activity of one neuron. However, we can't visualize 128 dimensions, and even if we could, most of that space is also empty or captures noise. Importantly, the most important parts of the neural activity tend to live in a much lower-dimensional subspace. This is because neurons are correlated with each other, and those correlations constrain the population activity to a (relatively) small section of this total state space.

Dimensionality reduction is about finding that smaller subspace. It allows us to take our high-dimensional data and transform it into a smaller number of dimensions that capture the structure we care about. This gives us a compact way to visualize neural activity, compare across conditions, and use as the basis for further analysis.

For a more thorough review of how these methods are used in neuroscience, see `this review <https://www.nature.com/articles/nn.3776>`_ by Cunningham & Yu, which covers the scientific motivations for population analyses and offers some really good practical guidance on selecting and interpreting dimensionality reduction methods. 

The Covariance Matrix
---------------------

Every dimensionality reduction method in this module starts from the same place: a matrix of firing rates (or spike counts) where rows are observations (time bins or trials) and columns are neurons. The key statistical object underlying most of these methods is the **covariance matrix**. We can often think of different techniques as different ways of breaking down the covariance matrix.

Computing the Covariance Matrix
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

The covariance matrix :math:`\Sigma` captures how every pair of neurons co-varies across observations. If you have a data matrix :math:`X` of shape :math:`n \times p` (number of observations by number of neurons), the covariance matrix is:

.. math::

   \Sigma = \frac{1}{n-1}(X - \bar{X})^T(X - \bar{X})

where :math:`\bar{X}` is the mean across observations for each neuron. This gives you a :math:`p \times p` symmetric matrix. In our case, the matrix is :math:`128 \times 128` (although not exactly 128 channels since you turned off some earlier!). 

Each entry :math:`\Sigma_{ij}` tells you how much neurons :math:`i` and :math:`j` tend to fluctuate together across trials. The diagonal entries :math:`\Sigma_{ii}` are each neuron's variance. The off-diagonal entries are the pairwise covariances — positive if two neurons tend to increase and decrease together, negative if one goes up when the other goes down, and near zero if they vary independently. Dimensionality reduction is, at its core, a structured way to summarize the trends that this matrix captures.

.. warning:: **WARNING**

   This is why we care so much about making sure we don't have artifacts that introduce correlations between recorded channels of neural activity. They wreak havoc on our covariance estimates!

In code, getting your covariance matrix is pretty straightforward. You can call ``np.cov(X)`` on your data matrix of spikes.

1. Compute the covariance matrix. Start by getting a data matrix that excludes the artefact channels you identified earler. For each trial, use the 200 ms before go cue and 500 ms after go cue.

Eigendecomposition
------------------

The covariance matrix can be decomposed into its eigenvalues and eigenvectors:

.. math::

   \Sigma = V \Lambda V^T

where :math:`V` is a matrix whose columns are the eigenvectors (the principal directions of variation) and :math:`\Lambda` is a diagonal matrix of eigenvalues (the variance along each direction). The eigenvectors are orthogonal and ordered by how much variance they explain. The first eigenvector points in the direction of maximum variance in the population; the second points in the direction of maximum remaining variance orthogonal to the first; and so on.

Scree Plot
%%%%%%%%%%

A scree plot shows the eigenvalues in descending order. It tells you how the total variance is distributed across dimensions: does most of the variance concentrate in a few dimensions (suggesting low-dimensional structure), or is it spread evenly (suggesting the data is high-dimensional)?

2. Decompose your covariance matrix and generate a scree plot. There are commands in both python and MATLAB to perform the eigen decomposition. On the x-axis, plot the dimension and on the y-axis plot the eigenvalues. For this reaching dataset, how is the variance distributed? 

Principal Component Analysis (PCA)
----------------------------------

PCA finds the orthogonal axes that capture the most variance in the data. It's the simplest and probably the most widely used dimensionality reduction method you will come across.

Mathematically, PCA projects each data point :math:`x_n` (a :math:`p`-dimensional firing rate vector) onto the top :math:`M` eigenvectors of the covariance matrix:

.. math::

   z_n = U_M^T (x_n - \mu)

where :math:`U_M` is the matrix of the top :math:`M` eigenvectors and :math:`\mu` is the data mean. The result :math:`z_n` is an :math:`M`-dimensional score for that observation.

3. Ok let's fit PCA to our data. Once again, there are functions in python and MATLAB to do this. Start by fitting a model with 10 dimensions. Does the dimensionality seem reasonable considering what your scree plot looks like? What percentage of the total variance are captured by these 10 dimensions?

4. Now, let's visualize neural activity in the top components. Project each time point into the top 2 dimensions using the above equation. Don't forgot to subtract the means! Your plot should be centered around zero. Color each time point by the target direction. Can you see clusters in this state space based on target direction?

    a. Now, make the same plot using the other components. Can you still see clusters based on target direction? 

Factor Analysis (FA)
--------------------

Factor analysis is a related technique that is heavily used in the lab. On it's surface, factor analysis looks similar to PCA. As in PCA, you get lower-dimensinoal subspace that you can project your data into. However, the underlying model is very different.

PCA treats all variance the same. It finds the directions of maximum total variance, regardless of whether that variance reflects coordinated fluctuations across neurons (signal) or independent noise within individual neurons. One downside of this is that neurons with a higher average firing rate matter more for PCA because they have more total variance.

Factor analysis explicitly separates these two sources of variance. It models the data :math:`x` as:

.. math::

   x = Wz + \mu + \epsilon

where :math:`z` is a low-dimensional latent variable (the shared factors), :math:`W` maps from the latent space to the observed neurons, and :math:`\epsilon` is neuron-specific noise drawn from :math:`\mathcal{N}(0, \Psi)` with :math:`\Psi` diagonal. The key constraint is that :math:`\Psi` is diagonal. This means that each neuron has its own independent noise variance (called the private variance), separate from the shared variance captured by the factors.

Why does this matter? In neural data, each neuron contributes two kinds of variability: shared fluctuations that reflect coordinated population activity (the signal we usually care about) and private fluctuations that are specific to that neuron (spiking noise, electrode noise, etc.). In PCA, these two are lumped together. FA pulls them apart. For this reason, our BCI decoders use FA instead of PCA to define a low-dimensional space for learning (i.e. `Sadtler <https://www.nature.com/articles/nature13665>`_, `Golub <https://www.nature.com/articles/s41593-018-0095-3>`_ and `Oby <https://www.pnas.org/doi/10.1073/pnas.1820296116>`_). In practice, the difference is most noticeable when neurons have very different noise levels. A neuron with high private variance will dominate PCA's top components just because it's noisy, even if it doesn't contribute much to the shared population signal.

5. Make a similar plot above showing datapoints in the top shared dimensions identified by FA. There is also an FA class in sklearn that you can call to fit and project your data. Use 10 dimensions again. What percentage of the total variance are captured by these 10 dimensions?

Compare the FA plot to the PCA plot. You may notice that the clusters are more cleanly separated in FA, or that the relative spacing of clusters changes. This is because FA has stripped out the private variance that was blurring the picture in PCA.

.. admonition:: On the term manifold

   Many papers from our lab group use the term "manifold" to refer to the low-dimensional population space that captures the variance shared across the population of interest. For example, in the BCI learning papers I linked above, perturbations are either "within the manifold" or "outside the manifold".

Selecting Dimensionality
------------------------

This is where science becomes more art. One of the most practically important decisions when applying any dimensionality reduction method to real neural data is choosing how many dimensions to actually reduce the dataset down to. Too few and you miss real structure in the population. Too many and you start fitting dimensions to noise, which can distort your understanding of the underlying geometry. In the above examples, I had you fit models with 10 dimensions, which is a reasonable starting point but may not be the best. 

Estimating the true, underlying dimensionality of a population is particularly tricky because our estimates can be affected by a number of factors that actually do not have anything to do with the dimensionality. Some examples are: the firing rates of the neurons in the population, the size of the population recorded, the task the animal is doing, and the amount of data you have. If you want to read more about this, I recommend `this really interesting paper. <https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1005141>`_ that has some great insights into how to approach estimating dimensionality.

The Elbow Method (Variance Explained)
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

The simplest approach involves interpreting the scree you generated earlier. Take a look at your scree plot (make sure you have a line connecting the top of each bar if you didn't earlier). You then look for an "elbow" in the line. This is the point where adding another dimension gives sharply diminishing returns in the variance that is explained.

The pros of this approach are its simplicity and interpretability. This makes it a good first approximation. However, this approach isn't very principled. Different people looking at the same plot will pick different cutoffs. Additionally, the elbow can be ambiguous without a sharp point.

Data Log-Likelihood
%%%%%%%%%%%%%%%%%%%

One of the more principled approaches that we can use with FA involves leveraging its probabilistic structure directly. Because FA defines an explicit generative model, We can evaluate the likelihood of held-out data under models with different numbers of factors. The data likelihood is a measure of how well a model explains the data, and is calculated as the probability of the data occuring given specific model parameters. The llikelihood calculation can be a pain but the rough procedure is as follows:

#. Split your data into a training set and a held-out test set.
#. Fit FA with different numbers of factors on the training data.
#. Evaluate the log-likelihood of the test data under each fitted model.
#. Choose the dimensionality where test log-likelihood plateaus or starts to decline.

Comparing Subspaces
-------------------

To end this section on dimensionality reduction, I want to walk through an example of how these techniques can be applied to answer a question about how the brain encodes information. A `popular example <hhttps://www.nature.com/articles/ncomms13239>`_ is understanding how movements are prepared and executed in motor cortical areas. The idea is that different computations (movement preparation and execution) live in different subspaces of the high-dimensional space. 

7. To start, use PCA to find a low-D subspace on the neural data from 300 ms before go cue to the go cue. We'll call this our preparatory activity. Find a separate low-D subspace for the activity from 200 ms after the go cue to 500 ms after the go cue. We'll call this our execution activity. Look at the scree plot for both of these models. What is an appropriate dimensionality for each? 

One standard approach for comparing the alignment of two subspaces in the high-D neural space is to look at the principal angles. The idea is that we can generalize the idea of measuring an angle between two lines to an arbitrary number of dimensions. Typically, principal angles are ordered to return the minimum angle measured between subspaces. This is the peice of information we are interested in: what is the smallest rotation we need to begin to align the subspaces? Another way to think about this is consider if all of the principal angles are zero. This would mean that are subspaces are identical to each other.

7. Measure the principal angles between your preparation and execution subspaces. I recommend checking out the subspace_angles function in scipy!

Interpreting principal angles is often very tricky. What do you do if the subspaces are not perfectly aligned but also not fully orthogonal? For example, what is a significant angle in the high-D neural space (which is the space that we are measuring the relationship between our subspaces)? However, this is still a useful first pass tool for understanding the relationship between different subspaces. Furthermore, many of the other tools we use for this problem rely on the intuition introduced here.

Conclusion
----------

The above sections are designed to be an introduction to the intuition behind dimensionality reduction for neural data. If you are using these techniques more extensively, I highly recommend trying your hand at implementing them yourself (especially FA) to gain a richer understanding of what's going on under the hood. Sections 12.1 and 12.2 in Pattern Recognition and Machine Learning by Bishop I think are particularly helpful for this.

