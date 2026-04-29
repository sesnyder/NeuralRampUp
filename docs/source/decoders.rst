..  _decoders:

---------------------------------------
Brain-Computer Interface (BCI) Decoders
---------------------------------------

If you've made it this far you've probably heard the term BCI before. A brain-computer interface (BCI) translates neural activity into control signals n real time. In this, lab we often use a BCI to have an animal control the velocity of a computer cursor. Additinally, we often refer to the algorithm that translates neural activity into cursor velocities as a 'decoder' or 'mapping'.

BCIs are most known for their clinical utility. For example, restoring `speech <https://www.nature.com/articles/s41586-023-06377-x>`_ or `the ability to walk <https://www.nature.com/articles/s41586-023-06094-5>`_ following a spinal cord injury. However, they can also be configured to answer basic science questions about neural functinos. Used in this way, we use a BCI to ask subjects to generate specific patterns of neural activity.  

The Kalman Filter
-----------------

The Kalman filter is the main algorithm used for BCIs in the MSSCABBY lab groups. We use an "intuitive" decoder to refer to a Kalman filter that was calibrated to give the animal good control. Many BCI learning studies from the lab have relied on changing various parameters of the intuitive decoder to perturb the animal's control, and then examined specifically how the animal recovered. (linked earlier). Therefore, it is worthwhile to be intimately familiar with how a Kalman filter is structured and fit to neural data. 

A Kalman filter consists of two components: a state model that describes how the cursor movements change over time, and an observation model, which describes how neural activity relates to the cursor movements.

The State Model
^^^^^^^^^^^^^^^

For a BCI cursor in this group, we usually choose to use cursor velocity to capture the state of the cursor:

.. math::

   v_t \in \mathbb{R}^{2 \times 1}

This contains the cursor's x and y velocity. The state-space model says that the velocity at time :math:`t` is a linear function of the velocity at time :math:`t-1`, plus some noise :math:`Q`:

.. math::

   v_t | v_{t-1} \sim \mathcal{N}(A v_{t-1},\; Q)

The matrix :math:`A` captures how velocity at one time step relates to velocity at the next time step. We typically set :math:`A = I` (the identity matrix), which says that the cursor velocity is expected to stay the same from one time step to the next. The noise covariance :math:`Q` controls the smoothness of the decoded velocities over time. We've found that setting :math:`Q = 2 \, \frac{m^2}{s^2} \times I` yields pretty good control for our animals.

The Observation Model
^^^^^^^^^^^^^^^^^^^^^

The state model describes how velocity evolves, but we can't observe velocity directly in a BCI (since this is what we are trying to estimate). What we actually observe is neural activity. The observation model describes the relationship between the velocity we care about and the neural data we actually see:

.. math::

   \hat{z}_t | v_t \sim \mathcal{N}(C v_t + d,\; R)

Here :math:`\hat{z}_t \in \mathbb{R}^{p \times 1}` is the neural observation at time :math:`t`. This could be the raw spike count vector from your recorded neural activity or a lower-dimensional set of latent factors obtained from a dimensionality reduction step like factor analysis. Using latent factors rather than raw spike counts yields a similar level of control as the raw spike counts and also lets design some neat perturbations!

The matrix :math:`C \in \mathbb{R}^{p \times 2}` maps from the 2-dimensional velocity to the expected neural observations. You can think of :math:`C` as an encoding model: given that the cursor is moving at a particular velocity, :math:`C` tells you what the average neural activity should look like. The vector :math:`d \in \mathbb{R}^{p \times 1}` captures the baseline neural activity that doesn't depend on velocity. The noise covariance :math:`R \in \mathbb{R}^{p \times p}` captures the leftover neural variability that our model doesn't explain.

The parameters :math:`C`, :math:`d`, and :math:`R` are fit using maximum likelihood from training data where we have both the cursor kinematics and the neural activity recorded simultaneously.

Fitting Parameters
----------------------

To estimate :math:`C`, :math:`d`, and :math:`R`, we need paired observations of cursor velocity and neural activity from a set of calibration trials where the subject performs the task under known conditions. Concatenate the velocity vectors and neural observations across all time bins from these trials. Then fit :math:`C` and :math:`d` by regressing the neural observations onto the velocities. This is a standard linear regression. The observation noise covariance :math:`R` is the covariance of the residuals from that regression.

The Predict-Update Loop
^^^^^^^^^^^^^^^^^^^^^^^

Once the parameters are fit, the filter runs a two-step loop at every time bin in a trial. At each time step, the filter first computes a prediction of the current velocity based on the previous estimate, :math:`P(v_t \mid f_{1:t-1})`, and then incorporates the new neural data using a measurement update to obtain :math:`P(v_t \mid f_{1:t})`. Because both the prediction and observation are modeled as Gaussians, the update reduces to combining two Gaussian distributions, yielding a new Gaussian estimate. 

The relative influence of the prediction and observation is determined by the Kalman gain:

.. math::

   K_t = P_t^- C^T \left(C P_t^- C^T + R\right)^{-1}

which depends on the uncertainty in both the state estimate and the measurements. When neural observations are noisy (large :math:`R`), the filter relies more heavily on the dynamics; when the neural activity is reliable, the gain places greater weight on the neural activity. By recursively applying this predict–update procedure over time, the Kalman filter produces smooth, accurate estimates of cursor velocity.

Cleaning Things Up
------------------

Because :math:`A`, :math:`Q`, and :math:`R` are all constant in this formulation, the Kalman gain :math:`K_t` converges to a fixed value after a few time steps. Once it has converged, the gain no longer changes and the entire filter reduces to a single linear update at each time step:

.. math::

   \hat{v}_t = M_1 \hat{v}_{t-1} + M_2 x_t + m_0

where :math:`M_1 = A - KCA`, :math:`M_2 = K \beta`, and :math:`m_0 = -Kd`. Here :math:`K` is the converged (steady-state) Kalman gain and :math:`\beta` relates the raw spike counts to the latent factors.

This is a significant practical simplification. Rather than recomputing the gain and uncertainty at every time step, you can precompute :math:`M_1`, :math:`M_2`, and :math:`m_0` once and run the decoder as a simple linear recurrence. The decoded velocity at each step is just a weighted combination of the previous decoded velocity and the current neural observation. This is the form that we typically implement in our real-time BCI system.

This loop runs once per time bin. The output :math:`\hat{v}_t` at each step is the decoded velocity. Cursor position is recovered by integrating the decoded velocity using Euler's method.

Implement!
----------

Implement the Kalman filter. Fit a decoder and use it to estimate the trajectory for each trial. How does it do? Generate a plot that shows the estimated cursor trajectory and the actual cursor trajectory.