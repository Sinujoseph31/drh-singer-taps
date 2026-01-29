# DRH Singer Taps

[![Python 3.6+](https://img.shields.io/badge/python-3.6+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Singer taps for extracting diabetes research data from various CGM (Continuous Glucose Monitoring) manufacturers.

## Overview

This repository contains Singer-compliant data extraction taps for diabetes research data, supporting multiple CGM manufacturers and data formats. The taps use the [DRH Singer SDK](https://github.com/Sinujoseph31/drh-singer-sdk) for standardized data emission and validation.

## Supported Taps

### 🩺 tap_dexcom
Extracts data from Dexcom CGM systems supporting:
- **Dexcom Clarity** export format
- **Dexcom API** export format
- Participant metadata
- Study metadata
- Meal and fitness data

### 🩺 tap_simplera
Extracts data from Simplera CGM systems (Coming Soon)

## Features

- ✅ **6-Step Validation** - Comprehensive data validation before extraction
- ✅ **Granular Diagnostics** - Detailed validation reports for troubleshooting
- ✅ **Multi-Format Support** - Handles various CGM manufacturer formats
- ✅ **Environment-Based Configuration** - Flexible configuration via environment variables
- ✅ **Auto-Bootstrap** - Automatic virtual environment and dependency setup

## Installation

### Prerequisites

- Python 3.6 or higher
- Access to CGM data files in CSV format

### Install Dependencies

```bash
# Clone the repository
git clone https://github.com/Sinujoseph31/DRH-dexcom-tap.git
cd DRH-dexcom-tap

# Run the tap (it will automatically bootstrap the virtual environment and install dependencies)
python taps/tap_dexcom.py
```

The `requirements.txt` includes:
```txt
git+https://github.com/Sinujoseph31/drh-singer-sdk.git
```

## Quick Start

### 1. Set Environment Variables

```bash
export STUDY_DATA_PATH="/path/to/your/cgm/data"
```

### 2. Run a Tap

```bash
cd taps
python tap_dexcom.py > output.json 2> logs.txt
```

### 3. View Output

```bash
# View emitted records
cat output.json | grep '"type": "RECORD"' | head -5

# View validation diagnostics
cat output.json | grep '"stream": "drh_diagnostics"'
```

## Configuration

### Environment Variables

#### Required
- `STUDY_DATA_PATH` - Path to directory containing CSV data files

#### Optional File Overrides
- `PARTICIPANT_FILE` - Default: `participant.csv`
- `CGM_FILE_METADATA_FILE` - Default: `cgm_file_metadata.csv`
- `INSTITUTION_FILE` - Default: `institution.csv`
- `LAB_FILE` - Default: `lab.csv`
- `STUDY_FILE` - Default: `study.csv`
- `SITE_FILE` - Default: `site.csv`
- `INVESTIGATOR_FILE` - Default: `investigator.csv`
- `PUBLICATION_FILE` - Default: `publication.csv`
- `AUTHOR_FILE` - Default: `author.csv`
- `MEAL_FILE_METADATA_FILE` - Default: `meal_file_metadata.csv`
- `MEAL_DATA_FILE` - Default: `meal_data.csv`
- `FITNESS_FILE_METADATA_FILE` - Default: `fitness_file_metadata.csv`
- `FITNESS_DATA_FILE` - Default: `fitness_data.csv`

### Example Configuration

```bash
# Basic configuration
export STUDY_DATA_PATH="./data/study-001"

# Custom file names
export PARTICIPANT_FILE="participants_2024.csv"
export CGM_FILE_METADATA_FILE="cgm_metadata_v2.csv"

# Run tap
python taps/tap_dexcom.py
```

## Data Directory Structure

```
study-data/
├── participant.csv
├── institution.csv
├── lab.csv
├── study.csv
├── site.csv
├── investigator.csv
├── publication.csv
├── author.csv
├── cgm_file_metadata.csv
├── cgm_tracing_P-001_clarity_00001.csv
├── cgm_tracing_P-001_api_00002.csv
├── cgm_tracing_P-002_clarity_00003.csv
├── meal_data.csv (optional)
├── meal_file_metadata.csv (optional)
├── fitness_data.csv (optional)
└── fitness_file_metadata.csv (optional)
```

## Validation Checks

Each tap performs 6 comprehensive validation checks:

1. **Folder & Resource Check** - Verifies data directory exists and is accessible
2. **Mandatory File Presence** - Ensures all required files are present
3. **File Extension Validation** - Confirms all files have `.csv` extension
4. **File Schema Check** - Validates CSV headers match expected schema
5. **CGM Metadata Consistency** - Verifies files referenced in metadata exist
6. **CGM Data Integrity** - Ensures CGM data files contain actual data rows

### Validation Output

```json
{"type": "RECORD", "stream": "drh_diagnostics", "record": {
  "record_id": "uuid",
  "check_id": 1,
  "check_name": "Folder & Resource Check",
  "status": "PASSED",
  "details": "Detected 15 files from folder: ./data/study-001"
}}

{"type": "RECORD", "stream": "drh_validation_reports", "record": {
  "timestamp": "2024-01-01T12:00:00Z",
  "folder_name": "study-001",
  "overall_status": "SUCCESS",
  "report_json": "{\"status\": \"Validation complete\", \"checks_run\": 6, \"failed_checks\": 0}"
}}
```

## Output Streams

### Data Streams
- `cgm_tracing` - CGM glucose measurements
- `participant` - Participant information
- `study` - Study metadata
- `site` - Study sites
- `investigator` - Investigators
- `institution` - Institutions
- `lab` - Laboratories
- `publication` - Publications
- `author` - Authors
- `meal_data` - Meal data (optional)
- `fitness_data` - Fitness data (optional)

### Metadata Streams
- `cgm_file_metadata` - CGM file metadata
- `meal_file_metadata` - Meal file metadata (optional)
- `fitness_file_metadata` - Fitness file metadata (optional)

### Validation Streams
- `drh_diagnostics` - Detailed validation results
- `drh_validation_reports` - Validation summary

## Example Usage

### Basic Extraction

```bash
export STUDY_DATA_PATH="./taps/raw-data/dexcom-synthetic-cgm"
python taps/tap_dexcom.py > output.json
```

### Filter Specific Streams

```bash
# Extract only CGM tracing data
python taps/tap_dexcom.py | grep '"stream": "cgm_tracing"' > cgm_data.json

# Extract only validation diagnostics
python taps/tap_dexcom.py | grep '"stream": "drh_diagnostics"' > validation.json
```

### Pipe to Singer Target

```bash
# Example: Load to PostgreSQL using target-postgres
python taps/tap_dexcom.py | target-postgres --config target-config.json
```

## Troubleshooting

### Validation Failures

If validation fails, check the `drh_diagnostics` stream for detailed error messages:

```bash
python taps/tap_dexcom.py 2>&1 | grep "FAILED"
```

### Missing Files

Ensure all mandatory files are present in `STUDY_DATA_PATH`:
- `participant.csv`
- `institution.csv`
- `lab.csv`
- `study.csv`
- `site.csv`
- `investigator.csv`
- `publication.csv`
- `author.csv`
- `cgm_file_metadata.csv`

### File Extension Issues

The tap normalizes filenames by adding `.csv` extension if missing in metadata files.

## Development

### Project Structure

```
DRH-dexcom-tap/
├── taps/
│   ├── tap_dexcom.py          # Dexcom tap
│   ├── tap_simplera.py        # Simplera tap
│   └── raw-data/              # Sample data for testing
│       └── dexcom-synthetic-cgm/
├── requirements.txt           # Dependencies
└── README.md                  # This file
```

### Adding a New Tap

1. Create new tap file: `taps/tap_manufacturer.py`
2. Import DRH SDK: `from drh_sdk import DRHSingerEmitter`
3. Implement data extraction logic
4. Follow existing tap patterns for validation

## Dependencies

- [drh-singer-sdk](https://github.com/Sinujoseph31/drh-singer-sdk) - DRH Singer Protocol SDK
- `jsonschema>=4.0.0` - JSON schema validation

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Related Projects

- [DRH Singer SDK](https://github.com/Sinujoseph31/drh-singer-sdk) - Core SDK for DRH Singer protocol

## Support

- **Issues**: [GitHub Issues](https://github.com/Sinujoseph31/DRH-dexcom-tap/issues)
- **Documentation**: [Full Documentation](https://github.com/Sinujoseph31/DRH-dexcom-tap/wiki)

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/new-tap`)
3. Commit your changes (`git commit -m 'Add new tap'`)
4. Push to the branch (`git push origin feature/new-tap`)
5. Open a Pull Request
