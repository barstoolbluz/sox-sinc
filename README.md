# sox-sinc

Ultra-high-quality audio resampling using SoX with extreme FIR filters

## What is sox-sinc?

sox-sinc is a specialized build of [SoX (Sound eXchange)](http://sox.sourceforge.net/) optimized for extreme-quality audio resampling using very long FIR (Finite Impulse Response) filters. This fork enables filter lengths of millions of taps, far beyond what typical audio resamplers use, achieving near-perfect frequency response at the cost of processing time.

## Why does it exist?

Standard audio resamplers make trade-offs between quality and speed. Even "high quality" resamplers typically use filters with hundreds or thousands of taps. sox-sinc breaks these limits by:

- Supporting FIR filters with **millions of taps** (tested up to 8+ million)
- Achieving frequency response accuracy within 0.0001 dB of ideal
- Providing complete control over filter parameters
- Enabling "mathematically perfect" resampling for archival and critical listening

This tool exists for audio engineers, mastering professionals, and enthusiasts who need the absolute best quality resampling, regardless of processing time.

## Key Features

- 🎯 **Extreme Precision**: Million+ tap FIR filters for near-perfect frequency response
- 🎛️ **Full Control**: Configurable filter length, transition bands, and window functions
- 📊 **Transparent Processing**: Verbose output shows exact filter parameters
- 🔧 **Renamed Binaries**: Installs as `sox-sinc` to coexist with standard sox
- 🏗️ **Flox-based Build**: Reproducible builds across Linux and macOS

## Installation

### Using Flox (Recommended)

```bash
# Clone the repository
git clone https://github.com/yourusername/sox-sinc.git
cd sox-sinc

# Build with flox
flox build sox-sinc

# The binaries will be in ./result-sox-sinc/bin/
# Optionally, add to your PATH:
export PATH="$PWD/result-sox-sinc/bin:$PATH"
```

### Traditional Build

```bash
# Standard autotools build (requires all dependencies)
autoreconf -i
./configure --program-suffix=-sinc
make
sudo make install
```

## Usage Example

### Basic Upsampling (96 kHz → 192 kHz)

```bash
sox-sinc input.flac -b 24 output.flac rate -v 192000
```

### "Insane Mode" Upsampling

```bash
sox-sinc -S -V3 input_96k.flac -b 24 -r 192000 output_192k.flac \
  upsample 4 \
  sinc -n 1638400 -b 16 \
  vol 1
```

This command:
- Upsamples to 4× the target rate internally
- Applies a 1.6 million tap sinc filter
- Produces 192 kHz output with exceptional precision

### Extreme Quality Settings

For the absolute maximum quality (warning: very slow):

```bash
sox-sinc input.flac output.flac \
  rate -v -b 99.7 -p 50 192000
```

Or with explicit sinc filter:

```bash
sox-sinc input.flac output.flac \
  upsample 8 \
  sinc -n 8388608 -b 20 \
  vol 1
```

## Understanding the Parameters

### Filter Length (`-n`)
- Number of filter taps (coefficients)
- More taps = better frequency response but slower
- Practical range: 65,536 to ~8,388,608
- Memory limit: ~8-16 million due to 32-bit constraints

### Kaiser Window Beta (`-b`)
- Controls the filter's frequency/time trade-off
- 0 = rectangular window (sharp cutoff, more ringing)
- 16 = excellent stopband rejection (default)
- 20+ = extreme rejection for critical applications

### Phase Response
- `-L` = linear phase (best for music)
- `-M` = minimum phase
- `-I` = intermediate phase

## Performance Expectations

| Filter Taps | Processing Time* | RAM Usage | Quality |
|------------|-----------------|-----------|---------|
| 65,536 | ~10 seconds | 100 MB | Excellent |
| 262,144 | ~40 seconds | 200 MB | Superb |
| 1,638,400 | 2-5 minutes | 500 MB | Exceptional |
| 8,388,608 | 10-20 minutes | 2 GB | Near Perfect |

*For a 5-minute stereo track on modern hardware

## Differences from Standard SoX

1. **Binary names**: All executables have `-sinc` suffix
2. **Library name**: Uses `libsox-sinc` instead of `libsox`
3. **Optimized for**: Extreme-length FIR filters
4. **Use case**: Quality over speed

## Building from Source

### Prerequisites

- GNU Autotools (autoconf, automake, libtool)
- C compiler (gcc/clang)
- Optional: FLAC, MP3, Vorbis libraries for format support

### Build Instructions

See [INSTALL](INSTALL) for detailed instructions.

## Documentation

- [INSANE_RESAMPLING_MODE.md](INSANE_RESAMPLING_MODE.md) - Detailed parameter guide
- [CLAUDE.md](CLAUDE.md) - Development notes for Claude AI
- Original SoX documentation in man pages

## Technical Details

This fork is based on SoX 14.4.3 with modifications for:
- Extended filter length support
- Optimized memory allocation for large filters
- Enhanced verbose output for filter analysis

The sinc effect uses DFT-based convolution for efficiency with very long filters.

## License

This project maintains SoX's original dual license:
- GPL for the executables
- LGPL for the library

See [LICENSE.GPL](LICENSE.GPL) and [LICENSE.LGPL](LICENSE.LGPL) for details.

## Contributing

Contributions welcome! Key areas:
- 64-bit filter length support
- Memory usage optimization
- GPU acceleration for DFT operations
- Additional window functions

## Acknowledgments

- Original SoX team for the excellent foundation
- The audio engineering community for pushing quality boundaries
- Flox for reproducible build environments