# med100kx8_no_db
The presented program is a modular and multithreaded data collection and analysis application for psychokinesis experiments, using a true random number generator (the med100kx8) as a device under test. It's written in Python. This is a simplified version of the broader med100kx8, which stores a variety of data to a database. This narrowed-down version stores no data. The expanded version, which stores a broad range of real-time data for later analysis, can be found [here](https://github.com/danosb/quantum_influence/tree/main/med100kx8).


## Program Summary

The program overview is as follows:

1. The user inputs basic data to configure the run.

1. The program connects to the random number generator, gets the raw bytes from it, and converts these bytes to binary strings of 1s and 0s.

1. It uses Scott Wilber's Random Walk Bias Amplification (RWBA) method with advanced processing methods for increased effect size in anomalous cognition [details](https://drive.google.com/file/d/1rP7Ee35K0kbQ3zXCcZvnj3d5WqmROTT_/view). By default we use a bound of 31, which is the distance to the upper or lower bound. Basically every 1 that's generated is a step to the right, every 0 generated is a step to the left. This runs until either the upper or lower bound of 31 / -31 is hit. We track how many steps it took to hit the bound and which bound was hit.

1. Results are then grouped into subtrials, trials, windows, window groups, and supertrials. A subtrial consists of one attempt to hit an upper or lower bound. By default there are 21 subtrials per trial, 5 trials per window, 10 windows per window group. A window lasts roughly 1 second. A supertrial equals one overall program execution, which can be configured either to execute indefinitely or based on a configured amount of seconds.

1. For each trial, window, and window group we calculate p and z values to determine probability of randomness. Both one-tailed (targeting either just more 1s or just more 0s) and two-tailed (targeting both more 1s and more 0s) are supported in a given supertrial. 

1. Calculated probability values are used to control a graphic window that displays a spinning cube, a bar-chart, and if two-tailed is selected then an on/off light. Behavior of objects and displayed values is controlled by window group probabilities. The goal is to influence the output of the random number generator in such a way so as to reach targets in the graphical window. 

1. This program makes use of a multi-threaded design for concurrent processing of data extraction and analysis. Speed is of critical importance because we want to use all or mostly all of the bits generated by the random number generator.

## Understanding console output:

**z-value**, **p-value**, and **surprisal value** are all just different ways of viewing the probability of the selected outcome occurring randomly. You can convert between them with algorithms. In the context of probability theory, **z-value** and **p-value** are very common. 
- **z-value** (also known as a z-score) represents the number of standard deviations an observation or statistic is from the mean of a standard normal distribution. 
- **p-value** is the probability of observing a value as extreme as, or more extreme than, the z-score under the null hypothesis. This is a value between 0 and 1 and as such can be easily translated to a percentage (e.g. p=0.05 means 5% probability of results being random).
- **surprisal value** is less commonly used. In the context of Scott's paper here, the **surprisal value** (SV) is calculated from the p-value and uses weights to provide a more uniform range of probabilistic outcomes.

The thing you'll see used most in other studies, and perhaps the simplest to understand, is the **p-value**. A commonly used threshold to denote statistical significance is p < 0.05 (5% probability of being random). p < 0.005 (0.5% chance of being random) is considered **very significant**. It's important to note that in the code I've shared above I made a choice to invert the **p-value** (1-p). So, instead of targeting p < 0.05 we instead target p > 0.95. I found this to be more intuitive since I wanted the cube to spin faster when the results were less probable, and it felt more natural to have the faster spinning speed associated with a higher numeric value.

It may seem confusing that we are using **subtrials**, **trials**, **windows**, and **window groups**. In effect, each of these is an aggregation of the thing before it (a group of subtrials is a **trial**, a group of trials is a **window**, a group of windows is a **window group**). Studies have shown the effects of psi to be small, but statistically significant over large data-sets. It takes a lot of data to ensure that an anomalous effect is really occurring (e.g., if I flip a coin 4 times, and get heads 3 times, the probability of this happening randomly is quite high). As such, tests require a lot of data, and subsequent statistical analysis on that date. The general purpose and intent of Scott's approach here is to amplify and identify the impact of any anomalous (aka psychokinetic, or psi) effects. If we get a positive result in a single subtrial, or even trial, in isolation it doesn't tell us much, but looking at large volumes of subtrial outcomes over time (via windows or window groups) gives us much more data and thus we can evaluate the probability of anomalous effect (psi) occurring with much greater confidence.

Lastly, to reference the two other values returned in the console during program execution...
- **Last window was hit? (yes/no)** - This only applies to, and is only returned during, one-tailed analysis (when doing uni-directional targeting - only more 1s or alternatively only more 0s). For one-tailed analysis, a window hit is determined based on whether the **z-value** for that window has a polarity (positive or negative value) corresponding to the target direction (if targeting more 1s then a positive z-value for the window corresponds to a *hit*/*success*, if targeting more 0s then a negative z-value is a *hit*). In one-tailed we use the number of hits within a window group to calculate window group p and z values. For two-tailed analysis (where we watch for both more 1s and more 0s) window hits are not used to calculate window group p or z values. 
- **Running overall window bound tracker** - For both one-tailed and two-tailed, in effect, this tracks consecutive z-value polarity for successive window outcomes. So, if a window has a z-value that's positive then our bound tracker adds 1. If a window has a z-value that's negative then we subtract one. The tracker maxes out at +5 and -5 (e.g., if it's at -5 and we get a z-value < 0 then we do not subtract 1). This running bound isn't actually used for anything meaningful, neither in terms of probability calculations nor visual effects in the graphical window.


## Files

Here's a brief description of the files:

* **main.py**: The main execution script for the program. 
* **extract_numbers.py**: Extracts raw bytes from the random number generator and converts them into binary strings.
* **analyze_subtrial.py**: Performs the random walk basis analysis to determine whether upper or lower bound is hit, and in how many steps.
* **process_trial.py**: Combines subtrial data into trials and performs some analysis.
* **graphic.py**: Generates and controls the graphic window with the spinning cube, bar-chart, on/off switch, and displayed data.
* **p_and_z_funcs.py**: Used to calculate probabilities of trials, windows, and window groups.
* **participant.py**: Prompts the user for input data (personal, environtal, supertrial configurations).

## Setup Instructions

1. Ensure that pip and Python 3.6 or later are installed
1. Run the following:
   
    ```pip install pyftdi datetime scipy requests pytz libusb urllib3 futures pygame numpy pyopengl libusb1 PyOpenGL_accelerate pymysql```
   
    ```pip install playsound==1.2.2```
   
## Windows Setup Additional Instructions

1. Install FTDI drivers: https://ftdichip.com/wp-content/uploads/2021/08/CDM212364_Setup.zip (not sure if this is actually required)

2. Download https://zadig.akeo.ie/
- Options > List all devices
- Find med100kx8
- Change driver to libusb-win32 and click Replace


## OSX Setup Additional Instructions

1. Nothing additional required. Device debugging instructions can be found in the device_debug subfolder.


## Running

1. Navigate to quantum_influence\med100kx8_no_db directory in terminal
```python main.py```
or 
```python3 main.py```


## Demo (click image)

Two-tailed (turn device on/off) with indefinite run-time selected (click for video):

[![Demo on Youtube](https://i.ibb.co/h8dbbnC/ss.png))](https://youtu.be/wIXsSdyPN0Y)


## Contributing
Contributions are welcome. Please submit a Pull Request or open an issue for any enhancements, bug fixes, or feature requests.
