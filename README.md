# Matched filtering , ISI and eye diagram

## 📌 Overview

This project simulates a basic **BPSK (Binary Phase Shift Keying) digital communication system** using Python.

The simulation demonstrates the complete transmission and reception process:

**Random Bits → BPSK Mapping → Upsampling → RRC Pulse Shaping → AWGN Channel → Matched Filtering → Delay Compensation → Symbol Sampling → Eye Diagram Analysis**

The project implements the **Root Raised Cosine (RRC) filter manually using NumPy**, without relying on a dedicated communication toolbox.

The effect of different **Signal-to-Noise Ratio (SNR)** values is also investigated using eye diagrams and symbol-alignment plots.

---

## 🎯 Objectives

The main objectives of this project are:

* Generate random binary data.
* Convert binary data into BPSK symbols.
* Perform signal upsampling.
* Implement an RRC pulse-shaping filter from mathematical equations.
* Transmit the pulse-shaped signal.
* Add Additive White Gaussian Noise (AWGN).
* Implement a matched RRC filter at the receiver.
* Compensate for transmitter and receiver filter delays.
* Perform optimum symbol sampling.
* Generate eye diagrams.
* Compare system performance at different SNR levels.
* Validate received symbols against the original transmitted symbols.

---

## 🧠 System Block Diagram

```text
                 TRANSMITTER
                 
 Random Bits
     │
     ▼
 BPSK Mapping
  (0 → -1)
  (1 → +1)
     │
     ▼
   Upsampling
  8 Samples/Symbol
     │
     ▼
 RRC Pulse-Shaping Filter
     │
     ▼
 Transmitted Signal
     │
     ▼
 ┌───────────────────────┐
 │      AWGN Channel     │
 │  SNR = 30 dB / 10 dB  │
 └───────────────────────┘
     │
     ▼
 Received Noisy Signal
     │
     ▼
 Matched RRC Filter
     │
     ▼
 Delay Compensation
     │
     ▼
 Optimum Sampling
     │
     ▼
 Recovered BPSK Symbols
     │
     ├──────────────► Symbol Alignment Plot
     │
     ▼
   Eye Diagram
```

---

## ⚙️ System Parameters

| Parameter                |        Value |
| ------------------------ | -----------: |
| Modulation               |         BPSK |
| Number of Symbols        |         1000 |
| Samples per Symbol (SPS) |            8 |
| Filter Span              |    6 Symbols |
| Roll-off Factor (α)      |          0.5 |
| Symbol Period (Ts)       |          1.0 |
| Sampling Frequency (Fs)  |         8 Hz |
| RRC Filter Length        |   49 Samples |
| SNR Test Cases           | 30 dB, 10 dB |
| Timing Offset            |    0 Samples |

---

## 🔧 Technologies Used

* **Python**
* **NumPy**
* **Matplotlib**

No dedicated communication toolbox is required.

### Required Python Libraries

```bash
pip install numpy matplotlib
```

---

## 📂 Project Structure

```text
BPSK-RRC-Eye-Diagram/
│
├── bpsk_rrc_eye_diagram.py
├── README.md
└── results/
    ├── eye_diagram_30dB.png
    ├── eye_diagram_10dB.png
    ├── symbol_alignment_30dB.png
    └── symbol_alignment_10dB.png
```

> The `results` folder can be added if you want to store screenshots of the generated graphs.

---

# 🔬 Working Principle

## 1. Random Bit Generation

The simulation generates 1000 random binary values:

```python
bits = np.random.randint(0, 2, num_symbols)
```

The generated bits contain either:

```text
0 or 1
```

---

## 2. BPSK Mapping

The binary values are converted into BPSK symbols using:

```python
symbols = 2 * bits - 1
```

Therefore:

```text
Bit 0 → -1
Bit 1 → +1
```

This produces the BPSK symbol sequence.

---

## 3. Upsampling

The system uses:

```text
8 samples/symbol
```

The symbols are therefore upsampled before pulse shaping.

```python
tx_up = np.zeros(num_symbols * sps)
tx_up[::sps] = symbols
```

This inserts zeros between consecutive symbols.

---

## 4. Root Raised Cosine Filter

An **RRC filter** is generated mathematically.

The important parameters are:

```text
Roll-off factor α = 0.5
Symbol period Ts = 1
Sampling frequency Fs = 8
Filter span = 6 symbols
```

The filter length is calculated as:

```python
filter_len = span * sps + 1
```

Therefore:

```text
Filter Length = 6 × 8 + 1
              = 49 samples
```

The filter coefficients are normalized by their energy.

---

## 5. Pulse Shaping

The upsampled BPSK sequence is convolved with the RRC filter:

```python
tx_sig = np.convolve(tx_up, h_rrc, mode='full')
```

This produces the pulse-shaped transmitted signal.

Pulse shaping helps control the bandwidth of the transmitted signal and provides the desired response when combined with the receiver's matched filter.

---

# 📡 AWGN Channel

To simulate a practical communication channel, Additive White Gaussian Noise is added to the transmitted signal.

Two SNR conditions are tested:

```python
snr_test_cases = [30, 10]
```

### 30 dB SNR

Represents a relatively high-SNR condition.

The received signal contains comparatively less noise.

### 10 dB SNR

Represents a lower-SNR condition.

The received signal contains more noise, making symbol detection more difficult.

---

# 🔄 Matched Filtering

At the receiver, the same RRC filter is used as a matched filter:

```python
rx_matched = np.convolve(rx_sig, h_rrc, mode='full')
```

The combination of:

```text
RRC Transmit Filter
        +
RRC Receive Filter
        =
Raised Cosine Overall Response
```

provides the desired zero-crossing behavior at the symbol sampling instants.

---

# ⏱️ Filter Delay Compensation

Each 49-sample FIR filter introduces a delay of:

```text
(49 - 1) / 2 = 24 samples
```

There are two filters:

```text
Transmitter RRC → 24 samples
Receiver RRC    → 24 samples
```

Therefore, the total cascade delay is:

```text
24 + 24 = 48 samples
```

The program calculates this as:

```python
total_cascade_delay = filter_len - 1
```

which gives:

```text
48 samples
```

The receiver removes this delay before performing symbol sampling.

---

# 🎯 Optimum Symbol Sampling

The receiver samples the matched-filter output every 8 samples:

```python
detected_symbols = rx_matched[
    optimum_sample_idx :: sps
][:num_symbols]
```

where:

```python
optimum_sample_idx = total_cascade_delay + timing_offset
```

For the current simulation:

```text
total cascade delay = 48 samples
timing offset = 0
```

Therefore, sampling begins at:

```text
Sample 48
```

and continues every:

```text
8 samples
```

---

# 👁️ Eye Diagram

The project generates eye diagrams using two-symbol-long signal segments.

The eye diagram is useful for visually analyzing:

* Timing margin
* Noise
* Inter-symbol interference
* Sampling point
* Signal quality

The simulation compares:

```text
30 dB SNR
vs.
10 dB SNR
```

### High SNR — 30 dB

The eye diagram should appear relatively open and well defined.

This indicates that the signal has a larger separation between the possible symbol trajectories.

### Lower SNR — 10 dB

The traces become more spread out because of increased noise.

The eye becomes less clean, demonstrating the effect of reduced SNR on the received signal.

---

# 📊 Symbol Alignment Validation

The program also compares:

```text
Original BPSK Symbols
        vs.
Sampled Matched-Filter Output
```

This provides a direct validation of the receiver sampling process.

At high SNR, the received samples should remain close to the original BPSK levels:

```text
+1
-1
```

At lower SNR, the received samples show greater variation because of noise.

---

# 📈 Expected Results

The simulation produces four main plots.

### 1. Eye Diagram — 30 dB

Shows a relatively clean and open eye.

### 2. Symbol Alignment — 30 dB

Received samples should closely follow the original BPSK symbols.

### 3. Eye Diagram — 10 dB

Shows increased spreading of the signal traces due to noise.

### 4. Symbol Alignment — 10 dB

Received samples show greater deviation from the ideal symbol levels.

---

# ▶️ How to Run

## Step 1 — Install Python

Install Python 3.x on your system.

Check the installation:

```bash
python --version
```

---

## Step 2 — Install Required Libraries

Run:

```bash
pip install numpy matplotlib
```

---

## Step 3 — Clone the Repository

```bash
git clone https://github.com/YOUR_USERNAME/BPSK-RRC-Eye-Diagram.git
```

Navigate into the project:

```bash
cd BPSK-RRC-Eye-Diagram
```

---

## Step 4 — Run the Program

```bash
python bpsk_rrc_eye_diagram.py
```

The program will display:

* Validation information
* Eye diagram for 30 dB SNR
* Symbol alignment plot for 30 dB SNR
* Eye diagram for 10 dB SNR
* Symbol alignment plot for 10 dB SNR

---

# 🧪 Example Validation Output

```text
--- Mandatory Validation ---
Filter Length: 49 samples
Total Cascade Delay (Tx + Rx): 48 samples
Sampling occurs only AFTER removing 48 samples.
------------------------------
```

---

# 📚 Key Communication Concepts Demonstrated

This project provides practical implementation of several digital communication concepts:

1. **BPSK Modulation**
2. **Symbol Mapping**
3. **Upsampling**
4. **Pulse Shaping**
5. **Root Raised Cosine Filtering**
6. **AWGN Channel**
7. **Signal-to-Noise Ratio**
8. **Matched Filtering**
9. **FIR Filter Delay**
10. **Optimum Symbol Sampling**
11. **Eye Diagram Analysis**
12. **Symbol Alignment Validation**

---

# 🚀 Possible Future Improvements

The project can be extended with:

* [ ] BER calculation
* [ ] BER vs SNR curve
* [ ] QPSK modulation
* [ ] 16-QAM modulation
* [ ] Adjustable roll-off factor
* [ ] Adjustable timing offset
* [ ] Timing recovery
* [ ] Carrier frequency offset
* [ ] Carrier phase offset
* [ ] Constellation diagrams
* [ ] Raised Cosine vs Root Raised Cosine comparison
* [ ] Interactive SNR control
* [ ] Real-time signal visualization
* [ ] Channel impulse-response simulation

---

# 🎓 Learning Outcomes

After completing this project, the following concepts can be understood practically:

* How BPSK symbols are generated.
* Why upsampling is required for pulse shaping.
* How an RRC filter shapes a digital communication signal.
* Why matched filtering is used at the receiver.
* How FIR filters introduce delay.
* Why correct symbol timing is important.
* How AWGN affects digital communication.
* How SNR influences received signal quality.
* How an eye diagram helps evaluate a communication system.
* How Python can be used to simulate communication systems without a dedicated communication toolbox.

---

# 📌 Applications

The concepts demonstrated in this project are relevant to:

* Digital communication systems
* Wireless communication
* Software-defined radio
* Modems
* Satellite communication
* Optical communication
* Cellular communication
* Digital signal processing
* Communication-system simulation

---

# 👨‍💻 Author

**Rakesh Karmakar**

Electronics and Communication Engineering

Interested in:

* Embedded Systems
* Digital Communication
* Digital Signal Processing
* Electronics
* IoT
* AI-integrated Embedded Systems

---

# ⭐ If You Found This Project Useful

If this project helped you understand BPSK, RRC filtering, matched filtering, or eye diagrams, consider giving the repository a ⭐ on GitHub.

---

## 📄 License

This project is available for educational and academic purposes.
