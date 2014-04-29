ECE281_Lab5
===========

PRISM Computer Full Implementation


### Phase 1: Test Program

#### PRISM Implementation

Implementing the final PRISM design to an FPGA was accomplished in two steps. First implementing the PRISM controller components themselves using the Controller, IO, Memory, Datapath, and Top Module code provided and the ALU design created for Lab4. This phase was tested by verifying that a testbench waveform was consistent with the assembly code program provided. The second step was to implement the PRISM design on the NEXYS2 FPGA using the interfacing code provided in Lab3 and the PRISM computer design from the previous section. FPGA function was verified by checking that the output matched what was predicted by the program and the testbench.

The first step was implemented fairly smoothly with no major debugging steps which necessitated reporting. The testbench waveform output is shown below and was verified by comparing the instruction register and output values to those in the PRISM program also shown below. This program essentially loads a 8 into the accumulator then increments and  writes the value while the accumulator value is negative (8 <= accumulator <= f). The outputs, cycling from 9 to f then remaining at 0, are consistent with the provided program.

Full Waveform
![alt text](https://raw.githubusercontent.com/IanGoodbody/ECE281_Lab5/master/FullWaveform.jpg "Full Waveform")

Specific analysis for each instruction is provided below. Note that the different periods for each instruction is a result of the multiple decode steps required by different instructions. The NOP, or No Operation, phase of the program has been omitted as the instruction was never called and because it represented the reset phase of the program when all values were set to their defaults.

LADI
![alt text](https://raw.githubusercontent.com/IanGoodbody/ECE281_Lab5/master/ladi.jpg "LADI")

The load immediate instruction has a three step, immediate execute instruction. The "Fetch" phase is outlined in red. The data containing the instruction was loaded in the previous clock cycle. The important control signals in this segment, as will be true with the rest of the fetch segments analyzed, are "irld" and "pcld." These signals are both low as the instruction has been loaded from ROM in the previous instruction. These clock cycles simply represent the controller processing the signal

The single decode step is outlined in yellow, here the data bus assumes the value 0x8 from ROM that the program is instructed to load. "pcld" is set to high indicating that the program is actively moving through the ROM addresses of the program. "accld" is set to high as the LADI command loads the data value into the accumulator. Lastly "memsel_l" is set to its active low signal as the computer is actively reading from the ROM memory.

The final immediate execute step is shown in blue. In this phase the computer is completing the current instruction while actively loading in the next instruction. The data bus has assumed 0x6 as the next instruction consistent with the "pc" bus, and the signals "irld," "pcld," and "memsel_l" are all active showing that the instruction register is loading the new value and that the program counter is progressing though the memory.  In this block the accumulator has also assumed the value 0x8 which represents the execution of the LADI instruction.

ADDI
![alt text](https://raw.githubusercontent.com/IanGoodbody/ECE281_Lab5/master/addi.jpg "ADDI")

The ADDI instruction has fetch(red) and decode (yellow) phases identical to the LADI instruction with a few minor differences in the data values. The data bus loads a 0x1 during the fetch stage and the accumulator has been incremented from it's previous value (8 => 9). The main difference occurred in the "opsel" bus where it sent a 6, compared to a 7 for LADI, to the ALU which resulted in addition operation.

OUT
![alt text](https://raw.githubusercontent.com/IanGoodbody/ECE281_Lab5/master/out.jpg "OUT")

The OUT instruction's fetch command has the standard signal arrangement; however, unlike the previous two instructions the OUT instruction has two decode cycles.

The first decode cycle has the value 3 loaded into the data path, showing where output port address and the "marlo" flag is triggered indicating that the  3 value will be stored into the LSB memory address register. PC load indicates advancement to the next memory value upon the next cycle.

The second decode cycle has 03 loaded into the address register and the "addrsel" signal set to high indicating that the address is coming from the MARs. Not ethat the "jumpsel" signal is low indicating that this address reassignment will not reset the program counter as it would in a jump and tha this address is to be used for I/O functions. as to this purpose, the "r_w" signal is set to low and the "iosel_l" signal is active corresponding to the OUT instruction's output function. The "enaccbuffer" signal is active as the OUT instruction requires that the data from accumulator be loaded to the data bus.

The direct execute step shown in blue has the control signal configuration set to load the next instruction with signals prompting the instruction register to load a new value, the program counter to advance, and the I/O system to read from memory.


JN
![alt text](https://raw.githubusercontent.com/IanGoodbody/ECE281_Lab5/master/jn.jpg "JN")

JMP
![alt text](https://raw.githubusercontent.com/IanGoodbody/ECE281_Lab5/master/jmp.jpg "JMP")

Both the Jump and Jump Negative instructions have similar control signal operations and outputs over their three decode cycles. While fetch is consistent with the other signals analyzed, the first and second decode step are unique in that they actively load values into "marhi" and "marlo." In the first two decode steps "marlold" and "marhild" are active one after another as the data values 0x2 and 0x0 are loaded from the data bus into the registers. the control signals "r_w," "memsel_l,"  and "pcld" are consistent with this purpose.

In the third decode stage the controller prepares the computer to jump by activing the "jumpsel" and "iosel" signals. Note that in the JN instruction shown "alesszero" is written as a 1 indicating that a jump will occure.

Because a jump is to occur in both waveforms shown, the execute step is identical for each. the "pc" and "addr" buses have assumed the value in the MARs as the execution of the jump command.


#### FPGA Implementation

The second step involved programming the PRISM component so that it could be implemented in the NEXYS2 FPGA. The PRISM component was written into the top shell; however, even after all minor code bugs had been addressed, the program would freeze when implementing the jump command. 

Using the control bus outut, the instruction register was wired ot the four rightmost LED's on the FPGA's to indicate the programs process. Writing the IR to the LEDs indicated that the program preformed satisfactorily until it reached the jump-negative or JN command, at which point the FPGA would freeze and the output would be stuck on 9. The asynchronous reset, which seemed to fail without the IR output (as the 9 in the output was stored in a separate register) was shown to work as the instruction sets would operate as interned on the prompt.

After consulting Captain Trimble and Dr. Neebel for help with this error, similar errors had been displayed in the seve3ral classes, it was advised that the Clock signal be removed from the ROM device.  This change did solve the freezing problem, possibly due to multiple decode cycles involved with the instruction synchronizing dis-favorably with the Memory system. This change made the ROM section of the memory asynchronous; however, because the memory system has its own asynchronous reset, the ROM acting like a mux did not change the operation of the device. 

The program was tested by observing the output cycle from 9 to f then remain in stasis at 0. Additionally the reset capability was checked to ensure that the program was restarted after the signal had been applied.

### Phase 2 Programming Assignment

#### Counter

The PRISM programming assignment provided was to create a counter that would cycle from 0 to 99 either incrementing or decrementing on each iteration depending on a user input.

The program was designed to increment the value except when certain break cases were met. The tens and ones digits were stored in separate memory locations and were to be incremented unless the value was a 9 while incrementing or a 0 while decrementing. These cases were established by adding a -9 (0x7) or a 0x0 to each digit, while incrementing and decrementing respectively, and testing if the resulting value was zero. If it was not the value was incremented, adjusting for the offset created from the subracting test operation. If the value was zero the digit was reset to a 0 or 9 depending on increment or decrement cases respectively. 

The output of the program was tested on the PRISM simulator and on the FPGA. In both cases the program demonstrated satisfactory performance.

#### Adder

Full functionality was achieved by creating a complete 4 bit unsigned adder using the PRISM language. The program was to read in two unsigned hexadecimal inputs and yield their unsigned hexadecimal sum. The main challenge of this program was to determine if a carry resulted from the PRISM add operation. This was determined by testing for generate and propagate conditions for each bit. Each bit was analyzed by rotating each bit into the MSB slot, the two MSBs were then ANDed together to determine if the bits generated a carry then, with the exception of the original LSB, ORed together to determine if the bit propagated. The result of each operation was stored into memory. 

Jump Negative instructions were used to determine the generate/propagate conditions for each bit. If a generate or propagate flag was "true" then the MSB of the stored value would be a 1. Generate bits were tested first and the propagate bits in a cascading sequence. Depending on the sequence of propagates and generates the program would either jump to a "carry" or "noCarry" memory block what would write the carry bit to the 16^1 place output. The 16^0 place output was determined by the add operator.

Like the Counter the second program was tested in the PRISM simulator then on the FPGA. Both tests demonstrated satisfactory results with only minor debugging.

The only error in the program was an absence of necessary jump commands at the end of the program which caused the default output of the 16^1 place to be written at 1. 

#### Documentation

Dr. Neeble helped debug the ROM error that was preventing the FPGA implementation from successful operation 
