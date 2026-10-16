# Unmix the Signal (ICA)

Recovering clean audio sources from mixed recordings using Independent Component Analysis (ICA), with centering, PCA whitening and FastICA all implemented from scratch in NumPy. No library ICA is used.

## What this project does

1. Loads clean sources (speech and music) and mixes them with my own mixing matrix, so each "microphone" hears a blend.
2. Centers the mixed signals.
3. Whitens them with PCA, using the covariance matrix and eigendecomposition.
4. Runs FastICA (tanh contrast, symmetric decorrelation) to find the unmixing matrix.
5. Exports the separated signals as playable `.wav` files.
6. Evaluates the result against the originals with correlation and SDR, after fixing the order, sign and loudness that ICA cannot know.

Bonus work: PCA-only versus ICA comparison and separation quality at different SNR levels.

## Audio sources

| File | What it is | Source link | Licence |
|---|---|---|---|
| `speech.wav` | “Podcast AI Slop” by qubodup (speech clip) | https://freesound.org/people/qubodup/sounds/842832/ | Creative Commons 0 (CC0) |
| `music.wav` | “acoustic picking guitar 100bpm key of C” by ajubamusic (converted from MP3 to WAV) | https://freesound.org/s/320800/	 | Creative Commons Attribution 3.0 (CC BY 3.0) |

Add one row for every extra file you use (ambient sound, songs for the Shazam library, phone recordings).

## Repository contents

| Path | Description |
|---|---|
| `ica_implementation.ipynb` | The full code, results, plots and write-up |
| `speech.wav`, `music.wav` | Original clean sources |
| `mixed_mic1.wav`, `mixed_mic2.wav` | The mixed recordings (output of the notebook) |
| `separated_source1.wav`, `separated_source2.wav` | The separated signals (output of the notebook) |
| `README.md` | This file |

The separated files can come out in a different order, upside-down, or at a different loudness than the originals. This is normal for ICA (see "Notes" below).

## How to run

1. Install Python 3 and the libraries: `pip install numpy scipy matplotlib`
2. Put the `.wav` files in the same folder as the notebook.
3. Open `ica_implementation.ipynb` in VS Code or Jupyter and choose **Restart**, then **Run All**.

## Results

Test with 2 sources (speech and music) mixed into 2 signals, with no noise added:

| Method | Speech | Music |
|---|---|---|
| ICA: correlation with original | 1.0000 | 1.0000 |
| ICA: SDR | 48.5 dB | 53.0 dB |
| PCA whitening only: correlation | 0.79 | 0.80 |
| PCA whitening only: SDR | 4.3 dB | 4.4 dB |

ICA gives one clean source per output. After PCA whitening alone, each output is still a blend of both sources.

### Separation quality at different noise levels

Gaussian noise was added to the mixed signals at several signal-to-noise ratios (SNR) before running the same pipeline.

| SNR of the recordings | ICA SDR | PCA-only SDR |
|---|---|---|
| 40 dB | 34.6 dB | 4.4 dB |
| 30 dB | 24.7 dB | 4.3 dB |
| 20 dB | 14.9 dB | 4.1 dB |
| 10 dB | 6.5 dB | 3.1 dB |
| 5 dB | 3.6 dB | 2.3 dB |
| 0 dB | 1.9 dB | 1.4 dB |

ICA quality falls as noise increases, and at 0 dB the two methods are close. PCA stays around 4 dB because it never separates the sources.

## Notes and limitations

- **Scale and permutation ambiguity.** ICA cannot know which output is source 1, which way up a wave is, or how loud the original was. The evaluation matches each original to its best estimate by correlation, then fixes the sign and loudness before computing SDR.
- **Sample rates.** The two source files had different sample rates (48000 Hz and 44100 Hz), so the music is resampled to match the speech before mixing.
- **Synthetic mixing.** The mixing is exactly linear and has no noise or echo, so near-perfect scores are expected. Real recordings from rooms contain echoes and delays, so results would be lower.
- **Requirements of ICA.** It needs at least as many mixtures as sources, and at most one source can be Gaussian (bell-shaped).
