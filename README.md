# BIDS Extension Proposal: Transcranial Magnetic Stimulation (TMS)

## Status

This proposal is under development and has not yet been submitted as an official BIDS Extension Proposal (BEP). Feedback from the community is welcome and encouraged.

## Overview

TMS is a non-invasive neurostimulation technique used in both research and clinical contexts. Integrating TMS data into BIDS enables:

- Reproducibility of stimulation protocols
- Interoperability between different navigation systems (Localite, Nexstim, etc.)
- Unified documentation of stimulation parameters and coil positioning
- Integration with MRI, EEG/EMG, and behavioral datasets

## File Structure

This extension introduces new files and metadata under the `tms/` directory inside each subject/session:
```
sub-<label>/
	└── ses-<label>/
		└── tms/
			├── sub-<label>[_ses-<label>]_task-<label>[_acq-<label>][_run-<label>]_coordsystem.json
			├── sub-<label>[_ses-<label>]_task-<label>[_acq-<label>][_run-<label>]_tms.tsv
			├── sub-<label>[_ses-<label>]_task-<label>[_acq-<label>][_run-<label>]_tms.json
			├── sub-<label>[_ses-<label>]_task-<label>[_acq-<label>][_run-<label>]_markers.tsv
			└── sub-<label>[_ses-<label>]_task-<label>[_acq-<label>][_run-<label>]_markers.json
```
		
These files document:

- **Registration and navigation systems**
- **Hotspot localization and stimulation protocols**
- **Coil positioning and electric field estimation**
- **Motor response and EMG channel linkage**

## Specification

The full specification is available in the [`specification/`](./specification/) directory, and includes:

- Field descriptions and definitions
- TSV and JSON metadata examples
- Harmonization with other BIDS modalities (e.g., MEG, EEG)

## Authors

This extension is being developed by:

- **Oleg Shevtsov**, ORCID: [0009-0008-3460-0633](https://orcid.org/0009-0008-3460-0633)  
- **Milana Makarova**, ORCID: [0000-0002-9351-6588](https://orcid.org/0000-0002-9351-6588)  
- **Matteo Feurra**, ORCID: [0000-0003-0934-6764](https://orcid.org/0000-0003-0934-6764)

## Contact

For questions or contributions, please contact:  
**Oleg Shevtsov** — <olegshevts@gmail.com>

## License

This project is licensed under the [MIT License](LICENSE).
