---

layout: default
title: "HomePage"
permalink: /cpu-homepage

---
{::options auto_ids="false" /}


# 8-Bit CPU Simulation

Welcome to an overview of my 8-bit CPU simulation project!


This Project was made in [Logisim Evolution](https://github.com/logisim-evolution/logisim-evolution), which is a educational tool for simulating digital logic circuits. To open and use the CPU on your own system, you will need to install Logisim Evolution and you will also need a VHDL simulator. I used [ModelSim-Intel](https://www.intel.com/content/www/us/en/software-kit/750368/modelsim-intel-fpgas-standard-edition-software-version-18-1.html), but it should work with other VHDL simulators as well.

The project is based of off Ben Eater's [Tutorial Series on Youtube ](https://www.youtube.com/watch?v=HyznrdDSSGM&ab_channel=BenEater). In the series, Ben Eater builds an 8-bit CPU on breadboards, using commonly available components. I chose not to physically build the CPU, but my simulation is still very closely based on Ben Eater's design.

To start, here is a photo of what the completed CPU looks like:

![screenshot of the CPU](/docs/assets/images/CPU_full.jpg)

Below you will find explanations on some of the components I made for this project. Not all of the components are explained so if you would like to look at those, please vist the github page where you can download the Logisim Evolution file and open it to see the various custom components I made. I tried to use as few of the prebuilt logisim components as possible, because I wanted to build the CPU myself to fully understand and learn how it functions.

# Table of Contents
* TOC
{:toc}


# Demo Video {#Demo}

Here is a video demo of the CPU:
<iframe width="560" height="315" src="https://www.youtube.com/embed/kl-4qApM0LM?si=KaeV3ZnZ6zxbw7G4" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>


# Explanation Video {#explanation-video}

# Bus  {#Bus}

The CPU is built around an 8-bit bus. This is essentially just eight wires running vertically through the CPU, and all of the various components connect to this bus to communicate with each other. It also has eight LEDs connected to the top of it to display what value is currently being outputed on the bus.

![screenshot of the bus](8bit-CPU-simulation/docs/assets/images/Bus.jpg)

# Registers  {#Register-Main}

The Registers are made up of a series of eight D flip-flops, with some control logic. There are 11 inputs, consisting of eight data inputs, aswell as a load, clock, and enable output signal. There are eight pins that output the register values at all times, and there are eight more output pins that only ouput when the enable pin is pulled high.

---
---

![screenshot of the register circuit](/assets/images/Register_Full.jpg)
![screenshot of the register appearance](/assets/images/Register_Appearance.jpg)

---
---

Below you can see a close up of one of the sections or the register. As you can see it is based around a D flip-flop, but it also has some gates at it's input. Every clock pulse, the d flip-flop either needs to take in a new value from the **In0** pin, or it needs to keep storing the same value as it was before the clock pulse. And this behaviour must be controlled by the **Load** signal. The two AND gates and the OR gate in the register act as a multiplexer and choose which value the D flip-flop should store. Then there is an LED tied to the output and a **D0** pin which constantly outputs the value stoed in the D flip-flop. Finally, the right most section of the screenshot shows an enable pin which controls a three-state buffer, that enables and disables the **Out0** pin.

![screenshot of a single part of the 8-bit register circuit](/assets/images/Register_Single.jpg)

The A, B, and Output registers all use this register component.

## Instruction Register {#Register-Instruction}

The Insruction Register is slightly different. It uses the same 8-bit register as described above, but only the four least significant ouput bits are connected to the bus. That is because the CPU only uses 4 bits for it's instructions, and therefore the instruction register only uses four out of it's eight bits. 

![screenshot of the instruction Register](/assets/images/Instruction_Register.jpg)

In that screenshot you can also see 7 boxes, with the titles **II, clk, IO, A0, A1, A2** and **A3**. These are tunnels. Logisim has a feature where, you can attach a pin to a tunnel, then create a tunnel with the same name in a different location, and then the new tunnel will also be connected to that pin. It's basically just an invisible wire that lets me clean up the look of the CPU to make it easier to use and understand. The 
**II, IO, A0, A1, A2** and **A3** tunnels in this screenshot are all connected to the Control Logic of the CPU and the **clk** tunnel is connected to the system clock.

## Flag Register {#Register-Flag}

The Flag register is similar to the Instruction register, in the sense that it is also a full 8-bit register, but this time, only two of the bits are being used. Currently, this register is connected to the **ZF** and **CF** which are connected to the zero-flag and carry-out of the ALU respectively. Then the **ZFO, CFO** and **FI** flags are connected to the CPU control logic.

![screenshot of the Flag Register](/assets/images/Flag_Register.jpg)

# ALU  {#ALU}

The ALU, or arithmetic logic unit in this CPU is essentially two 4-bit adders, chained together to create an 8-bit adder. It also has an ouput enable control pin, just like the registers do, and it has a **SU** pin which changes the operation from A+B to A-B. You can also notice that it has output pins labelled **S** and **O**. The **S** pins are controlled with the enable signal, and the **O** pins are always one. Finally, there is also a carry out pin.

![screenshot of the ALU circuit](/assets/images/ALU.jpg)

In the main CPU circuit, the eight always-on outputs of the ALU are all connected to a NOR gate, which then functions as a zero flag. That is because, for the output of that NOR gate to be a High signal, all of the inputs must be 0.

![screenshot of the ALU](/assets/images/ALU_zero.jpg)

# Control Logic  {#Control-Logic}

![screenshot of the control logic appearance](/assets/images/control_logic_app.jpg)
![screenshot of the control logic](/assets/images/control_logic.jpg)

This section is a bit more complex than the rest. It has eight inputs, including the clock signal and a Prog switch. The Prog switch is explained in the demo video, but it is also explained in the control logic VHDL video below. The clock signal is connected to a 3-bit up counter, which cycles from 0 to 5. This is used to keep track of which step in any given instruction the CPU is. The remaining **A-** inputs are the current instruction, taken from the instruction register, and the **ZFO** and **CFO** pins from the flag register. All of these feed into the VHDL control logic component and then go to various tunnels to control the CPU.

Below is a video explaining in further detail the VHDL of the control logic. I ramble on a bit at the beginning so skip to 4:35 if you only want to hear about the VHDL explanation.

<iframe width="560" height="315" src="https://www.youtube.com/embed/xp-arJalWLY?si=dzr3OnzQDiYWXpJI" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

# RAM and RAM Programmer  {#RAM}

The CPU instructions and data is stored in a 16 byte ram, and it is accessed and programmed by the RAM programmer. In this screenshot below, the top box that has "Run" and" Program" in the middle, is the RAM programmer, and the bottom box with the 3 gray and 5 red leds is the RAM itself. You can also see the Program switch and the Write memory button up in the top left corner. There is also several control signals and a selector, but all of these features are explained in the demo below.
![screenshot of the RAM and RAM programmer](/assets/images/RAM_main.jpg)
<iframe width="560" height="315" src="https://www.youtube.com/embed/ORG5ruiBKfM?si=UqSfe8LGrqjU-kwP" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

Taking a closer look at the RAM programmer, it consists of two selectors, one 4-bit and one 8-bit. Both of the selectors are controlled by the **S** pin, and this basically switches the control of the RAM from the CPU to the user. So the input pins labeled **APx** and **Dpx** are the user controlled inputs, while the switches labeled **Ax** and **Dx** are controlled by the CPU. For the CPU controlled address pins, there is also a 4-bit register, and that is used for storing which memory address the CPU wants to access.

![screenshot of the RAM programmer circuit](/assets/images/RAM_programmer.jpg)

The actual RAM is made up of 16 8-bit rams and a decoder which decodes the 4-bit address input to a 16 bit output which is then used to select which of the 8-bit RAM modules the program wants to access. Each of the 8-bit RAM modules is then made up of eight D_Latch components. 

![screenshot of the 128 bit ram circuit](/assets/images/RAM_circuit_128.jpg)
![screenshot of the 8 bit ram circuit](/assets/images/RAM_circuit.jpg)

# Program Counter  {#Program-Counter}

The program counter is basically a 4-bit counter with some added functionality. On each clock cycle, the counter counts one value up, if the CE signal is enabled and the JC signal is disabled. If the CE signal is disabled, nothing happens. If the JC signal is enabled, then the counter loads in a 4-bit value from the bus, essentially doing a jump from whatever step in the program it was currently executing, to whatever step was on the bus. Finally, there is a CO flag which controlls if the counter is outputting it's value onto the bus. This is used for accessing instructions from the RAM, using the RAM programmer.

![screenshot of the program counter](/assets/images/prog_counter.jpg)

This is all done using a combinational logic circuit I designed myself, although I am sure that it is not original since it is not very complex. It is shown below.

![screenshot of the program counter circuit](/assets/images/prog_counter_circuit.jpg)