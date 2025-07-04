# TMS-BIDS Extension Specification

## 1. Introduction

Transcranial Magnetic Stimulation (TMS) is a non-invasive brain stimulation technique used in cognitive neuroscience, clinical research, and therapy. Despite its widespread use, TMS data lacks a standardized, FAIR-compliant structure for documentation, sharing, and multimodal integration. This specification defines a Brain Imaging Data Structure (BIDS) extension for TMS that supports structured storage of stimulation parameters, spatial coordinates, and integration with other modalities like MRI, EEG, and EMG.

## 2. BIDS Principles and Compatibility

This extension follows core BIDS principles:

- Folder and filename consistency
- Tabular data in `.tsv` with header rows
- Sidecar `.json` files for metadata
- Inheritance principle
- Modality-specific folders (`tms/`, `eeg/`, `beh/`, etc.)

## 3. Folder and File Structure

```
sub-<label>/
	└── ses-<label>/
		└── tms/
			├── sub-<label>[_ses-<label>]_task-<label>[_acq-<label>][_run-<label>]_coordsystem.json
			├── sub-<label>[_ses-<label>]_task-<label>[_acq-<label>][_run-<label>]_tms.tsv
			├── sub-<label>[_ses-<label>]_task-<label>[_acq-<label>][_run-<label>]_tms.json
			├── sub-<label>[_ses-<label>]_task-<label>[_acq-<label>][_run-<label>]_markers.tsv
			├── sub-<label>[_ses-<label>]_task-<label>[_acq-<label>][_run-<label>]_markers.json
			├── sub-<label>[_ses-<label>]_task-<label>[_acq-<label>][_run-<label>]_events.tsv
			└── sub-<label>[_ses-<label>]_task-<label>[_acq-<label>][_run-<label>]_events.json
```

## 4. Use Case Stages

### The TMS-BIDS extension supports multiple stages of TMS-based research:

- **Stage 0 — Registration**: The process of digitally registering control points for alignment with coordinate systems and anatomical scans.
- **Stage 1 — Hotspot Mapping**: EMG responses and electric field estimation.
- **Stage 2 — RMT Calibration**: Collection of Resting Motor Threshold parameters.
- **Stage 3 — Targeted Stimulation**: Experimental behavioral tasks or interventional protocols.

## 5. Detailed overview of data structure

### 5.1 `_coordsystem.json` — Coordinate Metadata

The _coordsystem.json file follows the established BIDS approach used in other modalities (e.g., anat, nirs, eeg). It adopts the common structure for defining coordinate systems, spatial units, anatomical landmarks, and fiducials. This ensures compatibility with existing tools and maximizes interoperability.
A _coordsystem.json file is used to specify the fiducials, the location of anatomical landmarks, and the coordinate system and units in which the position of landmarks or TMS stimulation targets is expressed. Fiducials are objects with a well-defined location used to facilitate the localization of sensors and co-registration. Anatomical landmarks are locations on a research subject such as the nasion (for a detailed definition see the coordinate system appendix).
The _coordsystem.json file is REQUIRED for navigated TMS stimulation datasets. If a corresponding anatomical MRI is available, the locations of anatomical landmarks in that scan should also be stored in the _T1w.json file which accompanies the TMS data.

- `ImageData`: Description of the anatomical data used for co-registration. Includes Levels: DICOM, NIFTI, MR-less.
- `IntendedFor`: Path to the anatomical file this coordinate system refers to. BIDS-style path.
- `AnatomicalLandmarkCoordinateSystem`: Defines the coordinate system for the TMS points localization. Name of the coordinate system used: Individual, MNI, Talairach, etc.
- `AnatomicalLandmarkCoordinateSystemUnits`: Units of the coordinates of Anatomical Landmark Coordinate System. Must be one of: "m", "mm", "cm", "n/a".
- `AnatomicalLandmarkCoordinateSystemDescription`: Free-form text description of the coordinate system. May also include a link to a documentation page or paper describing the system in greater detail.
- `AnatomicalLandmarkCoordinates`: Key-value pairs of the labels and 3-D digitized locations of anatomical landmarks, interpreted following the, (for example, {"NAS": [12.7,21.3,13.9], "LPA": [5.2,11.3,9.6], "RPA": [20.2,11.3,9.1]}.
- `AnatomicalLandmarkCoordinatesDescription`: [x, y, z] coordinates of anatomical landmarks. NAS - nasion, LPA - left preauricular point, RPA - right preauricular point
- `DigitizedHeadPoints`:  Relative path to the file containing the locations of digitized head points collected during the session. (for example, "sub-01_headshape.pos"). 
- `DigitizedHeadPointsNumber`: Number of digitized head points during co-regestration.
- `DigitizedHeadPointsDescription`: Free-form description of digitized points. 
- `DigitizedHeadPointsUnits`: Unit type must be one of: "m", "mm", "cm", "n/a"
- `RmsDeviation`: RMS deviation values per landmark (NAS, LPA, RPA) and overall. (for example, {"NAS": [0.7], "LPA":[1.1], "RPA":[1.2]})
- `RmsDeviationUnits`: Unit of RMS deviation values (e.g., mm)
- `RmsDeviationDescription`: Description of how RMS deviation is calculated.

* Parameters include:

The list below reflects the key parameters currently required for describing coordinate systems in TMS datasets. However, as modern studies often involve multimodal designs combining TMS with EEG or fNIRS, additional fields from other modalities may also be included where appropriate. 
These may include:
- EEGCoordinateSystem, EEGCoordinateUnits, EEGCoordinateSystemDescription
- NIRSCoordinateSystem, NIRSCoordinateUnits, NIRSCoordinateProcessingDescription, NIRSCoordinateSystemDescription
- FiducialsDescription, FiducialsCoordinates, FiducialsCoordinateUnits, FiducialsCoordinateSystem, FiducialsCoordinateSystemDescription

### Optional Headshape Files (*_headshape.<extension>)

This file is RECOMMENDED.

3D digitized head points  that describe the head shape and/or EEG electrode locations can be digitized and stored in separate files. These files are typically used to improve the accuracy of co-registration between the stimulation target, anatomical data, etc. The acq-<label> entity can be used when more than one type of digitization in done for a session, for example when the head points are in a separate file from the EEG locations.
For example:
```
sub-<label>/
   └─ ses-<label>/
		└── tms/
			├─ sub-<label>[_ses-<label>]__acq-HEAD_headshape.pos 
			└─ sub-<label>[_ses-<label>]__acq-EEG_headshape.pos  
```
These files supplement the DigitizedHeadPoints, DigitizedHeadPointsUnits, and DigitizedHeadPointsDescription fields in the corresponding _coordsystem.json file. Their inclusion is especially useful when sharing datasets intended for advanced spatial analysis or electric field modeling.

### 5.2 `_markers.tsv` — Stimulation Site Coordinates (optional sidecar `_markers.json` )

Stores stimulation target coordinates and optional coil's orientation information. Supports multiple navigation systems (e.g., Localite, Nexstim) via flexible fields. 
Required columns:

- `MarkerID`: Unique identifier for each marker. This column must appear first in the file.
- `PeelingDepth`: Depth “distance” from cortex surface to the target point OR from the entry marker to the target marker (mm).
- `target_x`: X-coordinate of the target point in millimeters.
- `target_y`: Y-coordinate of the target point in millimeters.
- `target_z`: Z-coordinate of the target point in millimeters.
- `entry_x`: X-coordinate of the entry point in millimeters.
- `entry_y`: Y-coordinate of the entry point in millimeters.
- `entry_z`: Z-coordinate of the entry point in millimeters.
- `Matrix4D`:  4x4 affine transformation matrix for the coil positioning (instrument markers of Localite systems)
- `coil_x`:  X component of coil's origin location.
- `coil_y`:  y component of coil's origin location.
- `coil_z`:  Z component of coil's origin location.
- `normal_x`:  X component of coil normal vector.
- `normal_y`:  y component of coil normal vector.
- `normal_z`:  Z component of coil normal vector.
- `direction_x`:  X component of coil direction vector.
- `direction_y`:  Y component of coil direction vector.
- `direction_z`:  Z component of coil direction vector.
- `ElectricFieldMax_x`:   X coordinate of max electric field point.
- `ElectricFieldMax_y`:   Y coordinate of max electric field point.
- `ElectricFieldMax_z`:   Z coordinate of max electric field point.
- `Timestamp`:   Timestamp of the stimulation event in ISO 8601 format.

### Field Ordering Rationale

The _markers.tsv file defines the spatial locations and orientation vectors of stimulation targets used in TMS experiments. When designing this structure, we drew partial inspiration from existing BIDS files such as *_electrodes.tsv (EEG), which capture electrode positions. However, no existing modality in BIDS explicitly supports the full specification required for navigated TMS — including stimulation coordinates, orientation vectors, and electric field estimates.
This makes _markers.tsv a novel file type, tailored to the specific needs of TMS. Fields are ordered to reflect their functional roles:
- Identification: MarkerID appears first, enabling structured referencing in the _tms.tsv file. May include not only a unique ID number but also a step count determines the stepping and number of pulses produced per mark.
- Spatial Coordinates: target_*, entry_* and PeelingDepth  describe the position of the stimulation point in the selected coordinate system. coil(x,y,z) describe the position of the TMS coil in the selected coordinate system.
- Orientation Vectors: normal_* and direction_* vectors or transformation matrix ("Matrix4D") define the coil orientation in 3D space — a critical factor in modeling TMS effects.
- Electric Field (optional): ElectricFieldMax_* defines where the electric field is maximized.

This design supports both minimal and advanced use cases: basic datasets can include just the spatial coordinates, while high-resolution multimodal studies can specify full coil orientation and field modeling parameters.

### 5.3 `_tms.tsv` — Stimulation Parameters

Stores all stimulation data. Contains one row per stimulation step. Required columns:

- `CoilDriver`: Method of coil control ("manual" , "cobot","robot", etc.).
- `CoilID`: Coil identifier (e.g. coil_1, coil_2). Should be described in Hardware part in json sidecar. 
- `StimulusMode`: Type of stimulus(e.g. single, twin, burst, etc). Depends on Stimulator options.
- `CurrentDirection`: Direction of induced current (e.g. normal, reverse).
- `Waveform`: Shape of the TMS pulse (e.g. monophasic, biphasic, etc).
- `ProtocolName`: (when using sequence stimulation)The sequence timing mode unique name. (e.g. sici,lici,custom, etc)
- `InterDoublePulseInterval`:  IDPI - The duration between the beginning of the first pulse to the beginning of the second pulse in twin or dual stimulation mode (ms).
- `InterPulseInterval`: IPI - The duration between the beginning of the first pulse to the beginning of the second pulse (Interval between pulses in train in sequence mode)(ms).
- `BurstPulsesNumber`: Number of pulses in a burst.
- `PulseRate`: Number of pulses per second (pps or Hz).
- `TrainPulses`: Number of pulses in each train.
- `RepetitionRate`: The repetition rate refers to the number of repetitions per second (pps) at which the pulses are generated (pps or Hz).
- `InterRepetitionInterval`: The duration between the beginning of the first burst to the beginning of the second burst. T=1/f (msec).
- `TrainDuration`: The duration of the train stimulation.
- `TrainNumber`: Number of trains in sequence.
- `InterTrainInterval`: The time interval between two trains is described as the time period between the last pulse in the first train to the first pulse in the next train (ms).
- `InterTrainIntervalDelay`: Additional individual time delay that can be added to the InterTrainInterval for each Train in custom protocols
- `TrainRampUp`:  Ramp-up coefficient for burst stimulation.  Ramp-up coefficient for burst stimulation. The Ramp Up function allows programming a gradual increase in amplitude with each train during burst stimulation.
- `TrainRampUpNumber`: The number of trains applied for Ramp Up function.
- `MarkerID`: Unique identifier of stimulation target point. Follow the _markers.tsv
- `StimStepCount`: Count of pulses determines the stepping and number of pulses produced per mark.
- `PulseAmplitude`: An expression for the power output level as a percentage of maximum stimulator power (% of max output).
- `DoublePulseAmplitude`: Actual for Dual_mode. The independent amplitude of the second pulse as a percentage of maximum stimulator power(% of max output).
- `PulseAmplitudeRatio`:  Actual for Twin_mode. Ratio of the amplitude of two pulses B/A.
- `PulseAmplitudeRMT`: Stimulation intensity as a percentage of the Resting Motor Threshold (% of RMT).
- `StimValidation`: Validation of the applied stimulation. Whether stimulation was validated ("yes", "no", etc).
- `CurrentGradient`: Actual measured value of the coil current gradient (A/µs).
- `ElectricFieldTarget`: Electric field at target (V/m).
- `ElectricFieldMax`: Peak electric field generated by the coil (V/m).
- `MotorResponse`: Motor response amplitude in µV.
- `Latency`: Latency of the motor response.
- `ResponseChannelName`: Channel name used for the response (motor-response) measurement.
- `ResponseChannelType`: Type of channel used for response measurement. The channel types are similar to the list of eeg modality channel type.
- `Comments`: Additional comments about the stimulation process
- `Timestamp`: Timestamp of the stimulation event in ISO 8601 format

### 5.4 Field Ordering Rationale

The order of parameters in _tms.tsv follows a hierarchical structure based on their variability during an experiment and their role in defining the stimulation process. Parameters are grouped into three logical blocks:

Block 1 — Stimulation Configuration:
This section includes core parameters that are typically set before the experiment and remain constant throughout the session:
- CoilDriver, CoilID, StimulusMode, CurrentDirection, Waveform

Block 2 — Stimulation Protocol Timing:
These parameters describe the temporal pattern and structure of stimulation protocols. They may vary between experimental sessions or protocols:
- ProtocolName, InterDoublePulseInterval, InterPulseInterval, BurstPulsesNumber, PulseRate, TrainPulses, RepetitionRate, InterRepetitionInterval, TrainDuration, TrainNumber, InterTrainInterval, InterTrainIntervalDelay, TrainRampUp, TrainRampUpNumber, CoilDriver, CoilID, StimulusMode, CurrentDirection, Waveform

Block 3 — Target-Specific Stimulation and Outcomes:
This section captures the most dynamic data — values that change from one stimulation point to another:
- MarkerID, StimStepCount, PulseAmplitude, DoublePulseAmplitude, PulseAmplitudeRatio, PulseAmplitudeRMT, StimValidation, CurrentGradient, ElectricFieldTarget, ElectricFieldMax, MotorResponse, Latency, ResponseChannelName, ResponseChannelType, Comments, Timestamp

This structure reflects the actual flow of TMS experimentation — from hardware configuration, through protocol design, to per-target application and physiological feedback. Grouping fields this way improves readability and aligns with practical data collection workflows.

### 5.5 Multimodal Integration

- TMS experiments can include:
- MRI data (for anatomical localization or field modeling)
- EEG recordings (online TMS-EEG protocols)
- EMG recordings (motor response)
- Behavioral measurements (e.g., task responses)

### 5.6 Sidecar JSON Files

Each .tsv file is accompanied by a corresponding .json sidecar that provides essential metadata. These sidecars serve both as formal definitions of the tabular fields and as containers for standardized metadata describing the context of data acquisition.

#### _tms.json Sidecar

The _tms.json file describes the contents of the associated _tms.tsv and provides additional metadata categories commonly found across BIDS modalities. It should include:

Field definitions for each column in _tms.tsv, including:
- Description
- Units (e.g., ms, %, A/µs)
- Levels (for categorical fields such as StimulusMode, CurrentDirection, etc.)

In addition, the following structured metadata blocks are RECOMMENDED:

Hardware Information: Details about the TMS stimulator, coil model, navigation system, amplifier (if applicable).
- StimulatorManufacturer, StimulatorModel, CoilModel, NavigationSystem, Manufacturer, ManufacturersModelName, DeviceSerialNumber, etc.

Task Information: Description of the cognitive/behavioral task during which stimulation is applied.
- TaskName, TaskDescription, Instructions, CogAtlasID, etc.

Institution Information: BIDS-compliant fields such as:
- InstitutionName, InstitutionAddress, InstitutionalDepartmentName

Electrical Stimulation Metadata:
- ElectricalStimulation: boolean value (true if stimulation was applied)
- ElectricalStimulationParameters: structured free-form description of stimulation protocol or parameters

These metadata fields enhance dataset interpretability, facilitate cross-site harmonization, and promote automated analysis pipelines.
Other .json sidecars (e.g., for _markers.tsv or _coordsystem.json) should follow similar BIDS conventions, focusing on field-level descriptions, spatial units, and hardware references.

### 5.7 Hardware Metadata

- Hardware section includes fields such as:
- TMS stimulator manufacturer and model
- Coil type and shape
- EMG amplifier specs

### 5.8 Coordinate Systems

The extension is agnostic to specific coordinate systems, but supports metadata indicating the system used (e.g., MNI, RAS, scanner-native).

## 6. Summary

The TMS-BIDS extension standardizes storage of TMS experiments in a multimodal, stage-aware, and FAIR-compliant way. It ensures reproducibility, hardware traceability, and compatibility with existing BIDS tools. Community feedback is welcome to refine this proposal prior to official BIDS inclusion.

