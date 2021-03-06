## Experimental Protocol

### General

**Room** EPFL, ELA 006 Anechoic Chamber

**Date** 2017 / 09 / 26

**Time Start** 12:19

**Time End** 19:46

**Team** Robin Scheibler, Hanjie Pan

### Description of the Experiment

The microphone array [pyramic](http://github.com/LCAV/pyramic) with 48 MEMS
microphones was placed on K&B turntable and rotated 360 degrees in increments
of 2 degrees for a total of 180 distinct angles. Three loudspeakers were placed
at fixed locations, all forming the same angle with the array, but with 3
different heights.

![setup](https://github.com/fakufaku/pyramic-dataset/raw/master/figures/experiment_setup.png)

A total of 8 samples were played by each loudspeaker for each angle:

* Linear sweep, `samples/sweep_lin.wav`, 48 kHz
* Exponential sweep, `samples/sweep_exp.wav`, 48 kHz
* White noise, `samples/noise.wav`, 48 kHz
* 3 female speech samples, `samples/fq_sampleX.wav`, `X=0,1,3`, 16kHz
* 2 male speech samples, `samples/fq_sampleX.wav`, `X=2,4`, 16kHz

The sweeps were generated by running

    python ./code/gen_sweeps.py

The random noise sample was generated using the `numpy.random.randn` function.

The speech samples were picked from the TIMIT corpus as follows

* `fq_sample0` : FAKS0 / SA1 (female)
* `fq_sample1` : FCMR0 / SA2 (female)
* `fq_sample2` : MDLD0 / SX13 (male)
* `fq_sample3` : FMGD0 / SX34 (female)
* `fq_sample4` : MJLN0 / SX99 (male)

All the samples were resampled at 48 kHz if necessary and stitched into one
file (`all_samples.wav`) before playback.  The playback was done at 48 kHz.

![anechoic_room](https://github.com/fakufaku/pyramic-dataset/raw/master/figures/anechoic_room_setup.jpg)

For each angle, the array was rotated by the appropriate amount
programatically using the computer interface from the turntable. 
Then, each loudspeaker played `all_samples.wav` while the Pyramic array
was recording.

The recorded file was saved as

    recordings/pyramic_spkrX_all_samples_Y.wav

where `X` is the index of the speaker with the following mapping:

    X=0 => Middle loudspeaker
    X=1 => Low loudspeaker
    X=2 => High loudspeaker

and `Y` is the counter-clockwise rotation angle in degrees of the _array_. The
corresponding relative rotation of the speaker around the array is `360 - Y`.

### Experimental Data and Equipment

The file `protocol.json` is machine readable and contains both the
environmental conditions (temperature and humidity), the material used, and the
geometry of the experiment (the relative locations of microphones and
loudspeakers).

#### Format description

We use machine readable JSON format with the following fields.

* `conditions` : Environmental conditions during the experiment
  * `start`/`end` : start and end time of experiment
* `equipment` : Equipment used during the experiment
  * `interface` : The audio interface used for the playback of sound samples
  * `speakers` : The brand and model of loudspeakers
  * `turntable` : The brand and model of the turntable on which the microphone array lies
  * `microphones` : The microphone array used for the experiment
* `geometry` : The geometry and locations of objects during the experiment
  * `microphones`
    * `locations` : A list of triplets describing the locations of the microphones relative to the center of the array
    * `height` : The height of the lowest microphone in the array with respect to the floor
    * `comment` : A short description of the other fields
  * `speakers`
    * `locations` : Location of the loudspeakers with respect to the array
      * `middle`/`low`/`high` : Names for the three loudspeakers locations
        * `distance` : Horizontal distance between the front of the loudspeaker and the center of the array
        * `height` : Vertical distance between the bottom of the wooffer baffle and floor

    * `baffles_corrections` : The vertical distance between the bottom of the woofer and the center of the `woofer` and `twitter` baffles
    * `comment` : A short description of the above fields

### Code Automating the Experiment

The scripts used for automating the experiment are stored in the `code` folder.
Each script contains a short description of its purpose.

* `gen_sweeps.py` generates a linear and an exponential sweep, as well as their time windowed versions
* `run_experiment.py` runs the full experiment
* `segment.py` segments the recorded files into separated files corresponding
  to each original sample. It resamples the recorded speech samples at 16 kHz.
* `stitch_samples.py` resample and stitch together all the samples and creates `all_samples.wav`

##### Dependencies

[Python 3.5.3](https://www.python.org/download/releases/3.0/) was used to run the
experiment. The following three standard modules are required and can be
installed via pip.

* [SoundDevice](https://github.com/spatialaudio/python-sounddevice)
* Numpy
* Scipy

In addition, these two custom modules should be downloaded. Edit `EASYDSP_PATH`
and `TURNTABLE_PATH` in `run_experiment.py` to indicate their locations.

* [Turntable Driver](https://github.com/LCAV/PyTurntableBK9640) `Commit: 27446ae17888e902a51f4867b59bd20064135fc7`
* [Easy DSP](https://github.com/LCAV/easy-dsp) `Commit: 16b4e807a8fe1d82caa81baf6345446fd6e655cb`


#### Identify Turntable device name

    >>> import visa
    >>> rm = visa.ResourceManager()
    >>> rm.list_resources()
    ('ASRL1::INSTR', 'ASRL2::INSTR', 'ASRL3::INSTR', 'ASRL4::INSTR', 'GPIB0::12::INSTR')

#### Run the experiment

The data was collected by running

    python code/run_experiment.py -a pyramic -r 0:360:2 -o 2 -t ASRL4::INSTR 192.168.0.101

where `-o 2` indicated the number of the playback device, `-t ASRL4::INSTR` the address of the turntable,
and `192.168.0.101` the IP address of the Pyramic array.

### About Synchronization of Playback and Recordings

**There is no synchronization between playback and recording.** The reason is
that the Pyramic array used for recordings currently lacks the capability of
controlling a loudspeaker synchronously with the acquisition. As a consequence,
the absolute time of the recorded samples is unknown. This means, for example,
that the time-of-flight (TOF) cannot be obtained from the samples.

### Notes

* Bugs with the USB interface crashed the computer several times and the
  measurements were thus taken by batch. The batches are the following. 

  * Batch 1: Angles 0 to 14
  * Batch 2: 16 to 180
  * Batch 3: 182 to 190
  * Batch 4: 192 to 206 
  * Batch 5: 208 to 216
  * Batch 6: 218 to 358
  
* During quality control a glitch in recording at angle 310 with spkr 0 was
  found and the measurement was repeated on the next morning, but before the
  setup was moved. When the measurement was repeated, the temperature was 21.6C
  and the humidity 48.9%. The glitched sample is preserved in the 'discarded'
  directory."

