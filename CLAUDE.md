# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SoX (Sound eXchange) is a comprehensive command-line audio processing utility and library. This fork focuses on FIR (Finite Impulse Response) filter improvements.

## Build Commands

Standard GNU Autotools build process:

```bash
# Configure the build
./configure

# Build SoX
make

# Install (requires root privileges)
make install

# Run tests on installed executables
make installcheck
```

For development from git sources:
```bash
# First time setup (requires automake >= 1.9, autoconf >= 2.62)
autoreconf -i

# Then follow standard build process
```

## Testing

Run comprehensive test suite:
```bash
# Basic lossless conversion tests
cd src
./tests.sh

# More extensive tests
./testall.sh
```

Test specific functionality:
```bash
# Test a simple conversion
./sox input.wav output.aiff

# Test with specific parameters
./sox input.wav -r 44100 -b 16 output.wav
```

## Architecture

Key components:
- **Core library** (`libsox`): Handles format conversion and effects processing
- **Main executable** (`sox`): Command-line interface
- **Format handlers**: Modular support for audio formats (wav.c, mp3.c, flac.c, etc.)
- **Effects processors**: Audio transformation modules (echo.c, reverb.c, etc.)
- **FIR filter components**: `fir.c` and `firfit.c` for finite impulse response filtering

The codebase uses:
- GNU Autotools build system
- Optional external libraries via `--with-*` configure flags
- Dual licensing: GPL for executables, LGPL for library
- OpenMP for parallel processing where available

## Common Development Tasks

When implementing new features or fixes:
1. Follow existing code style and conventions in neighboring files
2. Check for available libraries before adding dependencies
3. Use existing utilities and patterns found in the codebase
4. For format handlers, look at existing formats in src/ for examples
5. For effects, examine existing effects implementations

## Recent Development Focus

Current work centers on:
- FIR filter improvements (increased maximum filter lengths)
- Format handler bug fixes (MP3, AIFF)
- ID3 tag handling enhancements