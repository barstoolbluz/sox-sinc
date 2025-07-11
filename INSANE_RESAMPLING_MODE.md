# Insane Resampling Mode Documentation

## Overview

**Insane Mode** is an extreme-quality resampling option that uses sox-sinc with extraordinarily long FIR (Finite Impulse Response) filters. This mode prioritizes absolute mathematical precision over processing speed, using filters with millions of taps to achieve near-perfect frequency response.

**⚠️ WARNING**: This mode is computationally intensive and may take several minutes to process even short audio files. It requires significant RAM (potentially several GB for long files).

## When to Use Insane Mode

- **Archival masters**: When creating definitive versions for long-term preservation
- **Critical listening**: For the most demanding audiophile applications
- **Scientific analysis**: When frequency response accuracy is paramount
- **No time constraints**: When processing time is not a concern

## User-Configurable Parameters

### 1. **Filter Length** (Number of Taps)
- **Field name**: `filterTaps`
- **Type**: Integer
- **Default**: 1,638,400
- **Range**: 65,536 to ~8,388,608 (must be even)
- **Help text**: 
  ```
  Number of filter taps (coefficients). More taps = better frequency response 
  but longer processing time. Doubles RAM usage for each doubling of taps.
  
  ⚠️ MEMORY LIMIT WARNING: Sox may fail with allocation errors above 
  ~8-16 million taps due to 32-bit integer limits and array size 
  constraints. Exact limit depends on available RAM and system.
  
  Suggested values:
  - 65,536: Fast, still excellent quality
  - 262,144: High quality, reasonable speed  
  - 1,638,400: Ultra-high quality (default)
  - 4,194,304: Extreme quality, may hit limits
  - 8,388,608: Maximum practical (may fail)
  
  Error messages:
  - "calloc() failed": Too many taps for available memory
  - "arithmetic overflow": Exceeded 32-bit addressing limits
  ```

### 2. **Transition Band Width** (Hz)
- **Field name**: `transitionBand`
- **Type**: Float
- **Default**: 50 Hz
- **Range**: 1 to 1000 Hz
- **Help text**:
  ```
  Width of the transition between passband and stopband in Hz.
  Narrower = steeper filter rolloff but requires more taps.
  
  For upsampling to 176.4/192 kHz:
  - 50 Hz: Exceptional (default)
  - 100 Hz: Excellent
  - 500 Hz: Good, much faster
  ```

### 3. **Passband Frequency** (Hz)
- **Field name**: `passbandFreq`
- **Type**: Float  
- **Default**: Auto (0.45 × Nyquist of source)
- **Range**: 15,000 to (0.499 × source sample rate)
- **Help text**:
  ```
  Upper frequency limit to preserve. Leave at 0 for automatic 
  calculation based on source sample rate.
  
  Manual settings:
  - 20,000 Hz: Preserve up to 20 kHz
  - 22,050 Hz: Preserve up to 22.05 kHz (CD Nyquist)
  - 0: Auto-calculate optimal value
  ```

### 4. **Kaiser Window Beta**
- **Field name**: `kaiserBeta`
- **Type**: Float
- **Default**: 16.0
- **Range**: 0.0 to 50.0
- **Help text**:
  ```
  Kaiser window shape parameter. Higher values give better stopband
  rejection but wider transition band.
  
  - 0: Rectangular window (sharp cutoff, most ringing)
  - 8.6: Excellent rejection (-80 dB)
  - 16.0: Exceptional rejection (-140 dB) [default]
  - 25.0: Extreme rejection (-180 dB)
  ```

### 5. **Phase Response**
- **Field name**: `phaseResponse`
- **Type**: Enum/Select
- **Options**: `linear`, `minimum`, `intermediate`
- **Default**: `linear`
- **Help text**:
  ```
  Filter phase characteristics:
  
  - Linear: Zero phase distortion, symmetric impulse (best for music)
  - Minimum: Minimum delay, asymmetric impulse
  - Intermediate: Custom phase (advanced users only)
  ```

### 6. **Gain Compensation**
- **Field name**: `gainCompensation`
- **Type**: Float
- **Default**: 1.0
- **Range**: 0.25 to 4.0
- **Help text**:
  ```
  Output level adjustment to compensate for filter gain.
  1.0 = no change. Use higher values if output seems quiet.
  ```

### 7. **Allow Aliasing**
- **Field name**: `allowAliasing`
- **Type**: Boolean
- **Default**: false
- **Help text**:
  ```
  Allow aliasing for artistic effect or special cases.
  WARNING: Enables frequencies above Nyquist to fold back.
  Only for experimental use.
  ```

## Technical Explanation

### What Makes It "Insane"?

1. **Filter Length**: Traditional resamplers use filters with hundreds or thousands of taps. Insane mode uses millions, achieving frequency response accuracy within 0.0001 dB of ideal.

2. **Computation Method**: Uses DFT (Discrete Fourier Transform) convolution for efficiency with very long filters, but still requires massive computation.

3. **Memory Usage**: A 1.6M tap filter at 24-bit precision requires ~13 MB just for coefficients, plus working buffers.

4. **32-bit Limitations**: Sox uses 32-bit integers for array indexing in many places, which limits the maximum filter size. Attempting to use more than ~8-16 million taps will likely result in allocation failures or integer overflow errors.

5. **Processing Approach**: 
   - Upsamples to very high intermediate rate (e.g., 4× target)
   - Applies precise lowpass filtering
   - Downsamples to target rate
   - Each stage carefully controlled for maximum precision

### Example Command Generation

For 96 kHz → 192 kHz with default Insane settings:
```bash
sox-sinc input.flac -b 24 -r 192000 output.flac \
  upsample 4 \
  sinc -24000 -n 1638400 -L -b 16 \
  vol 1.0
```

## Performance Expectations

| Filter Taps | RAM Usage | Processing Time* | Quality | Status |
|------------|-----------|-----------------|---------|---------|
| 65,536 | ~100 MB | 5-10 seconds | Excellent | ✓ Always works |
| 262,144 | ~200 MB | 20-40 seconds | Superb | ✓ Always works |
| 1,638,400 | ~500 MB | 2-5 minutes | Exceptional | ✓ Default |
| 4,194,304 | ~1 GB | 5-10 minutes | Near Perfect | ⚠️ May fail |
| 8,388,608 | ~2 GB | 10-20 minutes | Theoretical Limit | ❌ Often fails |

*For a 5-minute stereo track on modern CPU

## Integration Notes

When implementing in your application:

1. **Progress Indication**: Essential due to long processing times
2. **Memory Check**: Verify available RAM before starting
3. **CPU Detection**: Use all available cores (`-j` option)
4. **Temp Files**: May need substantial temp space
5. **Batch Processing**: Consider queuing for multiple files
6. **Presets**: Offer "Insane Fast", "Insane Default", "Insane Ultimate"

## Advanced Tips

- For best results, ensure source files are high-quality (24-bit or higher)
- Dither should be applied as the final step if reducing bit depth
- Consider using intermediate float format for processing chains
- Monitor CPU temperature during extended processing