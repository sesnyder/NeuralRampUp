..  _audience:

----------------
Who this is for
----------------

This tutorial is primarily written for new trainees entering SCABBY labs. I assume a mixed skill level: some of you may have extensive programming experience but limited neuroscience background, while others may be coming from a neuroscience background with less coding and formal math experience. The modules are designed to get one up-to-speed on the common approaches used in the labs.

Prerequisites
-------------

You don't need to be an expert in any of the following, but having some baseline familiarity will make your life much easier. If you are missing experience in any of the following, I highly recommend completing one of the attached subject-specific tutorials before starting these exercises.

Programming
~~~~~~~~~~~

You should have basic experience with at least one scientific programming language. This includes basic data manipulation and visualization. The popular choices are either Python or MATLAB. In Python, familiarity with the NumPy, Pandas, and Matplotlib libraries will be helpful.

If you're new to coding, you have an almost overwhelming amount of options for beginner tutorials. If you're new to Python and want to get up to speed quickly for data science work, `Python Like You Mean It <https://www.pythonlikeyoumeanit.com/>`_ is an excellent free resource. The MIT opencourseware option for beginniner Python is also very good. For MATLAB, MathWorks has released a `series of tutorial videos <https://www.mathworks.com/support/learn-with-matlab-tutorials.html>`_.

.. note:: 
    
    **MATLAB vs. Python**

    There's not really a right answer to this one.

    **MATLAB** has a long history in academic neuroscience. Much of our existing rig code (i.e., our software that interfaces with the hardware that actually controls the experiment that an animal is doing) is written in MATLAB. If you're going to be modifying task code to run your own experiments, you'll need at least a working familiarity with MATLAB. MATLAB also has a strong ecosystem of toolboxes and a well-established pipeline for the kind of signal processing and analysis work we do. Additionally, MATLAB comes with an IDE that is well-suited for data science work right out of the box.

    However, **Python** has become increasingly dominant in the broader scientific computing world, and many of the newest tools for neural data analysis (e.g., deep learning frameworks, some advanced visualization libraries, the NWB project?) are being created in Python. Also, MATLAB is largely only used in academia, so if you're thinking at all about a career in industry after your time in the lab, then it is worth using Python as this experience is far more transferable. However, configuring a proper Python project environment is slightly more hands-on compared to firing up MATLAB if you don't have any coding experience.

    My personal opinion is that starting with Python now keeps your options open for the future, but if you're totally new to programming than it may be easier to get started in MATLAB.


The dataset is provided in both MATLAB and Python so the exercises can be completed in any language.

Linear Algebra
~~~~~~~~~~~~~~
Many of the analytical techniques in this tutorial (PCA, factor analysis, Kalman filters, and more) rely on matrix operations. You should be comfortable with the following: matrix multiplication, transposes, inverses, eigen decomposition. If you need a refresher, Gilbert Strang's *Linear Algebra and Its Applications* is a classic (you can find a PDF online), and his `MIT OpenCourseWare lectures <https://ocw.mit.edu/courses/18-06-linear-algebra-spring-2010/>`_ are freely available. I also recommend looking at 3Blue1Brown's `Essence of Linear Algebra <https://www.youtube.com/playlist?list=PLZHQObOWTQDPD3MizzM2xVFitgF8hE_ab>`_ for shorter reviews of topics.

Probability and Statistics 
~~~~~~~~~~~~~~~~~~~~~~~~~~

A working knowledge of probability distributions (Gaussian, Poisson), conditional probability, and basic statistical concepts (mean, variance, covariance) will come up repeatedly. I introduce the relevant concepts as they arise, but having some prior exposure will help. I still need a good reference here (so reach out if you have one that you like!)

Neuroscience
~~~~~~~~~~~~

A basic understanding of how neurons communicate will be helpful. At a minimum, you should know what an action potential is and understand that neural activity can be recorded using electrodes. Familiarity with the idea that neurons "fire" in response to stimuli and that firing rates can carry information will also be helpful. Ref here?

