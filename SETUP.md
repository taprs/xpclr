# XP-CLR Setup and Installation Guide

## Overview

This guide covers setting up the XP-CLR environment with numpy/scipy compatibility fixes.

## Important Notes

- **Python Version**: 3.10.x (required for scipy==1.14.0)
- **numpy**: 1.26.4 (downgraded from 2.x for compatibility)
- **scipy**: 1.14.0 (as specified)

## Quick Start

### Option 1: Using Conda Environment File (Recommended)

#### Full Environment (includes all versions)
```bash
conda env create -f environment.yml
conda activate xpclr
```

#### Minimal Environment (lets conda resolve versions)
```bash
conda env create -f environment-minimal.yml
conda activate xpclr
pip install -e .
```

### Option 2: Manual Setup

```bash
# Create environment with Python 3.10
conda create -n xpclr python=3.10 -y

# Activate environment
conda activate xpclr

# Install dependencies with exact versions
pip install scipy==1.14.0 'numpy<2' pandas scikit-allel h5py zarr pytest

# Install xpclr
pip install -e .
```

## Verification

After installation, verify the setup:

```bash
# Check versions
python -c "import scipy, numpy, xpclr; print(f'scipy: {scipy.__version__}, numpy: {numpy.__version__}, xpclr: {xpclr.__version__}')"

# Run tests
pytest xpclr/test/ -v
```

## Usage Examples

### Text Format Analysis
```bash
xpclr --format txt \
  --popA genotypes_pop1.geno \
  --popB genotypes_pop2.geno \
  --map snp_map.snp \
  --samplesA samples_pop1.txt \
  --samplesB samples_pop2.txt \
  --chr 1 \
  --out results.txt
```

### VCF Format Analysis
```bash
xpclr --format vcf \
  --input variants.vcf.gz \
  --samplesA samples_pop1.txt \
  --samplesB samples_pop2.txt \
  --chr chromosome_name \
  --out results.txt
```

### With Optional Parameters
```bash
xpclr --format vcf \
  --input variants.vcf.gz \
  --samplesA samples_pop1.txt \
  --samplesB samples_pop2.txt \
  --chr 1 \
  --out results.txt \
  --rrate 1e-8 \
  --ld 0.95 \
  --maxsnps 200 \
  --minsnps 10 \
  --size 20000 \
  --step 10000 \
  --verbose 20
```

## Compatibility Fixes

### Issue Resolved
Previously, VCF format analysis failed with:
```
TypeError: ufunc 'divide' not supported for the input types
```

### Root Cause
- numpy 2.x has stricter type coercion rules
- VCF loader attempted to load undefined genetic distance field
- Created object array that couldn't be divided

### Solution Applied
1. **Downgraded numpy**: 2.2.6 → 1.26.4
2. **Fixed code**: xpclr/util.py `load_vcf_wrapper()` function
   - Now properly handles None gdistkey parameter
   - Only loads genetic distance when explicitly provided
   - Allows xpclr to compute distances from positions

## Troubleshooting

### ImportError: No module named 'xpclr'
```bash
# Ensure you're in the correct environment
conda activate xpclr

# Reinstall xpclr
cd /path/to/xpclr
pip install -e .
```

### numpy version conflicts
```bash
# Force numpy 1.26.4
pip install --force-reinstall 'numpy==1.26.4'

# Verify
python -c "import numpy; print(numpy.__version__)"
```

### scipy version conflicts
```bash
# Ensure scipy==1.14.0
pip install --force-reinstall 'scipy==1.14.0'
```

## Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| python | 3.10.x | Runtime |
| numpy | 1.26.4 | Numerical computing |
| scipy | 1.14.0 | Scientific computing |
| pandas | 2.3.3+ | Data handling |
| scikit-allel | 1.3.13+ | Genotype arrays |
| h5py | 3.16.0+ | HDF5 support |
| zarr | 2.18.3+ | Zarr support |
| pytest | 9.0.2+ | Testing |

## Platform Support

- ✅ macOS (Intel/ARM)
- ✅ Linux (x86_64)
- ✅ Windows (via WSL or conda)

## Testing

Run the included test suite:
```bash
pytest xpclr/test/ -v
```

Expected output:
```
xpclr/test/test_fromtext.py::test_loadtext PASSED
xpclr/test/test_fromvcf.py::test_load_vcf PASSED
```

## Support

For issues or questions:
1. Check the main repository: https://github.com/hardingnj/xpclr
2. Review the compatibility fixes in xpclr/util.py
3. Ensure you're using the correct environment: `conda activate xpclr`
