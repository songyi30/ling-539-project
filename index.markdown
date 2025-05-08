---
layout: post
title: "Extracting Vowel Quality Using Python"
date: 2025-05-07
---

## 🎯 Goal

This tutorial shows how to extract **vowel formants** (F1, F2) from a `.wav` audio file using the [parselmouth](https://github.com/YannickJadoul/Parselmouth) library, a Python wrapper for Praat.

---

## 📦 Step 1: Install Required Package

You need to install `praat-parselmouth`. Run this in your terminal:

```bash
pip install praat-parselmouth


---

## Formant Extraction

import parselmouth
import numpy as np
import matplotlib.pyplot as plt

def extract_formants(audio_path):
    snd = parselmouth.Sound(audio_path)
    formant = snd.to_formant_burg()

    times = np.linspace(0, snd.duration, 100)
    f1_list = []
    f2_list = []

    for t in times:
        f1 = formant.get_value_at_time(1, t)
        f2 = formant.get_value_at_time(2, t)
        if f1 and f2:
            f1_list.append(f1)
            f2_list.append(f2)

    return f1_list, f2_list

# Example usage
f1s, f2s = extract_formants("vowel_a.wav")

# Plot F1 vs F2 (vowel space)
plt.scatter(f2s, f1s)
plt.xlabel("F2 (Hz)")
plt.ylabel("F1 (Hz)")
plt.title("Vowel Formant Plot (F1 vs F2)")
plt.gca().invert_yaxis()
plt.show()

---
