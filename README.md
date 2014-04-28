ECE281_Lab5
===========

PRISM Computer Full Implementation


### Phase 1: Test Program

#### PRISM Implementation

Implementing the final PRISM design to an FPGA was acomplished in two septs. First implementing the PRISM controller compoents themselves using the Controller, IO, Memory, Datapath, and Top Module code provided and the ALU design created for Lab4. This phase was tested by verifying that a testbench waveform was consistent with the assembly code program provided. The second step was to implement the PRISM design on the NEXYS2 FPGA using the interfacing code proveded in Lab3 and the PRISM compugter design from the previous section. FPGA function was vefified by checking that the output matched what was prediced by the program and the testbench.

The first step was implemented fairly smoothly with no major debugging steps which necessitated reporting. The testbench waveform output is shown below and was verified by comparing the instruction register and output values to those in the PRISM program also shown below. This program essentialy loads a 8 into the accumulator then increments and  writes the value while the accumulator value is negative (8 <= accumulator <= f). The outputs, cycling from 9 to f then remaining at 0, are consistent with the provided program.

(Add entire screenshot of the waveform)

Specific analysis for each instruction is provided below. Note that the different periods for each instruction is a result of the multiple decode steps required by different instructions.

(Add lots of pretty pictures)

The second step involved programming the PRISM component so that it could be implemented in the NEXYS2 FPGA. The PRISM component was written into the top shell; however, even after all minor code bugs had been addressed, the program would freeze when implementing the jump command. 

Using the control bus outut, the instruction register was wired ot the four rightmost LED's on the FPGA's to indicate the programs process. Writing the IR to the LEDs indicated that the program preformed sastisfactoraly until it reached the jump-negative or JN command, at wich point the FPGA would freeze adn the output would be stuck on 9. The asyncronous reset, which seemed to fail without the IR output (as the 9 in the oupt was stored in a seperate register) was shown to work as the instruction sets would operate as intened on the prompt.

After consulting Captain Trimple and Dr. Neebel for help with this error, similar errors had been displayed in the seve3ral classes, it was advized that the Clock signal be removed fromt the ROM device.  This change did solve the freezing proglem, possibly due to multiple deconde cycles involved with the instruction syncronizing disfavorably with the Memory system. This change made the ROM section of the memory asycncronous; however, becuase the memory system has its own syncronous reset, the ROM acting like a mux did not change the operation of the device. 

The program was tested by observing the output cycle from 9 to f then remain in stasis at 0. Additionally the reste capability was checked to ensure that the program was restarted after the signal had been applied.
