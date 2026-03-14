# Parkinson's Tremor — Real Data Filter Analysis

DSP pipeline applied to real Parkinson's patient EMG data from PhysioNet.
Tremor frequency automatically detected via FFT, then suppressed using three IIR Biquad filter topologies.

Gerçek Parkinson hastası EMG verisine uygulanan DSP pipeline. Tremor frekansı FFT ile otomatik tespit edilmiş, ardından üç farklı IIR Biquad filtresi ile bastırılmıştır.

---

## Dataset / Veri Seti

| | |
|---|---|
| **Source / Kaynak** | PhysioNet Tremor Database (tremordb 1.0.0) |
| **Patient / Hasta** | g9 — DBS off, medication off, right hand |
| **Sampling rate / Örnekleme hızı** | 100 Hz |
| **Detected tremor frequency / Tespit edilen tremor frekansı** | 6.53 Hz |

---

## How It Works / Nasıl Çalışır
```
Raw EMG Data (PhysioNet)
        ↓
DC Offset Removal — signal centered around zero
        ↓
FFT — dominant tremor frequency auto-detected (6.53 Hz)
        ↓
IIR Biquad Filter Design — 3 topologies
        ↓
Zero-Phase Filtering (filtfilt) — no phase distortion
        ↓
Metrics + Visualization
```

---

## Filters / Filtreler

### 1. Notch Filter / Notch Filtre

**EN:** A 2nd-order IIR filter that attenuates a very narrow frequency band centered at the target frequency (6.53 Hz), while leaving all other frequencies completely untouched. The Q factor controls the width of the notch — higher Q means narrower and more selective filtering. Q=15 gives a bandwidth of 0.43 Hz around the tremor peak.

**TR:** Hedef frekans (6.53 Hz) etrafında çok dar bir bandı bastıran 2. dereceden IIR filtredir. Diğer tüm frekanslara dokunmaz. Q faktörü çentiğin genişliğini belirler — Q ne kadar yüksekse filtre o kadar seçici olur. Q=15 ile tremor tepesi etrafında 0.43 Hz genişliğinde bir çentik oluşturulur.
```
|H(f)|
  1 ─────────╮   ╭─────────
             │   │
  0          ╰───╯
             6.53 Hz
```

> Best for / En iyi kullanım: When tremor frequency is stable and known.
> Tremor frekansı sabit ve biliniyorsa.

---

### 2. Band-pass Filter / Band-geçiren Filtre

**EN:** A 2nd-order Butterworth filter that passes only the frequency band between 4.53–8.53 Hz and blocks everything outside. It does not suppress the tremor — instead it isolates it. Used for analysis and visualization of the tremor component alone.

**TR:** Sadece 4.53–8.53 Hz arasındaki frekansları geçiren, dışındakileri engelleyen 2. dereceden Butterworth filtredir. Tremoru bastırmaz, aksine onu izole eder. Yalnızca tremor bileşenini analiz etmek ve görselleştirmek için kullanılır.
```
|H(f)|
  0 ────╮           ╭────
        │           │
  1     ╰───────────╯
      4.53 Hz   8.53 Hz
```

> Best for / En iyi kullanım: Isolating and studying the tremor component.
> Tremor bileşenini izole edip incelemek için.

---

### 3. Band-stop Filter / Band-durduran Filtre

**EN:** A 2nd-order Butterworth filter that blocks the entire frequency band between 4.53–8.53 Hz and passes everything outside. It is a wider version of the notch filter, useful when the tremor frequency is not precisely known or varies over time.

**TR:** 4.53–8.53 Hz arasındaki tüm frekans bandını engelleyen, dışındakileri geçiren 2. dereceden Butterworth filtredir. Notch filtrenin geniş bantlı versiyonudur. Tremor frekansı tam bilinmiyorsa veya zamanla değişiyorsa tercih edilir.
```
|H(f)|
  1 ────╮           ╭────
        │           │
  0     ╰───────────╯
      4.53 Hz   8.53 Hz
```

> Best for / En iyi kullanım: When tremor frequency is variable or uncertain.
> Tremor frekansı değişken veya belirsiz olduğunda.

---

## Filters & Results / Filtreler ve Sonuçlar

| Filter / Filtre | Attenuation / Bastırma | Signal Preservation / Sinyal Koruma | Status / Durum |
|--------|-------------|---------------------|--------|
| Notch (Q=15) | -36.0 dB | 99.5% | ✅ Effective / Etkili |
| Band-pass | -0.1 dB | 3.8% | ❌ Isolation only / Sadece izolasyon |
| Band-stop | -45.3 dB | 78.3% | ✅ Effective / Etkili |

> **Best result / En iyi sonuç:** Notch filter @ 6.53 Hz — highest signal preservation with strong attenuation. / En yüksek sinyal koruma ile güçlü bastırma.

---

## Output / Çıktı

![Filter Comparison](real_data_filter_comparison.png)

### How to read the plot / Grafik nasıl okunur

The output figure contains **4 rows × 2 columns** of panels.
Çıktı görseli **4 satır × 2 sütun** panelden oluşmaktadır.

**Left column / Sol sütun — Time Domain / Zaman Domeninde:**

| Row / Satır | Content / İçerik |
|-------------|-----------------|
| Row 1 / 1. Satır | Raw signal — unfiltered tremor + noise / Ham sinyal — filtrelenmemiş tremor + gürültü |
| Row 2 / 2. Satır | Notch output — tremor peak removed, rest preserved / Notch çıktısı — tremor tepesi kaldırıldı, gerisi korundu |
| Row 3 / 3. Satır | Band-pass output — only tremor band remains / Band-pass çıktısı — sadece tremor bandı kaldı |
| Row 4 / 4. Satır | Band-stop output — tremor band removed, rest passes / Band-stop çıktısı — tremor bandı kaldırıldı, gerisi geçti |

**Right column / Sağ sütun — FFT Spectrum / FFT Spektrumu:**

| Row / Satır | Content / İçerik |
|-------------|-----------------|
| Row 1 / 1. Satır | Raw spectrum — dominant peak at 6.53 Hz / Ham spektrum — 6.53 Hz'de baskın tepe |
| Row 2 / 2. Satır | Notch spectrum — 6.53 Hz peak eliminated / Notch spektrumu — 6.53 Hz tepesi yok edildi |
| Row 3 / 3. Satır | Band-pass spectrum — only 4.53–8.53 Hz visible / Band-pass spektrumu — sadece 4.53–8.53 Hz görünür |
| Row 4 / 4. Satır | Band-stop spectrum — 4.53–8.53 Hz suppressed / Band-stop spektrumu — 4.53–8.53 Hz bastırıldı |

> **Dashed white line / Kesik beyaz çizgi:** marks the detected tremor frequency (6.53 Hz). / Tespit edilen tremor frekansını (6.53 Hz) gösterir.

> **Gray background signal / Gri arka plan sinyali:** raw signal shown for comparison. / Karşılaştırma için ham sinyal gösterilir.

---

## Files / Dosyalar

| File / Dosya | Description / Açıklama |
|------|-------------|
| `real_tremor.py` | Main script — FFT detection, filter design, metrics, visualization / Ana kod |
| `g9r15of.rit` | Raw EMG recording from PhysioNet / Ham EMG kaydı |
| `real_data_filter_comparison.png` | 4×2 panel filter comparison plot / Filtre karşılaştırma grafiği |

---

## Usage / Kullanım
```bash
pip install numpy scipy matplotlib wfdb
python real_tremor.py
```

---

## Reference / Kaynak

Rivlin-Etzion M, et al. (2006). Effect of Deep Brain Stimulation on Parkinsonian Tremor.
PhysioNet. https://physionet.org/content/tremordb/

## Author
**Sinan Berke AKÇA**
- GitHub: [@Sinansteiger](https://github.com/Sinansteiger)
- LinkedIn: [linkedin.com/in/sinan-berke-akça-623716256](https://www.linkedin.com/in/sinan-berke-akça-623716256)
