# Phase-Locked Loop IC design using Open-Source PDK
<img src = "PLL-Workshop-Banner_efabless.png">
This repository reflects the work done in the PHASE-LOCKED LOOP (PLL) IC Design using Open-Source PDK workshop, offered by VLSI System Design Corp. Pvt. Ltd in collaboration with Lakshmi Sathi. It is a 2-day workshop aiming to familiarise ourselves with open source tools, PDKs, Analog Design, and TapeOut Process under efabless.com. Furthermore, if you are a beginner, and visiting this GitHub repository, you can follow these steps to Design and Layout your own PLL IC.

## Taking the First Step
The beauty of open-source is that it is free and available to everyone. In the whole workshop, I have used only two tools and one PDK. Ngspice is an open-source mixed-signal electronic circuit simulator used in circuit design, pre-layout, and post-layout simulation. Magic VLSI Layout Tool is an open-source circuit layout editor, used in circuit layout. Google SkyWater PDK is an open-source process design kit, used in physical design. To install these open-source tools, I would suggest you go to their websites:-
  1. [Ngspice](http://ngspice.sourceforge.net/)
  2. [Magic VLSI Layout Tool](http://opencircuitdesign.com/magic/)
  3. [Google/Skywater 130nm PDK](https://github.com/google/skywater-pdk)

IT IS RECOMMENDED TO USE LINUX-BASED OS RATHER THAN ANY OTHER KERNEL-BASED OS.

## Table of Contents
- [About](#about)
- [DAY 1: PLL Theory and Tools setup](#day-1--pll-theory-and-tools-setup)
  - [Introduction to PLL](#introduction-to-pll)
  - [Introduction to Phase Frequency Detector](#introduction-to-phase-frequency-detector)
  - [Introduction to Charge Pump](#introduction-to-charge-pump)
  - [Introduction to VCO and Frequency Divider](#introduction-to-vco-and-frequency-divider)
  - [Tool setup and design flow](#tool-setup-and-design-flow)
  - [Introduction to PDK, specifications and pre-layout circuits](#introduction-to-pdk,-specifications-and-pre-layout-circuits)
  - [Circuit design simulation tool - Ngspice Setup](#circuit-design-simulation-tool---ngspice-setup)
  - [Layout design tool - Magic Setup](#layout-design-tool--magic-setup)
- [DAY 2: PLL Design and post-layout simulations](#day-2--pll-design-and-post---layout-simulations)
  - [PLL components circuit design](#pll-components-circuit-design)
  - [PLL components circuit simulations](#pll-components-circuit-simulations)
  - [Steps to combine PLL sub-circuits and PLL full design simulation](#steps-to-combine-pll-sub---circuits-and-pll-full-design-simulation)
  - [Troubleshooting steps](#troubleshooting-steps)
  - [Layout design](#layout-design)
  - [Layout Walkthrough](#layout-walkthrough)
  - [Parasitics extraction](#parasitics-extraction)
  - [Post Layout simulations](#post-layout-simulations)
  - [Steps to combine layouts](#steps-to-combine-layouts)
  - [Tapeout theory](#tapeout-theory)
  - [Tapeout labs](#tapeout-labs)
- [Future Scope](#future-scope)
- [Extra Reference Material](#extra-reference-material)
- [Acknowledgements](#acknowledgements)


## *****************************************************************************


## About
This workshop presents a basic overview of Phase-Locked Loop IC design using Open-Source Google-Skywater 130nm node, using an intuitive approach to designing a simple PLL with little math and without diving into complex frequency domain analysis and control system theory. Starting with basic PLL concepts, this overview spans all the steps in the IC design flow - circuit design, simulation, layout, parasitics extraction, post-layout simulation and finally, it also briefly includes the use of the latest caravel harness to make tape-outs.

## DAY 1: PLL Theory and Tools setup
PLL is a control system implemented using an analog circuit, which is used in many applications like, FM modulation and demodulation circuits, motor speed controls and tracking filters, frequency shifting decodes for demodulation carrier frequencies, time to digital converters, Jitter reduction, skew suppression, clock recovery, to get a precise clock signal without frequency or phase noise. PLL actually mimics the reference means to have the same or a multiple of the reference frequency and a constant phase difference with it. In this workshop, our focus was to design a PLL for clock signal operations in different SoCs and ICs. PLL uses a Voltage Controlled Oscillator and gives an output that has superior spectral purity nearly equal to that of Quartz Crystal. This figure shows a PLL implemented on a chip with Raven SoC.
<img src = "ravenSoc.png">

### PLL components and their details
PLL has five main components, and they are:-
  - Phase Frequency Detector
  - Charge Pump
  - Low Pass Filter
  - Voltage Controlled Oscillator
  - Frequency Divider

<img src = "pllcomponents.png">

#### Phase Frequency Detector
It compares the phase difference between the input signal and output signal and generates a pulse signal according to it. The width of the pulse is used to determine the phase of the signal. As for width increases, phase also increases. The PFD output voltage is used to control the VCO such that the phase difference between the two inputs is held constant, making it a negative feedback system. The dead zone is one of a problem with PFD. The phase difference below which PFD output is not able to reach the desired logical level and fails to turn on the charge pump switches is called the dead zone. To tackle this, one should use precision PFDs, which overcome this issue.
<img src = "pfd.png">     <img src = "pfdfun.png">

#### Charge Pump
The PFD generates a digital signal, but VCO uses an analog signal as input. So, here comes Charge Pump in the picture.
The charge pump converts the digital signal of PFD into analog control signal which is given as input to the VCO. This can be done using the current steering circuit. When the UP signal is active the capacitor gets charged, this increases the voltage at charge pump output. When the DOWN signal is active, the capacitor gets discharged through the ground. This output voltage controls the VCO. An increase in voltage, speeds up the oscillator, while a reduction in voltage, slows down the oscillator. 


<img src = "cp.png">   <img src = "cpup.png">   <img src = "cpdown.png">   <img src = "cptran.png">

#### Low Pass Filter
The main issue with the charge pump is charge leakage, it keeps charging the capacitor even when inputs are off. This is tackled by using Low Pass Filter in the output. After that, the low-frequency dc voltage signal is sent into a dc amplifier, which boosts the signal level. After that, the VCO receives the enhanced signal. This will smoothens the output, without LPF, PLL cannot lock and mimic the ref signal. Even though, the primary function of the LPF is to maintain the stability of the system. Mathematically, only one capacitor at the output of the Charge pump makes the system unstable because if we see the frequency domain analysis of the circuit then there will be two poles in the transfer function which makes it oscillating and highly unstable. Thus an RC LPF is added at the output such that output gets stabilize. The value of Cx should be roughly around one-tenth of C. The loop filter bandwidth must be less than one-tenth of the highest output frequency.

<img src = "lpf.png">

#### Voltage Controlled Oscillator
VCO is the heart of PLL and it generates the desired high-frequency clock output. Voltage-controlled oscillators are the actual parts that produce alternating digital clock signals. The most common one is the Ring oscillator. It contains an odd number of inverters and flips the output. So, VCO can be implemented using simple inverters. It is necessary to design this VCO such that the range of output frequency we want for the PLL is within the range of frequency the VCO can produce properly. The frequency of this clock signal can be controlled by the input voltage. The frequency depends on delay and delay depends on the current supplied. The LPF output serves as a VCO control signal. Current sources are used at the top and the bottom with the Vctrl voltage to control the ring oscillator. An analog signal is produced by the VCO and its amplitude is proportional to the LPF output amplitude.

<img src = "vco.png">

#### Frequency Divider
A frequency divider is used to convert the whole system into a frequency multiplier. A PLL with a frequency divider on its feedback loop is called a clock multiplier PLL. Such a PLL can make clock signals which are multiples of the reference signals. A toggle flip flop generates the clock which has twice the time period of the input clock given to it, that is we obtain the clock having half the frequency of the input clock provided to the frequency divider. For the 8x clock multiplier, we should divide the output clock by 8 to generate a feedback clock. For obtaining one-eighth of frequency, we should cascade three toggle flip flops. When the frequency difference of each input is 0, which shows a consistent phase difference, the loop is locked. If the two signals are shifted by 180, the output voltage must be at the highest. The output voltage generated will be zero in the absence of an input signal to allow the VCO to function at a fixed frequency. This frequency is referred to as the oscillator's operating frequency.
Lock Range: The range of frequencies for which PLL can maintain lock, already in the locked state.
Capture Range:  The frequencies for which PLL is able to lock in from an unlocked state.
Settling Time: The time within which the PLL is able to lock in the form of an unlocked condition.

<img src = "freqd.png">

#### Design Flow
- Specifications of the IC
- SPICE level circuit development
- Pre-layout simulation
- Layout development
- Parasitic Extraction
- Post-layout simulation
- Tapeout: A brief Introduction

#### Secifications
- Supply voltage (VDD) = 1.8V
- Minimum Input Frequency = 5MHz
- Maximum Input Frequency = 12.5MHz
- Multiplier = 8 times
- Jitter (RMS) < 20ns
- Duty cycle = 50%








## **********************************************************************************






















NOTE:
- I have used my own Ubunutu OS to run all the SPICE Models and Layouts.
- This workshop was conducted successfully for the first time between July 31, 2021, to August 01, 2021.
