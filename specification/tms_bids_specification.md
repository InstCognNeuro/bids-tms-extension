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

### 5.1 `*_coordsystem.json` — Coordinate Metadata

The _coordsystem.json file follows the established BIDS approach used in other modalities (e.g., anat, nirs, eeg). It adopts the common structure for defining coordinate systems, spatial units, anatomical landmarks, and fiducials. This ensures compatibility with existing tools and maximizes interoperability.
A _coordsystem.json file is used to specify the fiducials, the location of anatomical landmarks, and the coordinate system and units in which the position of landmarks or TMS stimulation targets is expressed. Fiducials are objects with a well-defined location used to facilitate the localization of sensors and co-registration. Anatomical landmarks are locations on a research subject such as the nasion (for a detailed definition see the coordinate system appendix).
The _coordsystem.json file is REQUIRED for navigated TMS stimulation datasets. If a corresponding anatomical MRI is available, the locations of anatomical landmarks in that scan should also be stored in the _T1w.json file which accompanies the TMS data.

```
| Field                                           | Type    | Description                                                                                                                                                                                                                                                                     | Units / Levels                      |
| ----------------------------------------------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------- |
| `ImageData`                                     | string  | Description of the anatomical data used for co-registration. Includes Levels: DICOM, NIFTI, MR-less.                                                                                                                                                                            | Levels: `DICOM`, `NIFTI`, `MR-less` |
| `IntendedFor`                                   | string  | Path to the anatomical file this coordinate system refers to. BIDS-style path. (example: `bids::sub-01/ses-01/anat/sub-01_T1w.nii.gz`)                                                                                                                                          | BIDS path                           |
| `AnatomicalLandmarkCoordinateSystem`            | string  | Defines the coordinate system for the anatomical landmarks. See the Coordinate Systems Appendix for a list of restricted keywords for coordinate systems. If "Other", provide definition of the coordinate system in `AnatomicalLandmarkCoordinateSystemDescription`.           | —                                   |
| `AnatomicalLandmarkCoordinateSystemUnits`       | string  | Units of the coordinates of Anatomical Landmark Coordinate System. Must be one of: `"m"`, `"mm"`, `"cm"`, `"n/a"`.                                                                                                                                                              | `"m"`, `"mm"`, `"cm"`, `"n/a"`      |
| `AnatomicalLandmarkCoordinateSystemDescription` | string  | Free-form text description of the coordinate system. May also include a link to a documentation page or paper describing the system in greater detail.                                                                                                                          | Free text                           |
| `AnatomicalLandmarkCoordinates`                 | object  | Key-value pairs of the labels and 3-D digitized locations of anatomical landmarks, interpreted following the `AnatomicalLandmarkCoordinateSystem`. Each array MUST contain three numeric values corresponding to x, y, and z axis of the coordinate system in that exact order. | 3D coordinates                      |
| `AnatomicalLandmarkCoordinatesDescription`      | string  | `[x, y, z]` coordinates of anatomical landmarks. NAS — nasion, LPA — left preauricular point, RPA — right preauricular point                                                                                                                                                    | —                                   |
| `DigitizedHeadPoints`                           | string  | Relative path to the file containing the locations of digitized head points collected during the session. (for example, `"sub-01_headshape.pos"`)                                                                                                                               | File path or `"n/a"`                |
| `DigitizedHeadPointsNumber`                     | integer | Number of digitized head points during co-registration.                                                                                                                                                                                                                         | count                               |
| `DigitizedHeadPointsDescription`                | string  | Free-form description of digitized points.                                                                                                                                                                                                                                      | —                                   |
| `DigitizedHeadPointsUnits`                      | string  | Unit type. Must be one of: `"m"`, `"mm"`, `"cm"`, `"n/a"`.                                                                                                                                                                                                                      | `"m"`, `"mm"`, `"cm"`, `"n/a"`      |
| `RmsDeviation`                                  | object  | `{"RMS":[],"NAS":[],"LPA":[],"RPA":[]}` — deviation values per landmark                                                                                                                                                                                                         | values per marker                   |
| `RmsDeviationUnits`                             | string  | Unit of RMS deviation values.                                                                                                                                                                                                                                                   | `"m"`, `"mm"`, `"cm"`, `"n/a"`      |
| `RmsDeviationDescription`                       | string  | Description of how RMS deviation is calculated and for which markers.                                                                                                                                                                                                           | —                                   |
```
Example *_coordsystem.json:
```
{
  "ImageData": "NIFTI",
  "IntendedFor": "bids::sub-01/ses-01/anat/sub-01_T1w.nii.gz",
  "AnatomicalLandmarkCoordinateSystem": "Individual",
  "AnatomicalLandmarkCoordinateSystemUnits": "mm",
  "AnatomicalLandmarkCoordinateSystemDescription": "RAS orientation: origin halfway between LPA and RPA; x-axis points to RPA, y-axis orthogonal through NAS, z-axis orthogonal to xy-plane.",
  "AnatomicalLandmarkCoordinates": {
    "NAS": [12.7, 21.3, 13.9],
    "LPA": [5.2, 11.3, 9.6],
    "RPA": [20.2, 11.3, 9.1]
  },
  "AnatomicalLandmarkCoordinatesDescription": "[x, y, z] coordinates of anatomical landmarks: NAS (nasion), LPA (left preauricular), RPA (right preauricular)",
  "DigitizedHeadPoints": "sub-01_acq-HEAD_headshape.pos",
  "DigitizedHeadPointsNumber": 1200,
  "DigitizedHeadPointsDescription": "Digitized head points collected during subject registration",
  "DigitizedHeadPointsUnits": "mm",
  "RmsDeviation": {
    "NAS": [0.7],
    "LPA": [0.3],
    "RPA": [0.4],
    "RMS": [0.5]
  },
  "RmsDeviationUnits": "mm",
  "RmsDeviationDescription": "Root Mean Square deviation for fiducial points"
}
```
** Parameters include **:

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

### 5.2 `*_markers.tsv` — Stimulation Site Coordinates (optional sidecar `_markers.json` )

Stores stimulation target coordinates and optional coil's orientation information. Supports multiple navigation systems (e.g., Localite, Nexstim) via flexible fields. 
Required columns:

```
| Field                | Type   | Description                                                                                             | Units    |
| -------------------- | ------ | ------------------------------------------------------------------------------------------------------- | -------- |
| `MarkerID`           | string | Unique identifier for each marker. This column must appear first in the file.                           | —        |
| `PeelingDepth`       | number | Depth “distance” from cortex surface to the target point OR from the entry marker to the target marker. | `mm`     |
| `target_x`           | number | X-coordinate of the target point in millimeters.                                                        | `mm`     |
| `target_y`           | number | Y-coordinate of the target point in millimeters.                                                        | `mm`     |
| `target_z`           | number | Z-coordinate of the target point in millimeters.                                                        | `mm`     |
| `entry_x`            | number | X-coordinate of the entry point in millimeters.                                                         | `mm`     |
| `entry_y`            | number | Y-coordinate of the entry point in millimeters.                                                         | `mm`     |
| `entry_z`            | number | Z-coordinate of the entry point in millimeters.                                                         | `mm`     |
| `Matrix_4x4`         | array  | 4x4 affine transformation matrix for the coil positioning (instrument markers of Localite systems).   | —        |
| `coil_x`             | number | X component of coil's origin location.                                                                  | `mm`     |
| `coil_y`             | number | Y component of coil's origin location.                                                                  | `mm`     |
| `coil_z`             | number | Z component of coil's origin location.                                                                  | `mm`     |
| `normal_x`           | number | X component of coil normal vector.                                                                      | `mm`     |
| `normal_y`           | number | Y component of coil normal vector.                                                                      | `mm`     |
| `normal_z`           | number | Z component of coil normal vector.                                                                      | `mm`     |
| `direction_x`        | number | X component of coil direction vector.                                                                   | `mm`     |
| `direction_y`        | number | Y component of coil direction vector.                                                                   | `mm`     |
| `direction_z`        | number | Z component of coil direction vector.                                                                   | `mm`     |
| `ElectricFieldMax_x` | number | X coordinate of max electric field point.                                                               | `mm`     |
| `ElectricFieldMax_y` | number | Y coordinate of max electric field point.                                                               | `mm`     |
| `ElectricFieldMax_z` | number | Z coordinate of max electric field point.                                                               | `mm`     |
| `Timestamp`          | string | Timestamp of the stimulation event in ISO 8601 format.                                                  | ISO 8601 |
```

### Example _markers.tsv:

```
MarkerID	PeelingDepth	target_x	target_y	target_z	entry_x	entry_y	entry_z	Matrix_4x4	coil_x	coil_y	coil_z	normal_x	normal_y	normal_z	direction_x	direction_y	direction_z	ElectricFieldMax_x	ElectricFieldMax_y	ElectricFieldMax_z	Timestamp
M01	        15.3	        12.1	    25.4	    43.2	    11.9	    24.8	    42.8	    n/a	        11.7	    24.6	    42.5	    0.0	        0.0	        1.0	        1.0	        0.0	        0.0	            13.5	            26.2	            44.1	        2025-06-01T13:45:20.123Z
M02	        14.7	        18.0	    27.9	    41.0	    17.8	    27.5	    40.5	    [[1,0,0,0],[0,1,0,0],[0,0,1,0],[0,0,0,1]]	n/a	        n/a	        n/a	        n/a	        n/a	        n/a	        n/a	        n/a	        n/a	            19.3	            29.1	            42.2	        2025-06-01T13:45:25.789Z

```

### Field Ordering Rationale

The *_markers.tsv file defines the spatial locations and orientation vectors of stimulation targets used in TMS experiments. When designing this structure, we drew partial inspiration from existing BIDS files such as *_electrodes.tsv (EEG), which capture electrode positions. However, no existing modality in BIDS explicitly supports the full specification required for navigated TMS — including stimulation coordinates, orientation vectors, and electric field estimates.
This makes _markers.tsv a novel file type, tailored to the specific needs of TMS. Fields are ordered to reflect their functional roles:
- Identification: MarkerID appears first, enabling structured referencing in the _tms.tsv file. May include not only a unique ID number but also a step count determines the stepping and number of pulses produced per mark.
- Spatial Coordinates: target_*, entry_* and PeelingDepth  describe the position of the stimulation point in the selected coordinate system. coil(x,y,z) describe the position of the TMS coil in the selected coordinate system.
- Orientation Vectors: normal_* and direction_* vectors or transformation matrix ("Matrix4D") define the coil orientation in 3D space — a critical factor in modeling TMS effects.
- Electric Field (optional): ElectricFieldMax_* defines where the electric field is maximized.

This design supports both minimal and advanced use cases: basic datasets can include just the spatial coordinates, while high-resolution multimodal studies can specify full coil orientation and field modeling parameters.

### 5.3 `*_tms.json` — Sidecar JSON 

The _tms.json file is a required sidecar accompanying the _tms.tsv file. It serves to describe the columns in the tabular file, define units and levels for categorical variables, and—crucially—provide structured metadata about the stimulation device, task, and context of the experiment.

Like other BIDS modalities, this JSON file includes:

**Task information:**

- TaskName, TaskDescription, Instructions

**Device metadata:**

- Manufacturer, ManufacturersModelName, SoftwareVersion, DeviceSerialNumber, TrackingSystemName

**Institutional context:**

- InstitutionName, InstitutionAddress, InstitutionalDepartmentName

**Stimulation metadata:**

- ElectricalStimulation: whether TMS was applied (true/false)

- ElectricalStimulationParameters: optional free-text summary of protocol

Additionally, the _tms.json file introduces a dedicated hardware block called CoilSet, which captures detailed physical and electromagnetic parameters of one or more stimulation coils used in the session. This structure allows precise modeling, reproducibility, and harmonization of coil-related effects across studies.

Each entry in "CoilSet" is an object with the following fields:

```
Field									Type	Description
CoilID									string	Unique identifier for the coil, used to reference this entry from _tms.tsv.
CoilType								string	Model/type of the coil (e.g., CB60, Cool-B65).
CoilShape								string	Geometric shape of the coil windings (e.g., figure-of-eight, circular).
CoilCooling								string	Cooling method (air, liquid, passive).
CoilDiameter.Value						number	Diameter of the outer winding (usually in mm).
CoilDiameter.Units						string	Units for the diameter (e.g., mm).
MagneticFieldPeak.Value					number	Peak magnetic field at the surface of the coil (in Tesla).
MagneticFieldPeak.Units					string	Units for magnetic field peak (Tesla).
MagneticFieldPenetrationDepth.Value		number	Penetration depth of the magnetic field at a reference intensity level (e.g., 70 V/m).
MagneticFieldGradient.Value				number	Gradient of the magnetic field at a specific depth (typically in kT/s).
```

Example:
```
"CoilSet": [
  {
    "CoilID": "1",
    "CoilType": "CB60",
    "CoilShape": "figure-of-eight",
    "CoilCooling": "air",
    "CoilDiameter": {
      "Value": 75,
      "Units": "mm",
      "Description": "Outer winding diameter"
    },
    "MagneticFieldPeak": {
      "Value": 1.9,
      "Units": "Tesla",
      "Description": "Peak magnetic field"
    },
    "MagneticFieldPenetrationDepth": {
      "Value": 18,
      "Units": "mm",
      "Description": "Depth at which field reaches 70 V/m"
    },
    "MagneticFieldGradient": {
      "Value": 160,
      "Units": "kT/s",
      "Description": "Gradient 20 mm below coil center"
    }
  }
]
```

The _tms.json follows standard BIDS JSON conventions and is essential for validator support, automated parsing, and multimodal integration (e.g., aligning stimulation parameters with EEG or MRI metadata).


### 5.4 `*_tms.tsv` — Stimulation Parameters

Stores all stimulation data. Contains one row per stimulation step. Required columns:

```
| Field                        | Type    | Description                                                                                    | Units / Levels                              |
| ---------------------------- | ------- | ---------------------------------------------------------------------------------------------- | ------------------------------------------- |
| `CoilDriver`                 | string  | Control method for the coil. (manual, fixed, cobot, robot)                                     | Levels: manual, fixed, cobot, robot         |
| `CoilID`                     | string  | Coil identifier (e.g. coil\_1, coil\_2). Should be described in Hardware part in json sidecar. | —                                           |
| `StimulusMode`               | string  | Type of stimulus (e.g. single, twin, burst, etc). Depends on Stimulator options.               | Levels: single, power, twin, dual, sequence |
| `CurrentDirection`           | string  | Direction of induced current (e.g. normal, reverse).                                           | Levels: normal, reverse                     |
| `Waveform`                   | string  | Shape of the TMS pulse (e.g. monophasic, biphasic, etc).                                       | Levels: monophasic, biphasic, halfsine      |
| `ProtocolName`               | string  | The sequence timing mode unique name (e.g. sici, lici, custom, etc).                           | —                                           |
| `InterDoublePulseInterval`   | number  | Time from start of first to start of second pulse (twin or dual).                              | msec                                        |
| `InterPulseInterval`         | number  | Interval between pulses within a train.                                                        | msec                                        |
| `BurstPulsesNumber`          | integer | Number of pulses in a burst.                                                                   | —                                           |
| `PulseRate`                  | number  | Number of pulses per second.                                                                   | pps                                         |
| `TrainPulses`                | integer | Number of pulses in each train.                                                                | —                                           |
| `RepetitionRate`             | number  | Frequency of trains.                                                                           | pps                                         |
| `InterRepetitionInterval`    | number  | Time between start of burst N and N+1.                                                         | msec                                        |
| `TrainDuration`              | number  | Duration of the full train.                                                                    | msec                                        |
| `TrainNumber`                | integer | Number of trains in sequence.                                                                  | —                                           |
| `InterTrainInterval`         | number  | Time from last pulse of one train to first of next.                                            | msec                                        |
| `InterTrainIntervalDelay`    | number  | Optional per-train delay override.                                                             | msec                                        |
| `TrainRampUp`                | number  | Gradual amplitude increase per train.                                                          | —                                           |
| `TrainRampUpNumber`          | integer | Number of trains for ramp-up.                                                                  | —                                           |
| `MarkerID`                   | string  | Identifier of stimulation target.                                                              | —                                           |
| `StimStepCount`              | integer | Number of pulses applied at the marker.                                                        | —                                           |
| `FirstPulseAmplitude`        | number  | Intensity of the first or single pulse (% of max stimulator output).                           | %                                           |
| `SecondPulseAmplitude`       | number  | Intensity of the second pulse (dual mode).                                                     | %                                           |
| `PulseAmplitudeRatio`        | number  | Amplitude ratio of two pulses (B/A).                                                           | —                                           |
| `FirstPulseAmplitudeRMT`     | number  | Intensity of first/single pulse as % of RMT.                                                   | %                                           |
| `SecondPulseAmplitudeRMT`    | number  | Intensity of second pulse as % of RMT.                                                         | %                                           |
| `StimValidation`             | string  | Was the stimulation verified / observed.                                                       | yes / no / free-text                        |
| `CurrentGradient`            | number  | Measured gradient of coil current.                                                             | A/µs                                        |
| `ElectricFieldTarget`        | number  | Electric field at stimulation target.                                                          | V/m                                         |
| `ElectricFieldMax`           | number  | Peak electric field at any location.                                                           | V/m                                         |
| `MotorResponse`              | number  | Motor-evoked potential (MEP) amplitude.                                                        | µV                                          |
| `Latency`                    | number  | Delay between stimulation and response.                                                        | ms                                          |
| `ResponseChannelName`        | string  | Name of recorded EMG/EEG/MEG channel.                                                          | —                                           |
| `ResponseChannelType`        | string  | Type of channel (e.g. emg, eeg).                                                               | —                                           |
| `ResponseChannelDescription` | string  | Description of the response channel.                                                           | —                                           |
| `ResponseChannelReference`   | string  | Reference channel name if applicable.                                                          | —                                           |
| `Status`                     | string  | Data quality observed on the channel.                                                          | —                                           |
| `StatusDescription`          | string  | Freeform text description of noise or artifact affecting data quality on the channel.          | —                                           |
| `Timestamp`                  | string  | Timestamp in ISO 8601 format.                                                                  | ISO 8601                                    |

```
### Example *_tms.tsv:

```
CoilDriver	CoilID	StimulusMode	CurrentDirection	Waveform	ProtocolName	InterDoublePulseInterval	InterPulseInterval	BurstPulsesNumber	PulseRate	TrainPulses	RepetitionRate	InterRepetitionInterval	TrainDuration	TrainNumber	InterTrainInterval	InterTrainIntervalDelay	TrainRampUp	TrainRampUpNumber	MarkerID	StimStepCount	FirstPulseAmplitude	SecondPulseAmplitude	PulseAmplitudeRatio	FirstPulseAmplitudeRMT	SecondPulseAmplitudeRMT	StimValidation	CurrentGradient	ElectricFieldTarget	ElectricFieldMax	MotorResponse	Latency	ResponseChannelName	ResponseChannelType	ResponseChannelDescription	ResponseChannelReference	Status	StatusDescription	Timestamp
manual	coil_1	twin	normal	biphasic	sici	2.5	n/a	n/a	n/a	n/a	n/a	n/a	n/a	n/a	n/a	n/a	n/a	n/a	marker1	1	70	110	1.57	80	110	validated	45.2	92.3	118.5	420	24.0	APB	emg	Flexor carpi	linked-ears	good	n/a	2025-06-01T13:45:10.456Z
fixed	coil_2	single	reverse	monophasic	custom	n/a	n/a	n/a	n/a	n/a	n/a	n/a	n/a	n/a	n/a	n/a	n/a	marker2	1	75	n/a	n/a	85	n/a	validated	40.7	85.1	112.2	n/a	n/a	n/a	n/a	n/a	n/a	bad	high muscle tension 2025-06-01T13:46:12.789Z
```

### Examples *_tms.json:

```
{
  "TaskName": "HotSpot",
  "Instructions": "Focus on the fixation cross. Try to stay relaxed.",
  "TaskDescription": "Identifying optimal stimulation site for right-hand motor cortex (APB).",
  "SoftwareVersion": "3.2.1",
  "Manufacturer": "MagVenture or NexStim",
  "ManufacturersModelName": "MagProX100 MAGOPTION or MagProR30 MAGOPTION or NexStim",
  "TrackingSystemName": "Name of the navigation or tracking system used during TMS to localize stimulation targets and record coil positions. Examples include Localite TMS Navigator, Nexstim eXimia or Brainsight",
  
  "CoilSet": [
  {
    "CoilID": "1",
    "CoilType": "CB60",
    "CoilShape": "figure-of-eight",
	"CoilCooling": "air",
    "CoilDiameter": {
      "Value": 75,
      "Units": "mm",
	  "Description": "Diameter of the outer winding of the coil."
    },
    "MagneticFieldPeak": {
        "Value": 1.9,
        "Units": "Tesla",
		"Description": "Peak magnetic field at the surface of the coil."
    },
	"MagneticFieldPenetrationDepth": {
		"Value": 18,
		"Units": "mm",
		"Description": "Depth at which the electric field reaches 70 V/m under the center of the coil."
    },
    "MagneticFieldGradient": {
        "Value": 160,
        "Units": "kT/s",
        "Description": "Magnetic field gradient at 20 mm below the coil center."
    }
    },
	{
	"CoilID": "2",
    "CoilType": "CB60",
    "CoilShape": "figure-of-eight",
	"CoilCooling": "liquid",
    "CoilDiameter": {
      "Value": 75,
      "Units": "mm",
	  "Description": "Diameter of the outer winding of the coil."
    },
    "MagneticFieldPeak": {
        "Value": 1.9,
        "Units": "Tesla",
		"Description": "Peak magnetic field at the surface of the coil."
    },
	"MagneticFieldPenetrationDepth": {
		"Value": 18,
		"Units": "mm",
		"Description": "Depth at which the electric field reaches 70 V/m under the center of the coil."
    },
    "MagneticFieldGradient": {
        "Value": 160,
        "Units": "kT/s",
        "Description": "Magnetic field gradient at 20 mm below the coil center."
    }	
	}],
  "CoilDriver": {
    "Description": "Control method for the coil.",
    "Levels": {
      "manual": "Hand-held manually by operator",
      "fixed": "Mounted and fixed in position",
      "cobot": "Controlled via collaborative robot arm",
      "robot": "Controlled via industrial robotic manipulator"
    }
  },
  "StimulusMode": {
    "Description": "Type of stimulus pulse applied.",
    "Levels": {
      "single": "Single pulse",
      "twin": "Two pulses close in time",
      "dual": "Two independent pulses",
      "power": "High-intensity mode",
      "burst": "Rapid-fire burst of pulses",
	  "repetitive": "Repetitive stimulation",
      "sequence": "Custom pulse sequence"
    }
  },
  "CurrentDirection": {
    "Description": "Direction of induced current relative to cortex.",
    "Levels": {
      "normal": "Anterior-posterior",
      "reverse": "Posterior-anterior"
    }
  },
  "FirstPulseAmplitude": {
    "Description": "Power output level of first or single pulse.",
    "Units": "%"
  },
  "FirstPulseAmplitudeRMT": {
    "Description": "Power output level of first or single pulse in % of RMT.",
    "Units": "%"
  },
  "ElectricFieldTarget": {
    "Description": "Electric field value at the stimulation target.",
    "Units": "V/m"
  }
}
```

### 5.5 Field Ordering Rationale

The order of parameters in _tms.tsv follows a hierarchical structure based on their variability during an experiment and their role in defining the stimulation process. Parameters are grouped into three logical blocks:

Block 1 — Stimulation Configuration:
This section includes core parameters that are typically set before the experiment and remain constant throughout the session:
- CoilDriver, CoilID, StimulusMode, CurrentDirection, Waveform

Block 2 — Stimulation Protocol Timing:
These parameters describe the temporal pattern and structure of stimulation protocols. They may vary between experimental sessions or protocols:
- ProtocolName, InterDoublePulseInterval, InterPulseInterval, BurstPulsesNumber, PulseRate, TrainPulses, RepetitionRate, InterRepetitionInterval, TrainDuration, TrainNumber, InterTrainInterval, InterTrainIntervalDelay, TrainRampUp, TrainRampUpNumber, CoilDriver, CoilID, StimulusMode, CurrentDirection, Waveform

Block 3 — Target-Specific Stimulation and Outcomes:
This section captures the most dynamic data — values that change from one stimulation point to another:
- MarkerID, StimStepCount, FirstPulseAmplitude, SecondPulseAmplitude, PulseAmplitudeRatio, FirstPulseAmplitudeRMT, SecondPulseAmplitudeRMT, StimValidation, CurrentGradient, ElectricFieldTarget, ElectricFieldMax, MotorResponse, Latency, ResponseChannelName, ResponseChannelType, Comments, Timestamp

This structure reflects the actual flow of TMS experimentation — from hardware configuration, through protocol design, to per-target application and physiological feedback. Grouping fields this way improves readability and aligns with practical data collection workflows.

### 5.6 Multimodal Integration

- TMS experiments can include:
- MRI data (for anatomical localization or field modeling)
- EEG recordings (online TMS-EEG protocols)
- EMG recordings (motor response)
- Behavioral measurements (e.g., task responses)

### 5.7 Sidecar JSON Files

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

### 5.8 Hardware Metadata

- Hardware section includes fields such as:
- TMS stimulator manufacturer and model
- Coil type and shape
- EMG amplifier specs

### 5.9 Coordinate Systems

The extension is agnostic to specific coordinate systems, but supports metadata indicating the system used (e.g., MNI, RAS, scanner-native).

## 6. Summary

The TMS-BIDS extension standardizes storage of TMS experiments in a multimodal, stage-aware, and FAIR-compliant way. It ensures reproducibility, hardware traceability, and compatibility with existing BIDS tools. Community feedback is welcome to refine this proposal prior to official BIDS inclusion.

