# EEG Lab c-VEP BCI Speller Demonstrator

## Overview

This project is an EEG-based **code-modulated visual evoked potential (c-VEP)** speller and keyboard demonstrator built around the **[Dareplane](https://github.com/bsdlab/Dareplane)** framework. The purpose of the system is to present a flashing on-screen keyboard, record EEG responses while the user attends to a target key, and decode the attended target using **canonical correlation analysis (CCA)** or related c-VEP decoding methods. [Dold et al., 2025, J. Neural Eng. 22 026029](https://iopscience.iop.org/article/10.1088/1741-2552/adbb20)

The demonstrator is intended as a practical implementation of a c-VEP brain-computer interface pipeline rather than a purely theoretical reproduction. It combines stimulus presentation, EEG acquisition, signal decoding, and optional output of the decoded symbol into a single modular workflow. [Thielen et al. 2015](https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0133797), [Thielen et al. 2021](https://iopscience.iop.org/article/10.1088/1741-2552/abecef)

The setup follows the general c-VEP pipeline described in the Dareplane example system and is informed by the research literature on c-VEP stimulation, Gold code-based target encoding, reconvolution-based template methods, and calibration-efficient decoding.

---

## Project Goal

The goal of this experiment is to allow a user to select keys from a visual keyboard by **looking at the desired key**, while the system decodes the corresponding EEG response. In practical terms, the experiment demonstrates the following:

- presentation of a c-VEP speller interface
- synchronization between presentation and EEG acquisition
- decoding of attended targets from EEG data
- modular orchestration of the experiment using Dareplane services
- optional forwarding of decoded output for interactive use

This project is useful both as:
- a **BCI demonstrator** for showing c-VEP target selection in action
- a **development platform** for experimenting with decoding, timing, and interface changes
- a **starting point** for future work on more robust or more user-friendly EEG keyboard systems

---

## Setup Instructions

1. Make a [conda](https://www.anaconda.com/download) environment with Python 3.10 (not higher, as PsychoPy needs 3.10) as follows:

```
conda create --name dp-cvep python=3.10.15
```

2. Activate the `dp-cvep` conda environment as follows:

```
conda activate dp-cvep
```

3. Clone this repository either by downloading it as zip and extract or use `git clone`

4. After downloading the modules using the setup script, install all the requirements of each of the downloaded Dareplane modules (control room, LSL recorder, speller, decoder). Do so by changing the directory to the module root that contains the `requirements.txt` and do the following, still from within the active `dp-cvep` environment:

```
pip install -r requirements.txt
```

5. Install the [LSL Lab Recorder](https://github.com/labstreaminglayer/App-LabRecorder). Make sure it is running on the background. [Kothe et al. 2025](https://direct.mit.edu/imag/article/doi/10.1162/IMAG.a.136/132678/The-lab-streaming-layer-for-synchronized)


## Demo using OpenBCI Headset

1. Secure the headset to user and adjust the electrodes using the screw adjustments.
2. Download the [OpenBCI GUI](https://openbci.com/downloads) and extract it under the project root.
3. Make sure the headset is switched on, the bluetooth dongle is connected and then proceed to open the GUI.
4. Select "Cyton" under data source and select "Serial (from dongle)". The headset should now be connected to the application.
5. Adjust the gain for each electrode channel by going into the hardware settings. Make sure to "Send" the settings before exiting this screen.
6. Once adjusted, switch to the "Networking" tab on the right side and set the data stream to LSL.
7. Make sure the stream is called "obci_eeg1" and the stream type is set to time series RAW.
8. Hit begin stream and monitor the data stream in real-time.

## Electrode mapping for the headset

Cz -    n1p
O2 -    n2p
Tp8 -   n3p
FPz -   n4p
O1 -    n5p
Oz -    n6p
Pz -    n7p
TP7 -   n8p

## Running the demo

1. Make sure you have the LSL Lab Recorder running.

2. Activate the `dp-cvep` conda environment as follows:

```
conda activate dp-cvep
```

3. In the control room directory, find the file `run_cvep_experiment`. In it is a Python command to start the control room. Run it from the control room root:

```
python -m control_room.main --setup_cfg_path="path/to/dp-control-room/configs/cvep_speller.toml"
```

4. Open a browser and go to `localhost:8050`. You should see the control room. If you started the EEG source (actual or simulated), you should see this at the left top.

5. Training 
   1. To start the training phase, in this order, press `TRAINING` in the dp-cvep-speller (starts the speller UI) and `RUN TRAINING` in the Macros (starts the LSL recording). 
   2. The speller waits for a keypress to continue, press key `c`.
   3. The speller runs 10 cued trials (indicated by green cues), then stops. 
   4. Press `STOP LSL RECORDING` in the macros (stops the LSL recording and saves the data). 
   5. The speller waits for a keypress to finish, press key `c`. 
   6. Note, you can press key `q` or `escape` to stop the speller at any time manually.

5. Calibration
   1. Now you have supervised training data, so we can calibrate the model. Press `FIT MODEL` in the dp-cvep-decoder (calibrated the model). It prints the performance in the log (left bottom), and shows a figure. 
   2. Close the figure to continue. 
   3. The calibrated classifier is saved to file automatically. 
 
6. Online
   1. To start the online phase, in this order, press `LOAD MODEL` in dp-cvep-decoder (loads the trained model), `CONNECT DECODER` in dp-cvep-decoder (starts the decoder), `ONLINE` in dp-cvep-speller (starts the speller UI), `DECODE ONLINE` in dp-cvep-decoder (starts decoding), `RUN ONLINE` in Macros (starts the LSL recording). 
   2. The speller waits for a keypress to continue, press key `c`.
   3. Make sure the HTML game is opened up and the focus is shifted to this tab.
   3. The speller runs 999 trials, then stops. The classifier is applied using dynamic stopping, so trials will stop as soon as possible. If a symbol is selected, it is highlighted in blue and added to the history of actions.
   4. The speller waits for a keypress to finish, press key `c`.
   5. Note, you can press `q` or `escape` to stop the speller at any time manually. 

## Troubleshooting

There are some known issues and "solutions": 
- If you do not get the control room started, try the following: 
  - Kill all Python processes (e.g., hold ctrl+c, and/or `pkill -f python`), and restart.
  - Make sure there are no other LSL streams running yet (e.g., the EEG/mockup stream). Start the control room first. Only when the control room is alive, start any other streams.
  - First start without the LSL recorder, it crashes. Then restart with the Recorder, then it works. Magic.
- If you ran `FIT MODEL` and you get the error saying 'No training files found', double-check the saved data in the `data` directory. For instance, the file should have capitals for P001 and S001, which are lowercase depending on the LSL Recorder version.
- If you just ran the speller (either `TRAINING` or `ONLINE`), and want to run it again, it might complain that it 'wants to add keys that already exist'. Somehow the speller is not closed fully the previous time, so cannot reopen. Kill all processes and restart the control room. 
- If you just ran `ONLINE` and stopped the speller in any way, the decoder will still be running. Depending on your needs, stop the decoder by pressing `STOP` in dp-cvep-decoder.
- If you run the online phase and want to record the data, pressing `RUN ONLINE` might crash the decoder stream. A workaround is to not press `RUN ONLINE`, but instead record manually via the LSL Recorder app.


## System Architecture

The experiment is organized as a set of cooperating components. Each component has a clear responsibility within the overall pipeline.

### Main Components

#### 1. Control Room
The control room coordinates the experiment. It is responsible for starting and managing the flow of the different modules, such as the speller, decoder, and recording service.

#### 2. Speller
The speller displays the c-VEP keyboard interface on screen. Each key is associated with a predefined stimulation sequence. When the user visually fixates on one key, the brain response evoked by that stimulation pattern can later be decoded.

The speller is also responsible for the timing and presentation side of the experiment, which makes it one of the most critical modules in the setup.

#### 3. Decoder
The decoder receives EEG data and applies c-VEP decoding methods to determine which target the user was attending to. In this project, the decoding logic is based primarily on **CCA / rCCA-style approaches**, following the c-VEP literature and the Dareplane example pipeline.

#### 4. LSL Recording
The recording module stores EEG and related streams through the **Lab Streaming Layer (LSL)** ecosystem. This enables offline inspection, debugging, and later analysis of the experiment data.

#### 5. EEG Acquisition Hardware
The system assumes an EEG acquisition device that provides signals suitable for c-VEP decoding. In this project context, the intended acquisition device is a **self-constructed OpenBCI headset**.

---

## High-Level Experiment Flow

At a high level, the system works as follows:

1. The experiment environment is started.
2. The speller presents a flashing keyboard to the user.
3. The user fixates on a target key.
4. EEG is acquired during the stimulation period.
5. The decoder processes the incoming EEG.
6. The system estimates which key the user attended.
7. The decoded result can be displayed, logged, or forwarded for interaction.

This modular structure is useful because it separates:
- stimulus presentation
- EEG acquisition
- signal decoding
- experimental orchestration
- recording and later inspection

That separation makes the system easier to debug and extend.

---

## Scientific Background

c-VEP spellers rely on the fact that the visual cortex responds to code-modulated visual stimulation in a way that can be distinguished across targets. Instead of assigning a simple flicker frequency to each key, c-VEP systems use carefully designed binary sequences, often derived from **Gold codes** or related code families, to modulate the visual targets. Because each target follows a distinct temporal code, the EEG responses can be matched against known templates or decoded using correlation-based methods.

This project is grounded in the following ideas from the literature:

- **code-based visual target encoding** to distinguish multiple keys
- **CCA / rCCA-style decoding** for target identification
- **template-based and reconvolution-based modeling** for c-VEP responses
- **dynamic or early stopping** concepts for more efficient target selection
- **calibration-efficient workflows**, including low-training or reduced-training variants

The system therefore reflects both the engineering side of a BCI implementation and the methodological principles described in c-VEP research.

---

## Intended Use Case

This repository is intended for users who want to:

- reproduce a working c-VEP speller demonstration
- understand how the different Dareplane components interact
- run a practical EEG-based spelling experiment
- modify the speller layout or decoding behavior
- test hardware and software timing in a modular BCI environment

It is especially relevant for:
- EEG/BCI students
- researchers building proof-of-concept systems
- developers experimenting with Dareplane-based neurotechnology workflows

---

## Hardware Requirements

The setup is designed around EEG acquisition hardware and a computer capable of running the speller and decoder modules in real time.

### Required Hardware

- EEG acquisition device compatible with the intended software workflow
- electrodes and electrode cap or mounting system
- stimulus display monitor
- computer capable of running the Dareplane modules
- reliable data connection between EEG hardware and acquisition software
- comfortable seating and stable viewing position for the user

### Project-Specific Hardware Context

This project is intended to be used with a **custom-built OpenBCI headset**. Because signal quality and electrode placement are crucial in c-VEP experiments, hardware setup quality strongly affects decoding performance.

### Recommended Practical Conditions

For best results:

- use a stable monitor with consistent refresh behavior
- reduce visual distractions in the room
- ensure good electrode contact quality
- keep cables and acquisition hardware stable during runs
- minimize head and body movement during trials

---

## Software Requirements

All required packages can be installed by following the instructions provided above.

### Core Software Stack

- Python-based environment for running the project modules
- Dareplane framework and related services
- PsychoPy-based or equivalent visual presentation backend for the speller
- LSL-compatible recording setup
- required dependencies for EEG streaming, decoding, and control

### Main Project Modules

The system is built around these core modules:

- `dp-control-room`
- `dp-cvep-speller`
- `dp-cvep-decoder`
- `dp-lsl-recording`

## Repository Purpose and Organization

This repository contains the material needed to run and develop the c-VEP speller demonstrator.

A typical repository structure for this project will include some or all of the following:

- experiment source code
- speller logic and configuration
- decoder logic and configuration
- control-room integration
- recording utilities
- configuration files
- documentation
- optional scripts for testing and debugging

> Replace this section with the exact folder structure of your repository if you want the README to reflect the current codebase precisely.

### Suggested Repository Structure Section

```text
project-root/
├── data/
├── dp-control-room/
├── dp-cvep-decoder/
├── dp-cvep-speller/
├── dp-lsl-recording/
├── LabRecorder/
├── openbcigui/
├── game.html
└── README.md
