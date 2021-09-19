# PLL-Design-Using-Google-Sky-130nm
A Phase Locked Loop an Analog IP (Intellectual Property) that is used to generate a clock signal for digital synchronous circuit systems. This repo walks through the end-to-end design of a Phase Locked Loop, designed on the Skywater 130nm technology node. The design, simulation and layout of this PLL have been carried out using Open Source tools such as Ngspice (for simulations), Magic (for layout) and the Sky130 PDK (Process Design Kit for the transistor specifications).
# An Overview of the Design Procedure
There are several steps that have to be carried out from the inception of a design till its layout to obtain a precisely functioning IP. The following points discuss the important steps that have been carried out for the PLL.
1. Simulations: Different components of a large circuit system like a PLL have to be simulated first. Simulations make use of specifications given in the PDK to make copies of transistors. While performing simulations, transistor sizing, voltages and currents in each sub-circuit have to be adjusted to get a desired output. Simulations help in checking the overall functionality of the circuit. In this project, simulations have been carried out using Ngspice.
2. Layout: Layout refers to constructing a schematic diagram of different layers of transistors and interconnecting components. Once successful simulations of the IC have been carried out, the layout or the mask is drawn. The nature of materials that make up transistors and interconnects introudce delays and other material-dependent aberrations in the circuit. In this project, the IC layout has been drawn using Magic. Design Rule Checking is a method to verify whether a particular design is compliant with the restrictions posed by the process technology. Layout helps in executing DRC and extracts the necessary parasitics associated with the design.
3. Post-Layout Simulations: Once the parasititcs have been extracted, the are appended to the SPICE file created pre-layout and simulated again. This gives a more realistic picture of the functioning of the IP, complete with the inclusion of all delays and parasitics. After the post-layout simulations have been verified, the circuit can be sent to foundries for tapeout.

Figure 1: A Flow-Chart Illustrating the Design FLow

![flowchart](https://user-images.githubusercontent.com/90972284/133916350-71b24f85-c8ca-4437-8228-a060fa6b344d.png)

# PLL Components and Working
A PLL generates a precise and pure clock signal, i.e., a circuit that oscillates between a high and low voltage at a specified frequency. Clock Signals can be generated in two ways. 
1. Using Quartz Crystals: Upon passing a mechanical stress across the faces of a Quartz crystal, an electric voltage that oscillates at a given frequency is obtained. The frequency spectrum of this signal is pure, and there exist no spikes at any unwanted frequency. However, a pure quartz crystal oscillator suffers a limitation of being restricted to only a single frequency, and cannot be tuned. 
2. Voltage Controlled Oscillators: This is a circuit component can be implemented on-chip. It allows a good control over the spectral purity and frequency, and offers the additional feature of tunability. 

PLLs are used to make the VCO mimic the spectral purity of a Quartz oscillator while maintaining flexibility. 

Figure 2: Frequency Spectrum of (a) Crystal Oscillator and (b) VCO

![freqspec](https://user-images.githubusercontent.com/90972284/133916808-8d81438e-ae1e-4755-9821-452a887f4b21.png)

The mechanism by which PLL makes a VCO oscillate is by comparing the signal generated by a VCO to a reference signal and adjusting the error to ensure the VCO output mimics tlhe reference signal. Mimicing implies that the output generated differs from the input by a perfectly controlled frequency or phase amount. The reference signal could be generated by a Quartz crystal. The working of the PLL above can be illustrated using a control system. 

Figure 3: Control System Illustration of the working of a PLL

![controlsystem](https://user-images.githubusercontent.com/90972284/133916993-d5fec7f3-f1a6-4dbd-8a50-837d86ffe250.png)

The working of the different components of this control system are discussed below.

# Phase Frequency Detector (PFD)
The PFD compares the feedback signal with the reference signal. There are two possibilities. 

a. the output lags the reference 

b. the output leads the reference

For these distinct cases, the preceeding rising or falling edges can be detected using a Flip-Flop. Two flip-flops are used, one each for each signal, and their outputs are tied using an AND gate to their respective reset CLR pins. The state diagram of the above circuit is shown below.

Figure 4: State Diagram Explaining the Incorporation of Up and Down signals

![statediag](https://user-images.githubusercontent.com/90972284/133917218-e34b0138-ef65-4455-8099-9c57d0153225.png)

A drawback in this circuit would occur if the two signals had a very small delay. This is called the dead-zone, and it prevents from imporving very small delays (<1ns). The only way to compensate for this would be to design a more sensitive PFD.

# Charge Pump
A charge pump converts the digital measure of the difference in phase or frequency into an analog signal that is fed into the VCO. Charge pumps are implemented using current steering circuits. 

Figure 5: Current Steering Circuit

![currentster](https://user-images.githubusercontent.com/90972284/133917425-17e6635b-794d-4b83-b5a7-d9b649c8972c.png)

The reluctance of the capacitance to change the voltage across it immediately smoothens or averages the voltage. A limitation in this circuit would be the presence of leakage currents even when both transistors are off. Additionally there may be high-frequency variations at the output of the charge steering circuit. This can be eradicated using a Low-Pass Filter (LPF).
Some rules for picking capacitances:

a. C = C(LPF)/10

b. Loop Filter Bandwidth = (Highest output frequency of PLL)/10

# Voltage Controlled Oscillator (VCO)
A VCO is implemented using a Ring Oscillator, which is a series of odd number of inverters with a specific delay. The time period of the ring oscillator is given by
P = 2(delay of each inverter)(inverter count).
The frequency depends on delay and delay depends on the current supplied. For a larger current supplied, the output gets charged faster. Current starving mechanism is used to control the oscillation frequency. The range of frequencies that the VCO can produce must be in agreement with the frequencies required out of the overall PLL.

Figure 6: Image of a Ring Oscillator (Source: Wikipedia)

![image](https://user-images.githubusercontent.com/90972284/133918715-0b44f8ec-d123-4fb7-bf8f-edc329c33166.png)

# Frequency Divider
The frequency divider circuit is designed using a toggling flip-flop. The frequency obtained can be divided by different factors depending on the number of inverters used in back connecting the output to the input. 
