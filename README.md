# Phase-Locked-Loop (PLL) Design-using-SKY130nm-Technology
2-Day Workshop on 18th September 2021 and 19th September 2021

![coverpic](https://user-images.githubusercontent.com/45798709/133917253-d670da75-f738-49af-b574-6175c425841f.png)

## Overview
The two day schedule of workshop on designing Phase Locked Loop (PLL) using SKY130nm Technology has:

Day 1 : PLL Theory and Lab setup

Day 2 : PLL Labs and post-layout simulations

## Table of contents:
-  [Day 1 – PLL Theory and Lab setup](#Pll-lab)
    + [Introduction to PLL](#Introduction--to-PLL)
    + [Introduction to Phase Frequency Detector](#Introduction-to-Phase-Frequency-Detector)
    + [Introduction to Charge Pump](#Introduction-to-Charge-Pump)
    + [Introduction to VCO and Frequency Divider](#Introduction-to-VCO-and-Frequency-Divider)
    + [Tool setup and design flow](#Tool-setup-and-design-flow)
    + [Introduction to PDK, specifications and pre-layout circuits](#Introduction-to-PDK-specifications-and-pre-layout-circuits)
    + [Circuit design simulation tool - Ngspice Setup](#Circuit-design-simulation-tool-Ngspice-Setup)
    + [Layout design tool - Magic Setup](#Layout-design-tool-Magic-Setup)
   
-  [DAY 2 :PLL Labs and post-layout simulations ](#PLL-Labs-and-post-layout-simulations)
     +[PLL components circuit design](#PLL-components-circuit-design)
    + [PLL components circuit simulations](#PLL-components-circuit-simulations)
    + [Steps to combine PLL sub-circuits and PLL full design simulation](#Steps-to-combine-PLL-sub-circuits-and-PLL-full-design-simulation)
    + [Troubleshooting steps](#troubleshooting)
    + [Layout design](#Layout-design)
    + [Layout Walkthrough](#Layout-Walkthrough)
    + [Parasitics extraction](#Parasitics-extraction)
    + [Post Layout simulations](#Post-Layout-simulations)
    + [Steps to combine layouts](#Steps-to-combine-layouts)
    + [Tapeout theory](#Tapeout-theory)
    + [Tapeout labs](#Tapeout-labs)

- [Acknowledgement](#ack)
 


# DAY 1 - PLL Theory and Lab setup

The day 1 of the workshop covers about the basics of PLL, different blocks present in PLL and their functionalities and steps to install ngspice and magic for designing of PLL.
The day 1 has the following content

### Introduction to PLL:

Why PLL?
PLL is used to get a precise CLK aignal without frequency and phase noise and it has flexibilty to get a desirable clk_frequecy that we want to run. 

Phase locked loop called as PLL  is a feedback system which generates a high frequency , low noise clock signal.

Crystal Oscillators like quartz crystal and Voltage controlled oscillators  can be used to generate clock signal.

Crystal oscillators are the pure spectrum they are free from unwanted frequencies. The drawback of crystal oscillators is that they are operated at fixed frequecny and have very limited tuning range and not as flexible as VCO. Hence, Voltage controlled oscillators (VCOs) are used which can generate high frequency clock, but it has poor phase noise. To get a low phase noise, high frequency clock, we use phase locked loop which makes the VCO mimic the crystal to achieve low phase noise, while still getting high frequency clock.

VCO's can be implemented onchip easily and we an able to control the output frequency of VCO by changing the input voltage.However they dont have pure specturm just like quartz, they tend to have noise fluctuations in their phase. 

PLL purpose is to mimic with the reference signal maintaing a constant phase with the reference signal and a frequency same or multiple of reference signal.

The block diagram of PLL is as shown below:

![Pll_components](https://user-images.githubusercontent.com/45798709/133923207-0fccf03f-78e8-4785-8fe9-e4a494591ed5.png)


### Introduction to Phase Frequency Detector(PFD) :

It detects the phase (and frequency) error between the input reference clock IN (that is, crystal oscillator) and feedback clock FB. It generates two output signals, UP and DN, whose pulse width is proportional to the phase error.

For, detecting the phase error between the input reference clock and the feedback clock, we donot use XOR phase detector as it doesnt correct the frequency errors. Instead, we use Phase Frequency Detector , which will correct both, phase and frequency differences between the input reference clock and the feedback clock.

When falling edge of reference clock leads feedback clock, the UP signal goes high and DN signal remains low. UP signal remains high until it detects the falling edge of the feedback clock. As soon as feedback clock falling edge arrives, UP signal goes low. 

When falling edge of feedback clock leads reference clock, the DN signal goes high and UP signal remains low. DN signal remains high until it detects the falling edge of the reference clock. As soon as reference clock falling edge arrives, DN signal goes low. 

Dead zone is one of the problem of PFD. The phase difference below which PFD output is not able to reach the desired logical level and fails to turn on the charge pump switches is called the dead zone. Above precision PFDs are present, which overcomes this issues.

### Introduction to Charge Pump:

The loop filer takes input as voltage/current. Hence, the pulse width modulates signal at the output of PFD have to be transformed to volateg/current. Therefore, we use charge pump circuit. 

It generates the error current (ICP) which is proportional to the phase difference od pulse width between UP and DN which inturn is proportional to the input phase error. 

  Charge pump chagres or discharges the loop filter based upon the UP and DN signals. If UP signal is high, charge pump charges the loop filter. If DN signal is high, charge pump discharges the loop filter.
  
  Charge pump circuit have of charge leakage, which should be reduced as much as possible.

Loop Filter (LF):

It converts the error current into error voltage. It also controls the loop dynamics and stability of the system. 

For the stability purpose, C2 capacitor should be one-tenth of the C1 capacitor. Loop filter bandwidth should be one-tenth of the highes output frequency.

### Introduction to VCO and Frequency Divider:

### Voltage controlled oscilloscope:

VCO is the heart of PLL and it generates the desired high frequency clock output. 

The most common on-chip VCO used is the ring oscillators which consists of odd number of inverters. The period will be twice the number of inverter delays multiplied be the inverter delay. One of the way, to control the frequency of the ring oscillator is to vary the current using the current starving technique. It is important to note that, we should design the VCO is such a way that the range of frequencies we want for the PLL is generated properly by the VCO.

### Frequency Divider (N)

It divides the VCO clock to generate the feedback clock with same frequency as input reference clock.

A toggle flip flop generates the clock which has twice the time period of the input clock given to it, that is we obtain the clock having half the frequency of the input clock provided to the frequency divider. For 8x clock multiplier, we should divide the output clock by 8 to generate feedback clock. For obtaining one-eigth of frequency, we should cascade three toggle flip flops. 

### Important terms used in PLL

1. Lock-in range

Range of freuencies which PLL can stay in lock in steady state. It mainly depends on the range of frequencies VCO can produce and is limited by the dead zone of PFD.

2. Capture range 

Range of frequencies which PLL can lock when started from unlocked condition. Capture range is smaller than the lock-in range. It depends on the loop filter bandwidth.

3. Settling time

The time taken by the PLL to lock from an unlocked state is called the settling time. It depends on how quickly charge pump rises to the stable value.

## Tool setup and design flow:

The design tools used for the workshop for the design of PLL are

1. Ngspice : for transistor level circuit simulation

2. Magic : for layout design and parasitic extraction.
 
 These tools are completely open source and free, right now for the design and simulation of PLL we are using sky130nm PDK(Process design kit) where the minimum channel length should be 130nm

### Steps to access the tools:

From the ubuntu package , install Ngspice directly.  
For Magic we will compile it with the source code since  we need the latest version of Magic to support Skywater130nm technology node.

Ngspice: ngsice<circuit_file_name> 

This command is used to run the ngspice files , ngpspice files have .cir extension

Magic: magic - T <Technology_file_from_PDK><the_layout_file_to_open> 

The above command shows how to open a magic file for a particular technology node

It opens the layout file, where we can view the layout  and make modifications to it. Magic has many features, out of which we will be using parasitic extraction and gds features.

### Design Flow:

With the PLL specifications, we will perform SPICE-level circuit development and pre-layout simulation for each block of PLL. After the pre-layout simulation we develop layout.  After layout, we perform parasitic extraction in this step we will observe the parasitic components like R and C elements that are present with the PLL that effects delay in output response. After this, we generate spice netlist then we do the post simulation which is more accurate than thr pre-layout simulation.
Post-layout simulation results are compared with pre-layout simulation results, since the effect of paraisties are considered there might be small change in the signal delays and phase that needs to be observed and combined layout of all the blocks are integrated to genrate GDS file and then tesed and integrated with efabless caraval SOC.

## Circuit design simulation tool - Ngspice Setup

We use the following command to isntall Ngspice:

sudo apt-get install ngspice

Then we fetch the sky130 library which consists of the transistor models which is needed for the simulation.
## Layout design tool - Magic Setup

We will use the source code from opendesigncircuit.com. We can do the git clone and intall the Magic. Then, we should compile it, for that we should install the dependencies form the opendesigncircuit.com. Then we run the ./configure command. The we run the make command. Then we run the command

sudo make install

Then we should install the sky130nm technology information which Magic needs for desiging the layout.



# DAY 2 :PLL Labs and post-layout simulations 

## PLL Specifications

Corner = TT(Typical-Typical)

Supply voltage = 1.8V

Temperature = Room temperature

Input Fmin= 5MHz , Fmax= 12.5MHz

Multiplier = 8x

Jitter (RMS) < 20ns

Duty cycle = 50%
## PLL components circuit design

A spice file is a text file with .cir extenstion.

For example, if we want to write code for frequency divider circuit, we write the following command in terminal:

touch FD.cir

and then page will be created where we should write the code for the desired circuit. Don't forget to include the SPICE library file in it.

Similarly we do it for all the blocks of PLL.

After creating the .cir files we run the Ngspice simulation.

There are different blocks that are need to be simulated with in the PLL first we need to design the blocks of PLL and write spice code for that and then simulate the blocks using ngspice

### Phase Frequency Detector circuit(PFD) 
![pfd_ckt](https://user-images.githubusercontent.com/45798709/133927950-9099f15f-1323-4964-832e-28367853a883.png)

The phase frequency detector is designed using nmos and pmos for comparing the vco output signal frequency and phase with the reference clk frequency in order to observe the deviation of vco output with the refernece signal, so that if deviation is high we can reduce that by proper frequency multipilying operations that is why this PLL is also called as clk multiplier PLL.
The connections among  the mosfets are written and described in the spice code of PFD and then we will simulate the file PFD.cir

### charge pump
![cp_ckt](https://user-images.githubusercontent.com/45798709/133928175-369fb15a-c94e-4f5c-8048-96b2b293620d.png)



Charge pump converts the digital comparision of vref and output of frequency detector  into a analog signal with a frequeny range of the signal matching with vco signal frequency range

### Voltage controlled oscilloscope:

![vco_ckt](https://user-images.githubusercontent.com/45798709/133928319-5ca7acf0-afdb-4a54-9978-3021c24cf3d9.png)


### Frequency divider:


![fd_ckt](https://user-images.githubusercontent.com/45798709/133928369-f5e60f36-d5dd-485a-96a9-43b629ff2364.png)



Frequency divider is used for maintaing the flexibity of the PLL so that PLL can operate at different frequencies, Frequency divider makes sure that output signal frequency matches with vref frequency, if the frequency is divided in the frequency divider then the same division factor will be multiplied to the vref  signal frequency in order to maintain minimum phase error.

# PLL components circuit simulations

### Freqency divider Output:

The command for running Frequency divider is ngspice FD.cir
![fd_terminal](https://user-images.githubusercontent.com/45798709/133929138-f8a714b0-f1c9-43dd-b71e-7c9f9414f1cd.png)



![fd_out](https://user-images.githubusercontent.com/45798709/133928841-f2f1ae81-311e-48a5-ae5a-fc4ad908eab7.png)



### Charge pump Output:

The command for running charge pump is ngspice CP.cir

![cp_terminal](https://user-images.githubusercontent.com/45798709/133929179-3bf12cd3-6b81-438f-9b24-3d55894dea8c.png)

CP  node Voltage values:

![cp_pulse_terminal _valuespng](https://user-images.githubusercontent.com/45798709/133929224-9b27e817-a717-4248-9e26-b960d313b491.png)



![cp_output](https://user-images.githubusercontent.com/45798709/133928859-fa8d7d2c-1d17-41dc-8f1c-e188055dfd4e.png)


### Charge pump output with Up signal as a Pulse signal:
![cp_pulse_Output](https://user-images.githubusercontent.com/45798709/133928881-6aa2b5d3-0a28-4f82-8afd-8f7f26c5fbb2.png)

### VCO Output:


The command forVCO is ngspice VCO.cir

![vco_terminal](https://user-images.githubusercontent.com/45798709/133929298-94b2307e-4116-401d-b0f1-59a4277795ed.png)

VCO node voltage values :

![vco_values](https://user-images.githubusercontent.com/45798709/133929319-a43a2893-8b92-4e48-9d69-ed88e8bc35de.png)





![vcout](https://user-images.githubusercontent.com/45798709/133928920-154d3fa3-3d06-45e8-8f3a-fa2fe389e34e.png)

### Phase Frequency Detector Output:


![pfd_out](https://user-images.githubusercontent.com/45798709/133928955-2d70f4d3-e6d3-4ad2-8c99-d5a03db001d8.png)

### Phase Frequency Detector Output with 1ns delay:

![pfd_out_1ns](https://user-images.githubusercontent.com/45798709/133928981-df115367-03a0-42e1-aa68-770f713b2e34.png)

### Phase detector output 1:

![PD_output](https://user-images.githubusercontent.com/45798709/133929009-06bad90c-3aa1-4111-b51f-b5a777f76781.png)


### Phase detector closeup output :
![PD_output _2png](https://user-images.githubusercontent.com/45798709/133929026-8819ac13-c562-47fb-b0b9-54781cf50a95.png)

# Steps to combine PLL sub-circuits and PLL full design simulation:

All the sub-ciruits are integrated with in a single PLL module file by declaring modules to each sub-cicuit and calling each individual module separately acoording to their functionality with in the PLL circuit spice file.

![pll_prelayout_terminal](https://user-images.githubusercontent.com/45798709/133931018-9b10313c-cd7a-4679-9d00-ff0ac8c489f8.png)

### Troubleshooting steps:

If output doesn't match or mimic properly, first is to observe what kinds of issues you are facing. Always try to debug individual circuits rather than simulating the whole circuit.

If signals are coming flat up or the simulation is crashing then check whether the connectivity is done properly or any issues like wrong net names, capitaliszation issues or parameter value issues. 

If the signals are coming as expected but mimicing of signal is not happening then verify the following:

1. VCO is working within the required range or not.

2. Whether the PFD is able to detect small phase differences or not.

3. How is the response of charge pump? Is it fast or slow. Too much fluctuations in charging or discharging, then capacitor sizing is the thing where we have to pay the attention to. Also, check if there is charge leakage. If the charge pump is charging when the input is zero, then there is charge leakage issue.

4. If nothing works out, then try adjusting the loop filter by using the thumb rul

























