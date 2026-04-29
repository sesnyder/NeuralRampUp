..  _neuralstart:

--------------------------------
Getting Started with Neural Data
--------------------------------

Once you've verified your behavioral data, you're ready to start working with the neural recordings themselves. As with the behavioral data, it is important to first verify that the neural data looks reasonable before diving into any analyses. This section walks you through how to visualize the raw spike times and how to convert those spike times into binned firing rates, which are the most common way to summarize neural activity for later analyses.

Artifact Detection and Removal
++++++++++++++++++++++++++++++

1. To start, write a function that takes a trial number as an input and then generates a plot of the spikes times on each channel for that trial of the session. You should have channel number on the y-axis and time in the trial on the x-axis. Draw a thin vertical line for each spike time. This is called a raster plot and is a common way to visualize spike times across channels. 
   
   a. Make a raster plot for trial 1. Are there spikes on every channel? 
   b. Now make a plot for trial 325. Are there any time points where many channels have spikes at the exact same time? 
   
   This is called "coincidence spiking" and is often a sign of an artifact in the recording rather than real neural activity. Our electrodes can sometimes pick up electrical activity from muscle movements of the head or neck and register them as neural activity. If you see this, it is critical to identify and handle it before moving on to any downstream analysis. In later modules, we will discuss why this particular artifact is so problematic, but for now, just know that it is something you need to be able to spot.

2. Write a function to detect the timestamp of artifacts. We'll consider an artifact to be any single ms timestamp that has spiking across more than 80% of the channels. The coincidence spiking percentage should be an adjustable parameter. The function should iterate over the entire session and return a dict of trials and timestamps that are considered artifacts based on the coincidence spiking threshold.  
   a. Update your raster plot function so that spikes that are considered artifacts are plotted in red. Take another look at trial 325 and play with the threshold value a bit. Do you think 80% works?

3. Now let's consider how coincident activity is distributed across pairs of channels. For a given pair of channels, compute the percentage of spikes that occur at the same timestamp across all of the trials. Specifically, find the number of shared spike times between the two channels and divide by the total number of unique spike times on that channel. Test this on a few pairs of channels. What range of values do you see?
   
   a. Now compute this coincidence rate for every pair of channels and store the result in an N x N matrix, where N is the number of channels. Plot the matrix as a heatmap. 
   b. Write a function to remove the artifacts you detected in the previous question. Now recompute the coincidence rate heatmap. Plot the two matrices side by side. How did the distribution of coincidence rates change? Are there any channel pairs that still show high coincidence rates after cleaning? 
   
   We've now idenified a different artifact type to be aware of. Here, the issue is that a specific electrode pair are electrically coupled. This can happen when there is a physical short between electrodes. When this happens, the two channels will produce highly correlated spike trains. Not because two distinct neurons are firing together, but rather that both channels are detecting the same spikes from the same source. This is one reason we look at the coincidence matrix from the previous exercise even after cleaning: a pair of channels that still shows a high coincidence rate after removing global artifacts may be electrically coupled. In practice, this means you should be cautious about treating those channels as independent in any downstream analysis when they are likely not.

4. To treat this issue, you should exclude one or both of the channels in the pair from later analyses. Write a function that takes in the coincidence matrix and a threshold and returns a list of channels to exclude based on pairs that exceed the threshold. Make it an option to exclude both channels in the pair or just one. If you choose to exclude just one, make sure to only exclude one channel from each pair and not both.
   
   Again, we will review in subsequent modules why dealing with this type of artifact is so important, but for now just know that you should be on the lookout for this issue and have a plan for how to handle it when you see it.

.. note:: **Subsequent analyses should be performed on this cleaned up version of the dataset!**

Binning Spike Times into Firing Rates
++++++++++++++++++++++++++++++++++++++

We start with neural data that is recorded as a list of precise spike times, but many analyses require us to convert these into spike counts over discrete time windows. 

1. To start, write a function that takes in a list of spike times for a single channel and a time window (e.g., from -500 ms to +1500 ms relative to the go cue) and returns the number of spikes that occur within that window. Test your function on a few channels and trials to make sure it is working correctly.

2. Now divide the full trial period into equal-sized bins and count the number of spikes in each bin. Start with a bin size of 100 ms. Plot the spike counts over time. On the y-axis, plot the spike count for each bin as a bar and on the x-axis plot the time bins. Make this plot for 10 random channels. Then repeat with bin sizes of 50 ms, 10 ms, and 1 ms for the same 10 channels. How does the choice of bin size affect the shape of the result? At what point does the signal start to look noisy, and why?

3. Use the code you wrote for a question 1 and 2 to make a function to compute binned spike counts for a trial. This function should have bin size in ms as an input parameter. The output should be a matrix where each row is a channel and each column is a time bin. Use this function with a 50 ms bin size and add the binned spike count matrix for each trial as a column to the dataframe.  Add the bin times (in ms, relative to the start of the trial as 0 ms) as a separate column as well.


.. note:: 

    Many trainees start by working with data that they did not collect. The above exercises will get you familiar with the data but often still don't tell the whole story. There are always details about the experimental setup, known issues with specific sessions, or conventions in how things are labeled that aren't always captured in the file itself. If notes or an experiment log exist for a session, read them before you start analyzing. Additionally, try to talk to the person who actually collected the data. It is important to be aware of these details early before diving in to more complicated analyses.