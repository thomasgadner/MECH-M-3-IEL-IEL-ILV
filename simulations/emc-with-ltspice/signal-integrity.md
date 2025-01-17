---
marp: true
theme: beam
---

<!-- _class: title -->
# Signal Integrity and EMC in Digital Systems

---

# Signal Integrity and EMC in Digital Systems

**Even if your prototype communication link (Like USB 3.0, LVDS) appears functional, an in-depth check of signal quality is crucial to ensure a working Systems**


- **Key Reliability Factors:**
  - Distance 
  - Throughput
  - Environment

- **Risk:** Increased binary error rates can reach unacceptable levels, undermining the error correction mechanisms of the overall system.


- **IEEE Standard Compliance**
  - **Importance:** Ensures seamless interactions between members of the network.
  - **Benefit:** Adherence to IEEE recommendations improves network compatibility and stability.


---

# EMC Directives Compliance

- **Challenges:**
  - High slew rates increase the high freqency content the signals. On the other hand high slew rates are needed for fast communication interfaces.
  - Overshoots can occour if the termination of the transmission is not set properly.

- **Risk:** These issues often lead to noncompliance due to high harmonic content. 
- **Solution:** Addressing these aspects ensures compliance with electromagnetic compatibility (EMC) directives. (Reducing Slew Rates as much as Possible.)

---
# How to asses signal integrity ?

The eye diagram provides a convenient way to assess the conformity of a signal on either the transmitter or receiver side. The eye diagram is a time-based representation of the signal. This representation uses **persistence** to analyze a large number of symbols and make sure that **signal levels**, **jitter**, and **rise time** are appropriate.

- **Components:**
  - Eye opening: Indicates signal margin.
  - Rise and fall times: Represent transition quality.
  - Noise margin: Evaluates immunity to interference.

- **Interpretation:** A wide and open eye diagram signifies:
  - Low inter-symbol interference (ISI).
  - High signal integrity.

---
# jitter, signal levels, rise time

---

# Eye Diagramm



---

# How to use LT Spice to gain an understanding of signal integrity

LTspice allows simulation of the physical Hardware by playing test vectors through PWL files. These files are essential for providing robust coverage and identifying nonconformities, such as:

- **Long sequences of consecutive bits**  
- **Nonzero balanced sequences**  
- **Crosstalk from nearby transmission channels**  

Random data may require thousands of symbols to ensure specific scenarios, e.g., 11 consecutive high levels, are tested.

---
# Generating a PWL File for LTspice Test Vectors

### PWL File Format
Each line in the PWL file consists of two values:
1. **Time** (e.g., 0.1, 0.2, etc.)  
2. **Output** (voltage, current, temperature, etc.)

### Formatting Rules:
- Values are separated by **tab characters** (`\t`, ASCII code #09).  
- Each line ends with **CRLF**:  
  - **CR** (Carriage Return, ASCII code #13).  
  - **LF** (Line Feed, ASCII code #10).  

---
# Generaion of the PWL file using Python 

```python
import random

# Constants
NUMBER_OF_SAMPLES = 1000
DELAY = 6000

# Time 
transition_time = 20  # in nanoSeconds
baudrate = 115200 # Symbols per Second
symbol_time = 1_000_000_000 / 115200  # in nanoSeconds
pre_emphasis_time = 0.1  # in nanoSeconds

# Amplitude
initial_voltage = 0.0  # in Volts
voltage_high = 5.0  # in Volts
voltage_low = 0.0  # in Volts
```
---

# Generate the PWL File 

**This is the main programme for the generation of the signal pattern which can be imported into LT-Spice by means of a normal voltage source.**

```python

if __name__ == "__main__":
    with open("testV.txt", "w") as p_file:
        # Initialize
        p_file.write(f"0\t0\n")
        p_file.write(f"{DELAY}{"n"}\t{initial_voltage}\n")

        # Loop for every sample
        for sample in range(NUMBER_OF_SAMPLES):
            # Compute a sample
            logic_level = random.randint(0, 1)

            # Print sample
            if logic_level == 0:
                p_file.write(f"{(sample * symbol_time) + DELAY + transition_time:.2f}{"n"}\t{voltage_low:.2f}\n")
                p_file.write(f"{((sample + 1) * symbol_time) + DELAY - transition_time:.2f}{"n"}\t{voltage_low:.2f}\n")
            else:
                p_file.write(f"{(sample * symbol_time) + DELAY + transition_time:.2f}{"n"}\t{voltage_high:.2f}\n")
                p_file.write(f"{((sample + 1) * symbol_time) + DELAY - transition_time:.2f}{"n"}\t{voltage_high:.2f}\n")
```

---

# Example

**The lines thus created form the required test vector. For instance, if we were to attempt to simulate Rs232 or RS424,**

## Simulation in LT Spice
[PWL Program](./signal-integrity.ipynb)
[Test Vektor](./pwl_vec.txt)
[Simulation](./eye-diagram-pwl.asc)
[Plotsettings](./RS232.plt)

---

# Exercise in class 

1) **Add jitter to the program and see wath is happening to the eye diagram.**
2) **Make the simulation more realistic. Instead of using a bv source to add noise, integrate a push-pull driver, including parasitics, as you would expect from a real transceiver IC.** 

---

# Modelling microstrips and cables using the transmission line model. Ideal Transmission Line

- **Definition:** In the simplest case, the transmission network is assumed to be linear (i.e. the complex voltage across either port is proportional to the complex current flowing into it when there are no reflections), and the two ports are assumed to be interchangeable. If the transmission line is uniform along its length, then its behaviour is largely described by a two parameters called characteristic impedance,  and propagation delay.

## LTSpice Implementation:
- **Component:** `TLINE`
  - **Parameters:**
    - \( Z_0 \): Characteristic Impedance
    - \( TD \): Time Delay


---
# Example and Recap 
[Basic Idea](./transmission-lines.asc)


---
# Modelling microstrips and cables using the transmission line model. Lossy Transmission Line

- **Definition:** Models real-world transmission lines, considering losses and imperfections.
- **Features:**
  - **R:** Resistance (conductor losses)
  - **L:** Inductance (magnetic field storage)
  - **C:** Capacitance (energy between conductors)
  - **G:** Conductance (dielectric losses)

## LTSpice Implementation:
- **Component:** `LTLINE`
  - **Parameters:**
    - R, L, C, G per unit length
    - Propagation delay (TD)
    - Frequency-dependent losses
- **Use Case:** Simulating high-speed signal integrity in PCBs and cables.

---

# Example and Recap
[Ideal TL](./ideal-transmission-lines.asc)

---

# Other Sources of interfearance
### Electric-Field Coupling

Electric field coupling (also called capacitive coupling) occurs when energy is coupled from one circuit to another through an electric field. As we shall see, this is most likely to happen when the impedance of the source circuit is high.


### Magnetic Field Coupling
Magnetic field coupling (also called inductive coupling) occurs when energy is coupled from one circuit to another through a magnetic field. Since currents are the sources of magnetic fields, this is most likely to happen when the impedance of the source circuit is low.
