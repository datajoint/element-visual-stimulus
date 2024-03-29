# Tutorials

## Video Tutorial

[![tutorial](https://img.youtube.com/vi/dtEggZX9Fw4/0.jpg)](https://www.youtube.com/watch?v=dtEggZX9Fw4)

## Run in MATLAB

+ Requirement: MATLAB R2016b+

### Step 1 - Initialize screen

```matlab
>> stimulus.open
```

### Step 2 - Generate stimulus conditions and queue trials

Stimulus trials are generated and queued by the scripts in the `+stimulus/+conf`
directory.  You need to know which configuration script needs to be run.

For example, to prepare the `grate` stimulus, run

```matlab
>> stimulus.conf.grate
```

While the stimulus is loaded, you will see a sequence of dots `.` and asterisks `*`,
which respectively indicate whether the conditions are computed anew or are loaded from
the database.  Some stimuli take a long time to compute and you might like to run the
configuration before you begin the experiment.  On subsequent runs, the computed stimuli
will be loaded from the database and will not take as long.

### Step 3 - Run the stimulus

The stimulus must be run for a specific scan in the `experiment.Scan` table.  Table
`experiment.Scan` contains an example entry that can be used for testing. Its primary key
is `struct('animal_id', 0, 'session', 0, 'scan_idx', 0)`. During the experiment, the
correct scan identification must be provided.

The following command will run the queued stimulus trials for the example scan.

```matlab
>> stimulus.run(struct('animal_id', 0, 'session', 0, 'scan_idx', 0))
```

### Step 4 - Interrupt and resume the stimulus

+ While the stimulus is playing, you can interrupt with `Ctrl+c`. The stimulus program
will handle this event, cancel the ongoing trial, and clear the screen.

+ To resume the stimulus, repeat the `stimulus.run` call above.  Or to queue a new set 
of trials, run the configuration script again.

### Step 5 - Exit

```matlab
>> stimulus.close
```

## Run in Python

The stimulus configuration and playback are written and executed in MATLAB. However, the
control software can be written in Python to reproduce the steps above.

### Step 1 - Configure

Configure the MATLAB Engine API for Python as described in the 
[MathWorks documentation](https://www.mathworks.com/help/matlab/matlab_external/install-the-matlab-engine-for-python.html).

### Step 2 - Import packages

```python
import matlab.engine as eng
mat = eng.start_matlab()
```

### Step 3 - Initialize screen

```python
mat.stimulus.open(nargout=0)            
```

### Step 4 - Initialize conditions and queue trials

```python
mat.stimulus.conf.grate(nargout=0)  
```

### Step 5 - Run the stimulus for a specific scan

```python
f = mat.stimulus.run(dict(animal_id=0, session=0, scan_idx=0), nargout=0, async=True)
```

### Step 6 - Interrupt and resume stimulus

```python
f.cancel()
f = mat.stimulus.run(dict(animal_id=0, session=0, scan_idx=0), nargout=0, async=True)
```

### Step 7 - Exit

```python
f.done()  # True if stimulus is done
f.result()  # Waits until the stimulus is done
f.stimulus.close(nargout=0)  # Close the stimulus screen 
```

## Example queries

+ If the language is unspecified below, the queries run in both MATLAB and Python.

### All scans with any visual stimuli

```
visualScans = experiment.Scan() & stimulus.Trial()
```

### All scans with the `Monet` stimulus

```
monetScans = experiment.Scan() & (stimulus.Trial() * stimulus.Monet())
```

or

```
monetScans = experiment.Scan() & (stimulus.Trial() * stimulus.Condition() & 'stimulus_type="stimulus.Monet"')
```

### All unique conditions shown during a given scan

```python
## python
session_key = dict(session=7302)
scan_conditions = stimulus.Condition() & (stimulus.Trial() & session_key)
```

```matlab
% matlab
sessionKey = struct('session', 7302);
scanConditions = stimulus.Condition & (stimulus.Trial & sessionKey);
```
