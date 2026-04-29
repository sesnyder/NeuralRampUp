..  _behaviorvisualization:

--------------------
Visualizing Behavior
--------------------

Before running any analysis, you should spend some time just looking at the data. Here are a few exercises to walk you through how to get familiar with a dataset for the first time. I try to point out some of the things you should be paying attention to as you do so. I consider this to be a mix of sanity checks (does this data make sense or is something wrong?) and basic behavior visualizations (how was the animal doing on this day?). Additionally, I try to include some thoughts on how to start visualizing the data in a way that captures what you're actually interested in. 

.. note::
    Postdocs can probably skip this module.

Sanity Checks
+++++++++++++

1. Are any fields missing on any of the trials? If so, what do you think you shold do with those trials? What are the range of values for each field? Does that make sense? 

   Depending on the field that is missing or off, the trial should probably be excluded from any analyses that rely on that field. For example, if the `cursor_position` field is missing, then you won't be able to include that trial in any analyses that rely on the cursor position (e.g., computing movement trajectories). However, if the `time_trial_fail` field is missing, but the `trial_success` field indicates that it was a successful trial, then you can still include that trial in analyses of successful trials since the failure time is not relevant for those analyses.

2. How many trials are in this experiment session? How many trials are successes and how many are failures?
   
   This is a good check of the animal's overall performance. For example, is the session shorter or longer than you thought it would be? Did he fail a lot of trials or did he do well? If the session is very short or if the animal failed most of the trials, then you might want to consider excluding this session from your analyses.

Looking at Behavior
+++++++++++++++++++

3. The target window is a clunky way to represent target information. Can you add a column to the dataframethat has, for each trial, the angle of the peripheral target from the center starting point? 

   You can compute the target angle using the x-y coordinates of the target. The angle can be computed using the arctangent function. This will give you an angle in radians, which you can convert to degrees if you prefer. You should verify that you have exactly eight unique values for the target angle, corresponding to the eight targets that are evenly spaced at 45° increments around the center.

4. On average, how long does it take the animal to acquire the target? 

5. Make a plot for the acquisition time for each trial. Each trial should be a dot. You should put the trial number on the x-axis, and the target acquisition time on the y-axis. If the trial was a failure, then you can plot the time to failure instead. Use a different marker for failures.
   
   a. Can you color the dot depending on the target angle?
   b. Can you add a line that captures a sliding average of the acquisition time? Use a 20 trial window. (Hint: look up the pandas function "rolling")

   This is a very informative way to visualize the animal's performance over time. How does the acquisition time vary across the session? Does he get tired towards the end or stay focused? Do the failures cluster towards certain targets? Can you see target directions that were easier for him to hit? 

6. For the first trial, can you make a plot that shows the trajectory that the animal took from the center to the peripheral target? (i.e., plot the cursor position). You should draw the target on your plot as well and color the target and the trajectory by the color you used in the last question. 
   
   a. Now, on one figure, plot the movement trajectories for all of the trials in the experiment. Draw the 8 targets. Each trajectory should be colored by the target the animal is cued to move towards on that trial. If the animal doesn't reach the target, mark that movement trajectory with a dashed line instead.
   b. Repeat 7a with separate plots for the first 40 trials, a 40 trial window in the middle of the experiment and the last 40 trials. Is this version cleaner?

   Consider the trajectory plots you just made with the acquisition time plots you made in 6. Do you see any relationship between the movement trajectories and the acquisition times? For example, do the trials with longer acquisition times also have more circuitous trajectories whereas the faster acquisition times correspond to more direct trajectories? 
   
In the next module, we'll talk about how to go through a similar process for neural data, as well as how to extract the key quantities that you'll need for later analyses, including how to convert the spike times into binned firing rates.