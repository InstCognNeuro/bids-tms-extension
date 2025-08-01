# BEPXXX: TMS - Transcranial Magnetic Stimulation

**Authors**:

- **Oleg Shevtsov**, ORCID: [0009-0008-3460-0633](https://orcid.org/0009-0008-3460-0633)
- **Milana Makarova**, ORCID: [0000-0002-9351-6588](https://orcid.org/0000-0002-9351-6588)
- **Matteo Feurra**, ORCID: [0000-0003-0934-6764](https://orcid.org/0000-0003-0934-6764)

**Corresponding author**: Oleg Shevtsov [olegshevts@gmail.com]

**Submitted to**: BIDS Extension Proposals
**Status**: Draft
**GitHub**: 

## Introduction

Transcranial Magnetic Stimulation (TMS) is a non-invasive method for stimulating the human brain. TMS is widely used in neuroscience research to study cortical excitability, brain-behavior relationships, and to develop novel therapeutic interventions. Despite its growing adoption, no standardized data structure has been proposed for organizing and sharing TMS data in a FAIR-compliant way. We introduce an extension to the Brain Imaging Data Structure (BIDS) to support TMS data, enabling consistent documentation, data sharing, and multimodal integration with existing BIDS modalities such as MRI, EEG, EMG, and behavioral data.

## BIDS Common Principles

This extension adheres to core BIDS principles:

- Folder and file naming consistency
- Sidecar `.json` files for metadata
- `.tsv` files for tabular data with header rows
- Inheritance principle
- Modality-specific folders (`tms/`, `eeg/`, `beh/`, etc.)

## Folder Structure

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

## TMS-Specific File Naming Conventions

- `*_tms.tsv`: Main TMS stimulation parameters
- `*_markers.tsv`: Stimulation site coordinates
- `*_coordsystem.json`: Description of the coordinate space used in marker localization

## Use Case Stages

The TMS-BIDS extension supports multiple stages of TMS-based research:

- **Stage 0 – Registration**: The process of digitally registering control points for alignment with coordinate systems and applying synchronization to structural MRI
- **Stage 1 – Hotspot Mapping**: Extended tabular files store stimulation location, EMG response, electric field estimates, and device settings.
- **Stage 2 – RMT and Threshold Calibration**: Collecting data for Resting Motor Threshold determination. Only core stimulation parameters are recorded.
- **Stage 3 – Targeted Stimulation**: Used in intervention or behavioral tasks, includes coordinate files to support navigated stimulation and data integration.

## Multimodal Integration

TMS experiments can include:

- MRI data (for anatomical localization or field modeling)
- EEG recordings (online TMS-EEG protocols)
- EMG recordings (motor response)
- Behavioral measurements (e.g., task responses)

## JSON Sidecars

Each `.tsv` is accompanied by a sidecar `.json` file with:

- Field descriptions
- Units
- Levels (if categorical)
- Optional hardware specifications

## Hardware Metadata

Includes fields such as:

- TMS stimulator manufacturer and model
- Coil type and shape
- EMG amplifier specs

## Coordinate Systems

The extension is agnostic to specific coordinate systems, but supports metadata indicating the system used (e.g., MNI, RAS, scanner-native).

## Rationale

TMS (Transcranial Magnetic Stimulation) is increasingly used in cognitive neuroscience, psychiatry, and clinical neuromodulation. However, BIDS currently lacks any standardized way to represent TMS stimulation protocols, spatial coordinates, or associated EMG/EEG/behavioral outcomes. Existing BIDS modalities (EEG, MEG, MRI) do not support stimulation-specific metadata such as coil orientation, pulse parameters, electric field estimates, or navigated targeting. This extension fills that gap by introducing a dedicated tms/ modality folder, new TSV/JSON file pairs for stimulation events, and structured metadata aligned with FAIR principles. It ensures reproducibility and compatibility across experimental stages (hotspot mapping, thresholding, targeted stimulation), hardware platforms (MagVenture, Nexstim), and multimodal pipelines (MRI, EEG, EMG).

## Use Cases

1. Hotspot Localization in Motor Cortex
A researcher uses navigated TMS to identify the hotspot for motor evoked potentials (MEPs) in the hand area of the primary motor cortex. The markers.tsv stores the 3D coordinates of the coil and target location. The tms.tsv stores stimulation parameters and the EMG response for each pulse. The dataset can be shared with others or reused for retrospective analysis of electric field modeling.

2. Resting Motor Threshold (RMT) Calibration Study
A lab conducts an RMT calibration session using manual stimulation without navigation. Only core pulse parameters are recorded. The tms.tsv allows later analysis and statistical modeling of threshold data across participants.

3. Closed-Loop TMS-EEG Experiment
During an online TMS-EEG protocol, pulses are triggered in response to EEG features. Stimulation timing, coil parameters, and response metadata are captured in tms.tsv, and EEG data is saved in eeg/. The coordinate metadata enables field modeling in headspace aligned with the subject’s MRI.

4. Preoperative Motor or Language Mapping
In clinical contexts such as neurosurgery planning, navigated TMS is used to map eloquent cortical areas. The markers.tsv provides spatial reference to motor or language-related sites, which can be co-registered with anatomical MRI and surgical planning tools. This ensures maximal tissue preservation during tumor resection or epilepsy surgery.

5. Behavioral Neuroeconomics Experiment with Online TMS
A group conducts a decision-making task while applying online TMS to dorsolateral prefrontal cortex during critical stages of choice evaluation. The tms.tsv file logs precise stimulation timing aligned with trial-level behavioral markers saved in beh/. This enables fine-grained analysis of causal interference with valuation or control processes.

6. Concurrent TMS-fNIRS Study
In a simultaneous TMS–functional near-infrared spectroscopy (fNIRS) experiment, the researchers deliver bursts of TMS while recording cortical hemodynamic responses from prefrontal areas. The tms.tsv contains burst timing and parameters; coordsystem.json specifies scalp localization in shared space. This facilitates multimodal coregistration, allowing joint modeling of induced electric fields and vascular responses.

## Interoperability

This extension is designed to be interoperable with existing BIDS modalities:

- EEG/MEG: The ResponseChannelType and ResponseChannelName fields in tms.tsv align with EEG channel naming and types, enabling integration with simultaneous TMS-EEG recordings.

- MRI: The coordsystem.json allows alignment of TMS targets with anatomical scans. The IntendedFor key can point to T1-weighted images, enabling accurate co-registration and electric field modeling.

- EMG: Stimulation outcomes such as MEP amplitudes and latencies can be linked to EMG recordings, either via BIDS eeg/ folder or as side-loaded physiological data.

- Behavioral data: Synchronization with behavioral events stored in beh/ allows precise timing analysis of stimulation effects.

- fNIRS: Coordinates and stimulation timing can be aligned with fNIRS channels using shared coordinate frameworks.

- Events files: Stimulation timing and experimental context can also be synchronized using standard *_events.tsv files, enabling joint analysis with behavioral responses or external triggers.


The proposed file formats (_tms.tsv, _markers.tsv, _coordsystem.json) reuse conventions already present in EEG and NIRS (*_electrodes.tsv, *_optodes.tsv, coordsystem.json) and anatomical modalities, minimizing the learning curve for tool developers and researchers.

The coordsystem.json file supports extensions from other modalities. As modern studies often involve multimodal designs combining TMS with EEG or fNIRS, additional fields from those domains may also be included where appropriate. These may include:

- EEGCoordinateSystem, EEGCoordinateUnits, EEGCoordinateSystemDescription

- NIRSCoordinateSystem, NIRSCoordinateUnits, NIRSCoordinateProcessingDescription, NIRSCoordinateSystemDescription

- FiducialsDescription, FiducialsCoordinates, FiducialsCoordinateUnits, FiducialsCoordinateSystem, FiducialsCoordinateSystemDescription

- Additionally, the structure can also include *_headshape.pos files that contain 3D digitized head points. These files improve spatial alignment between stimulation targets, anatomical scans, and sensor positions.

## Validation Strategy

We plan to support validation of TMS-BIDS datasets through the official bids-validator by developing a dedicated plugin or extending existing schemas. The validation strategy includes:

- File presence and structure: Ensure required TMS files (*_tms.tsv, *_tms.json, *_markers.tsv, *_coordsystem.json) are present and named according to BIDS conventions.

- Column checks: Verify required columns exist in .tsv files, match expected names, and follow type constraints (e.g., numeric, categorical, ISO 8601 timestamps).

- Reference consistency: Cross-validate that MarkerID values used in *_tms.tsv exist in the corresponding *_markers.tsv.

- Units and levels: Check that units (e.g., "ms", "V/m", "%") and levels (for categorical fields like StimulusMode or CoilDriver) are properly defined in .json sidecars.

- Multimodal compatibility: Ensure that IntendedFor fields in coordsystem.json correctly reference anatomical scans, and that time alignment across modalities (e.g., events, behavioral data) is maintained.

- Headshape files: When present, verify naming and placement of *_headshape.pos files.

In the early phase, validation will also be supported via curated example datasets and integration tests contributed by the community. Strict validation will be optional to support minimal and legacy datasets during transition.

## Backward Compatibility

This extension introduces new file types and metadata fields under the dedicated tms/ modality folder and does not modify or deprecate any existing BIDS entities. Therefore, it is fully backward compatible with current BIDS datasets and tools.

All additions follow BIDS conventions for optional modality folders and sidecar metadata. Existing BIDS datasets remain valid without modification, and TMS datasets using this extension will not interfere with standard processing pipelines for MRI, EEG, or behavioral data.

Where possible, existing BIDS fields and patterns (e.g., coordsystem.json, *_events.tsv, IntendedFor) are reused to ensure maximal tool reuse and minimal changes to validators or data analysis workflows.

## Appendix A: Field Definitions

### *_coordsystem.json Parameters:

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

### *_markers.tsv Parameters:

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
| `Matrix4D`           | array  | 4x4 affine transformation matrix for the coil positioning (instrument markers of Localite systems).     | —        |
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
### *_tms.json Sidecar JSON 

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

### *_tms.tsv Parameters:

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

## Appendix B: Examples

### Example *_coordsystem.json:

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

### Example *_markers.tsv:

```
MarkerID	PeelingDepth	target_x	target_y	target_z	entry_x	entry_y	entry_z	Matrix4D	coil_x	coil_y	coil_z	normal_x	normal_y	normal_z	direction_x	direction_y	direction_z	ElectricFieldMax_x	ElectricFieldMax_y	ElectricFieldMax_z	Timestamp
M01	        15.3	        12.1	    25.4	    43.2	    11.9	    24.8	    42.8	    n/a	        11.7	    24.6	    42.5	    0.0	        0.0	        1.0	        1.0	        0.0	        0.0	            13.5	            26.2	            44.1	        2025-06-01T13:45:20.123Z
M02	        14.7	        18.0	    27.9	    41.0	    17.8	    27.5	    40.5	    [[1,0,0,0],[0,1,0,0],[0,0,1,0],[0,0,0,1]]	n/a	        n/a	        n/a	        n/a	        n/a	        n/a	        n/a	        n/a	        n/a	            19.3	            29.1	            42.2	        2025-06-01T13:45:25.789Z

```

### Examples *_tms.tsv:

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
    "Description": "Type of stimulus pulse train applied.",
    "Levels": {
      "single": "Single pulse",
      "twin": "Two pulses close in time",
      "dual": "Two independent pulses",
      "power": "High-intensity mode",
      "burst": "Rapid-fire burst of pulses",
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