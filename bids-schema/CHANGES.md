# CHANGES for TMS Extension (BEP-TMS)

This file summarizes all schema-level changes introduced to support the new TMS modality.

## Added

### objects/
- `metadata.yaml`: Added detailed metadata definitions for TMS-related fields from `_tms.json`, `_markers.json`, and `_coordsystem.json`.
- `modalities.yaml`: Registered new modality `"tms"`.
- `suffixes.yaml`: Added new suffix `"tms"` with description.
- `extensions.yaml`: Registered extension `"tms"` under BIDS extension proposals.

### rules/files/raw/
- `tms.yaml`: Added valid file naming rules for TMS-related suffixes: `tms`, `markers`, `coordsystem`.

### rules/sidecars/
- `tms.yaml`: Defined required and optional metadata fields for `*_tms.json`, `*_markers.json`, and `*_coordsystem.json`.

### rules/tabular_data/
- `tms.yaml`: Defined tabular structure for `*_tms.tsv` and `*_markers.tsv`.

## Notes

These changes support structured recording of Transcranial Magnetic Stimulation (TMS) experiments, including:
- Stimulation parameters
- Coil definitions via `CoilSet`
- Navigation and targeting markers
- Electric field modeling outputs
- Coordinate system registration
