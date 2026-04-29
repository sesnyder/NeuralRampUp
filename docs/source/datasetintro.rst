..  _datasetintro:

--------------------
Dataset Introduction
--------------------

Before you can get started with analyzing the data, you need to understand what you're working with. This module walks through the structure of the tutorial dataset.

The Experiment
--------------

For these exercises, I've included a basic recordings from a standard "center-out task". Here, a monkey is making movements to different targets. Each movement is considered a trial. The animal's hand movements are used to control the movement of a computer cursor that he sees on a screen in front of him. To start each trial, the animal must hold his hand in the center of his workspace. Then, after a variable delay, he gets a go-cue and must move his hand towards one of eight targets that are evenly spaced at 45° increments on a ring around the center. When they reach the peripheral target, they get a reward and it is considered a success. If they do not reach the target window, it is considered a failure. Additionally, I record from neural activity in the motor cortex while the animal performs this task.

File Organization
-----------------

An experimental session is stored as a single file (Either a `.pkl` (Python) or a `struct` MATLAB. The file is organized such that each row is a single trial (one attempted reach). 

To begin, load the data and take a look at how it is structured. Each column contains a different type of information about the trial. Below, I've walked through what each column contains and how to interpret it.

Trial Metadata
++++++++++++++

    **index** — The trial number within the session. Trials are ordered chronologically.

    **trialSuccess** — Whether the subject successfully acquired the peripheral target. `1` for success, `0` for failure.

    **centerTarget** - The center target location and size, stored as x-coordinate, y-coordinate and the target radius. Units are in centimeters on the screen.

    **reachTarget** — The peripheral target location and size, stored in the same format as the centerTarget (x-coordinate, y-coordinate, the target radius).

    **timeStartCenter** — The time (in ms) that the animal acquires the center hold target. 

    **timeGoCue** — The time (in ms) that the animal may move from the center target to the peripheral target.

    **timeTrialSuccess** — The time (in ms) that the animal successfully acquires the peripheral target. 

    **timeTrialFail** - The time (in ms) that the trial timed out due to the animal not acquiring the target.


Behavioral Data
+++++++++++++++

    **markerPosition** — A `t × 3` matrix of the cursor's x, y and z position at each time step during the trial. Each row corresponds to one time bin.

    **markerTimes** — A vector of the times (in ms) that the marker was sampled at. Each entry corresponds to an entry in markerPosition.

Neural Data
+++++++++++

    In the lab, we typically record neural activity using Utah microelectrode arrays. These arrays have a number of electrodes that each independently record the voltage activity from nearby neurons. We record "threshold crossings", which involves recording the timestamp each time that the voltage signal crosses a specific threshold. This voltage threshold is set such that it most likely arises from action potentials (AKA "spikes") of only the neuron or neurons that are in close proximity to the electrode tip. A more thorough option for processing this is to look at the shape of the waveform around the threshold crossing. Waveforms that result for the same neuron presumably have similar shapes, and thus the waveforms can be sorted to isolate neural activity that more likely results from the same single unit. The downside of this is that spike sorting is very time-intensive, and cannot be used for online applications (like BCI). 

    **spikeTimes** —  A dict or struct that has the timing of each threshold crossing (in ms) for a specific channel of recorded neural activity.
    
In the next section, we'll talk about how to explore the session to get a sense of how the animal did and how to extract the key quantities from this dataset that you'll need for later analyses, including how to compute the target angle and how to convert the spike times into binned firing rates.