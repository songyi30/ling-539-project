---
layout: post
title: "Measuring Vowel Duration and Formants in Python"
date: 2025-05-07
---

This tutorial shows how to extract **vowel duration** and **formants** (F1 and F2), two key acoustic features used to analyze vowel quality. While this can be done manually using software like [Praat](https://www.fon.hum.uva.nl/praat/)., it becomes time-consuming when working with large datasets. The Python code provided below automates the process for multiple `.wav` recordings.

---

## Requirements

1. You need `.wav` recordings of **isolated vowels** which are saved in a single **forlder**.
2. Install `parselmouth` if you haven't already:

    ```bash
    pip install praat-parselmouth
    ```

---

## Vowel Duration

To measure vowel duration, we'll use the `librosa` library to calculate vowel amplitude (energy) and detect how long the vowel lasts. `librosa` is widely used for analyzing and processing audio signals. In this tutorial, it is used to load .wav files, calculate short-time energy, and convert audio frames to time values in order to estimate vowel duration.

```python
import os
import librosa
import numpy as np
import csv

def vowel_duration(folder_path, threshold=0.01, output_csv="vowel_duration.csv"):
    """
    Estimate vowel duration (in milliseconds) from .wav files in a folder.
    Saves the output as a CSV file.
    """
    durations = {}

    for fname in os.listdir(folder_path): #make sure that you put a rigth path to the forlder where the audio files are located
        if fname.endswith(".wav"):
            path = os.path.join(folder_path, fname)
            y, sr = librosa.load(path)
            energy = librosa.feature.rms(y=y)[0]
            frames = np.nonzero(energy > threshold)[0]
            times = librosa.frames_to_time(frames, sr=sr)

            duration_ms = (times[-1] - times[0]) * 1000 if len(times) > 0 else 0.0
            durations[fname] = duration_ms

    with open(output_csv, "w", newline="") as file:
        writer = csv.writer(file)
        writer.writerow(["Filename", "Vowel Duration (ms)"])
        for fname, duration in durations.items():
            writer.writerow([fname, f"{duration:.1f}"])

    print(f"\nResults saved to: {output_csv}")
```
### Output
In the same folder where the audio files are saved, you will find a CSV file named **vowel_duration.csv**.


---

## Vowel Formants (F1 & F2)

To analyze vowel quality, the first (F1) and second (F2) formants are commonly used. The code below uses parselmouth (a Python wrapper for Praat) to extract average formant values for each file. parselmouth is a Python interface to Praat, a software tool commonly used in phonetics research. It allows users to access Praat's functionality—such as formant tracking and pitch analysis—directly from Python. In this tutorial, `parselmouth` is used to extract the first and second formants (F1 and F2), which are key acoustic features for analyzing vowel quality across multiple audio files.

```python
import os
import parselmouth
import numpy as np
import csv

def vowel_formants(folder_path, output_csv="vowel_formants.csv"): #make sure that you put a rigth path to the forlder where the audio files are located
    """
    Measure vowel formants (f1 and f2) from .wav files in a folder.
    Saves the output as a CSV file.
    """
    with open(output_csv, "w", newline="") as file:
        writer = csv.writer(file)
        writer.writerow(["Filename", "Average F1 (Hz)", "Average F2 (Hz)"])

        for fname in os.listdir(folder_path):
            if fname.endswith(".wav"):
                path = os.path.join(folder_path, fname)
                sound = parselmouth.Sound(path)
                formant = sound.to_formant_burg()
                times = np.linspace(0, sound.duration, num=100)

                f1_values, f2_values = [], []
                for t in times:
                    f1 = formant.get_value_at_time(1, t)
                    f2 = formant.get_value_at_time(2, t)
                    if f1 and f2:
                        f1_values.append(f1)
                        f2_values.append(f2)

                avg_f1 = np.mean(f1_values) if f1_values else 0.0
                avg_f2 = np.mean(f2_values) if f2_values else 0.0
                writer.writerow([fname, round(avg_f1, 1), round(avg_f2, 1)])

    print(f"\nFormant results saved to: {output_csv}")
```
### Output
In the same folder where the audio files are saved, you will find a CSV file named **vowel_formants.csv**.
