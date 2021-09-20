# PLL-Design-Using-Google-Sky-130nm
A Phase Locked Loop an Analog IP (Intellectual Property) that is used to generate a clock signal for digital synchronous circuit systems. This repo walks through the end-to-end design of a Phase Locked Loop, designed on the Skywater 130nm technology node. The design, simulation and layout of this PLL have been carried out using Open Source tools such as Ngspice (for simulations), Magic (for layout) and the Sky130 PDK (Process Design Kit for the transistor specifications).

# Contents
- [PLL-Design-Using-Google-Sky-130nm](#pll-design-using-google-sky-130nm)
- [An Overview of the Design Procedure](#an-overview-of-the-design-procedure)
- [PLL Components and Working](#pll-components-and-working)
  * [Phase Frequency Detector (PFD)](#phase-frequency-detector--pfd-)
  * [Charge Pump](#charge-pump)
  * [Voltage Controlled Oscillator (VCO)](#voltage-controlled-oscillator--vco-)
  * [Frequency Divider](#frequency-divider)
- [Figures of Merit of the Phase Locked Loop](#Figures-of-Merit-of-the-Phase-Locked-Loop)
  * [Lock Range](#lock-range)
  * [Capture Range](#capture-range)
  * [Settling Time](#settling-time)
- [Experiments](#experiments)
  * [Simulations & Layout](#simulations---layout)
  * [Layout Design](#layout-design)
  * [Post Layout Simulations](#post-layout-simulations)
- [Final Integration and Tapeout](#final-integration-and-tapeout)
- [References](#references)


# An Overview of the Design Procedure
There are several steps that have to be carried out from the inception of a design till its layout to obtain a precisely functioning IP. The following points discuss the important steps that have been carried out for the PLL.
1. Simulations: Different components of a large circuit system like a PLL have to be simulated first. Simulations make use of specifications given in the PDK to make copies of transistors. While performing simulations, transistor sizing, voltages and currents in each sub-circuit have to be adjusted to get the desired output. Simulations help in checking the overall functionality of the circuit. In this project, simulations have been carried out using Ngspice.
2. Layout: Layout refers to constructing a schematic diagram of different layers of transistors and interconnecting components. Once successful simulations of the IC have been carried out, the layout or the mask is drawn. The nature of materials that make up transistors and interconnects introduce delays and other material-dependent aberrations in the circuit. In this project, the IC layout has been drawn using Magic. Design Rule Checking is a method to verify whether a particular design is compliant with the restrictions posed by the process technology. Layout helps in executing DRC and extracts the necessary parasitics associated with the design.
3. Post-Layout Simulations: Once the parasitics have been extracted, they are appended to the SPICE file created pre-layout and simulated again. This gives a more realistic picture of the functioning of the IP, complete with the inclusion of all delays and parasitics. After the post-layout simulations have been verified, the circuit can be sent to foundries for tapeout.

Figure 1: A Flow-Chart Illustrating the Design FLow

![flowchart](https://user-images.githubusercontent.com/90972284/133916350-71b24f85-c8ca-4437-8228-a060fa6b344d.png)

# PLL Components and Working
A PLL generates a precise and pure clock signal, i.e., a signal that oscillates between a high and low voltage at a specified frequency. Clock Signals can be generated in two ways. 
1. Using Quartz Crystals: Upon passing mechanical stress across the faces of a Quartz crystal, an electric voltage that oscillates at a given frequency is obtained as a result of the piezoelectric nature of the crystal. The frequency spectrum of this signal is pure, and there exist no spikes at any unwanted frequency. However, a pure quartz crystal oscillator suffers a limitation of being restricted to only a single frequency, and cannot be tuned. 
2. Voltage Controlled Oscillators: This is a circuit component that can be implemented on-chip. It allows good control over the spectral purity and frequency and offers the additional feature of tunability. 

PLLs are used to make the VCO mimic the spectral purity of a Quartz oscillator while maintaining flexibility. 

Figure 2: Frequency Spectrum of (a) Crystal Oscillator and (b) VCO

![freqspec](https://user-images.githubusercontent.com/90972284/133916808-8d81438e-ae1e-4755-9821-452a887f4b21.png)

The mechanism by which the PLL makes a VCO oscillate is by comparing the signal generated by a VCO to a reference signal and adjusting the error to ensure the VCO output mimics the reference signal. Mimicking implies that the output generated differs from the input by a perfectly controlled frequency or phase amount. The reference signal could be generated by a Quartz crystal. The working of the PLL above can be illustrated using a control system. 

Figure 3: Control System Illustration of the working of a PLL

![controlsystem](https://user-images.githubusercontent.com/90972284/133916993-d5fec7f3-f1a6-4dbd-8a50-837d86ffe250.png)

The working of the different components of this control system is discussed below.

## Phase Frequency Detector (PFD)
The PFD compares the feedback signal with the reference signal. There are two possibilities. 

a. the output lags the reference 

b. the output leads the reference

For these distinct cases, the preceding rising or falling edges can be detected using a Flip-Flop. Two flip-flops are used, one each for each signal and their outputs are tied using an AND gate to their respective reset CLR pins. The state diagram of the above circuit is shown below.

Figure 4: State Diagram Explaining the Incorporation of Up and Down signals

![statediag](https://user-images.githubusercontent.com/90972284/133917218-e34b0138-ef65-4455-8099-9c57d0153225.png)

A drawback in this circuit would occur if the two signals had a very small delay. This is called the dead-zone, and it prevents improving very small delays (<1ns). The only way to compensate for this would be to design a more sensitive PFD.

## Charge Pump
A charge pump converts the digital measure of the difference in phase or frequency into an analog signal that is fed into the VCO. Charge pumps are implemented using current steering circuits. 

Figure 5: Current Steering Circuit

![currentster](https://user-images.githubusercontent.com/90972284/133917425-17e6635b-794d-4b83-b5a7-d9b649c8972c.png)

The reluctance of the capacitance to change the voltage across it immediately smoothens or averages the voltage. A limitation in this circuit would be the presence of leakage currents even when both transistors are off. Additionally, there may be high-frequency variations at the output of the charge steering circuit. This can be eradicated using a Low-Pass Filter (LPF).
Some rules for picking capacitances:

a. C = C_LPF/10

b. Loop Filter Bandwidth = (Highest output frequency of PLL)/10

## Voltage Controlled Oscillator (VCO)
A VCO is implemented using a Ring Oscillator, which is a series of an odd number of inverters with a specific delay. The time period of the ring oscillator is given by
P = 2(delay of each inverter)(inverter count).
The frequency depends on delay and delay depends on the current supplied. For a larger current supplied, the output gets charged faster. A current starving mechanism is used to control the oscillation frequency. The range of frequencies that the VCO can produce must be in agreement with the frequencies required out of the overall PLL.

Figure 6: Image of a Ring Oscillator (Source: Wikipedia)

![vco](https://user-images.githubusercontent.com/90972284/133918912-b482b1eb-35d4-4c93-aa01-1198ea72416b.jpg)


## Frequency Divider
The frequency divider circuit is designed using a toggling flip-flop. The frequency obtained can be divided by different factors depending on the number of inverters used in the back connecting the output to the input. 

Figure 7: Frequency Divider Circuit

![freq_divider jpg](https://user-images.githubusercontent.com/90972284/133924806-eb595830-1cba-4850-8131-bc30c482fa16.png)


# Figures of Merit of the Phase Locked Loop
## Lock Range
The range of frequencies for which PLL maintains its lock once it has already been locked is called the Lock Range. This parameter is limited by the dead zone. 

## Capture Range
The range of frequencies for which the PLL maintains the lock once it moves from the unlock to the locked zone. Usually, the capture range is smaller than the lock range. 

## Settling Time 
The time within which the PLL is able to lock in from an unlocked state is called the Settling Time of the PLL.

# Experiments
Some standard parameters are considered while making the SPICE files. 

1. VDD = 1.8V
2. T = 27 Degrees Celcius
3. Reference Clock = 5 to 12.5 MHz
4. Output Clock = 40 to 100 MHz (the Frequency Divider Circuit makes use of 3 inverters, which would result in the overall frequency multiplication by 8 times).

Transient analyses for specific time intervals are carried for different circuit components and then the PLL as a whole. Parameters such as jitter and capacitive loading are omitted to maintain simplicity in this particular design. 

## Pre-Layout Simulations
The first step to simulating the PLL is to simulate and check the working of each of the individual components of the PLL control system.
1. Phase Difference Detection

    Illustrated below is a pre-layout simulation of the Phase Different Detection circuit.
    
![image](https://user-images.githubusercontent.com/90972284/133928275-137b46ff-a0f8-49a4-a777-df77c2597e97.png)
    
![pd](https://user-images.githubusercontent.com/90972284/133919934-34c13a79-48e8-41de-b630-edd636429003.jpg)

As seen from the image, the Down signal is activated using the AND operation illustrated in the schematic.

2. Charge Pump

    Illustrated below is a pre-layout simulation of the Charge Pump circuit. Initially, it is assumed that there is no charge or voltage built up across the capacitor. 

![image](https://user-images.githubusercontent.com/90972284/133928420-4d48be3f-1252-4af4-b721-2b52a859eb97.png)

![charge_pump](https://user-images.githubusercontent.com/90972284/133920005-b15162bb-ef90-4a97-8b6c-6adfb79afc88.jpg)

As seen from the image, the voltage across the capacitor is steadily rising. Additionally, the fluctuations in the voltage as it rises are observed, which correspond to the rising and falling of the Up and Down signals.

3. VCO

    Illustrated below is a pre-layout simulation of the VCO circuit. 
    
![image](https://user-images.githubusercontent.com/90972284/133928476-cde10e1a-1b9e-444c-b62b-08350b9f5328.png)


![3stagevco](https://user-images.githubusercontent.com/90972284/133927729-c4eda51b-8326-4ee0-a64c-5f056daa30fe.jpg)

As seen from the image oscillations have been successfully generated. The oscillations are full-swing, and this is due to the presence of an additional inverter at the output. 

4. Frequency Divider

    Illustrated below is a pre-layout simulation of the Frequency Divider circuit.
    
 ![image](https://user-images.githubusercontent.com/90972284/133928562-eddba974-f2a4-4fc6-984d-356fc8ced3f4.png)
    
 ![freq_divider](https://user-images.githubusercontent.com/90972284/133920049-0e7541dc-b3a7-4f4e-bd99-dbe30245784c.jpg)
 
 As seen from the image, the frequency of the output is getting divided by half as compared to the input. 
 
 Now that the individual components of the circuit are functioning properly, they can be connected to each other appropriately to evaluate the working of the whole PLL.
 
 5. PLL 
 
    Different components simulated above are combined on spice using the ```.subckt``` command. Illustrated below is the pre-layout simulation of the PLL.
    
 ![image](https://user-images.githubusercontent.com/90972284/133928714-dd31d9fa-1245-49f5-9fb3-065db36de2fb.png)

    
 ![pll_imp](https://user-images.githubusercontent.com/90972284/133920281-12cd642f-a976-4f9c-b394-525f68748135.jpg)
 
As seen from the above image, the PLL output follows the reference signal exactly and suffers a delay of fewer than 0.5 ns. This proves that a high degree of match with the reference signal has been successfully obtained by the PLL.

Now that the pre-layout simulation of the PLL has been completed and no errors in operation have been observed, the next step to achieving the design goals (layout) can be carried out.

## Layout Design

Different colours in the layout indicate different materials used to build different lithographical layers in the semiconductor well.

1. PFD

    Given below is the layout drawing of the Phase Difference Detecting Circuit. 
    
![image](https://user-images.githubusercontent.com/90972284/133928811-8984f05c-0ecb-46b8-b780-d58d1cdf6260.png)

![PFD_Layout](https://user-images.githubusercontent.com/90972284/133919360-6c598f4e-5366-4410-95ff-29e91c077cc4.jpg)

Total Area of the PFD - 49.1 umsq.

2. Charge Pump

    Given below is the layout drawing of the Charge Pump Circuit. 
    
![image](https://user-images.githubusercontent.com/90972284/133929047-24cf9b76-605c-45ae-bfcd-b5547dec0a92.png)

![CP_Layout](https://user-images.githubusercontent.com/90972284/133922736-6d7ff00b-9e6b-4778-83d9-29d749665bef.jpg)

Total Area of the Charge Pump - 132.3 umsq.

3. Voltage Controlled Oscillator

    Given below is the layout drawing of the VCO Circuit. 
    
![image](https://user-images.githubusercontent.com/90972284/133928905-09eda42a-bc10-4450-84b8-eb268e781d7d.png)
    
![VCO_Layout](https://user-images.githubusercontent.com/90972284/133923219-914bdef6-33f7-4acb-a663-912e54fc0d66.jpg)

Total Area of the Charge Pump - 57.7 umsq.

4. PLL
    Given below is the layout drawing of the entire PLL Circuit. 
    
![image](https://user-images.githubusercontent.com/90972284/133929099-d135f90d-d38c-438e-9a21-ba67d59aa106.png)
    
![PLL_Layout](https://user-images.githubusercontent.com/90972284/133923751-20a10bdb-cb5a-4f70-9720-494cf6e9f4b9.jpg)

Total Area of the PLL - 493 umsq.

## Post Layout Simulations

Parasitics are extracted from the layout of the PLL circuit. This file is then converted to a SPICE and then simulated. 
For study purposes, the post-layout simulation has been carried out with a 10ns phase delay and a 1ns phase delay.

Extraction is carried out using the following code.
```
extract all %extracts all parasitics from the layout
ext2spice cthresh 0 rthresh 0 
% the previous command is basically setting the threshold or minimum value of R and C that counts as a parasitic
ext2spice %conversion to SPICE
```

a. 10ns Delay

![postlayout](https://user-images.githubusercontent.com/90972284/133923960-4c7c139d-cba8-42ae-be1f-33f714e10a8b.jpg)

![postlyout2](https://user-images.githubusercontent.com/90972284/133923972-087a2537-a7dc-45f6-ac0a-191b97777837.jpg)

As seen from the above images, the PLL is able to successfully generate the clock signal.

b. 1ns Delay

![postlayout10n1](https://user-images.githubusercontent.com/90972284/133924091-c355269c-d139-425c-8025-a97558786774.jpg)

![postlayout10n2](https://user-images.githubusercontent.com/90972284/133924095-8c70ee64-fc35-4fd1-a4cc-3be1aba9373b.jpg)

As seen from the above images, the PLL is also able to successfully generate the clock signal. Even with a small lag of 1ns, the circuit is successfully functioning.

# Final Integration and Tapeout
There are certain important intermediate steps that have to be carried out before tapeout so that the newly created IP can interface with the external hardware components easily. Some examples are:
1. I/O Padding - Since the designed IP would be sized in the order of micrometres, it cannot be connected to other hardware components using wires. Dedicated I/O pins have to be specified for the IP.
2. Peripherals: In order to have serial connectivity, peripherals associated with serial communication protocols must be integrated with existing designs. 
3. Memory: If some kind of memory devices are required to interact and interface with the IP, dedicated procedures have to be followed to accommodate for their operation.

Ensuring that the IP meets all of the above criteria is a tedious task, therefore, the IP is made to ride on the Caravel SoC Vehicle by Efabless.
The IP is placed and routed inside the container. PnR can be completed either manually or using tools such as OpenLane. Once the PnR has been completed, the verification of the SoC's connectivity is carried out. This is then integrated onto the SoC and goes through another round of verification to ensure that the behaviours are as expected. 

# References
[1] Franco, S. (2020). *Design with Operational Amplifiers and Analog Integrated Circuits*.

[2] *Analog integrated Circuits by Dr Shouri Chatterjee*. YouTube. Retrieved September 19, 2021, from https://www.youtube.com/playlist?list=PLpiNwuPPfrOlH9S06zLFxbcnVNz5ogDUf. 

[3] Razavi, Behzad. *Design of Analog CMOS Integrated Circuits*.
