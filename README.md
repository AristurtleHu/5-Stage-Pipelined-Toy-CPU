## Project 2.2: Implement a 5-Stage Pipelined Toy CPU
implement a 5-stage pipelined CPU which can process all the instructions from Project 2.1 ([code](https://github.com/AristurtleHu/Single-Cycle_CPU_of_RISC-V))

### Introduction
Opening `proj_2_2_top.circ` with Logisim-evolution, you will find several subcircuits inside it.

*   **TOP:** This is the top-level circuit. It represents the implementation of the target toy CPU that includes the subcircuits.
*   **testbench:** This is a completed circuit for testing. You can write any binary format instruction to the instruction memory (ROM) and observe what your circuit operates.

Recall the classic 5 stages of CPU executing an instruction:
1.  **Instruction Fetch (IF)**
2.  **Instruction Decode (ID)**
3.  **Execute (EX)**
4.  **Memory (MEM)**
5.  **Write Back (WB)**

In a 5-stage pipelined CPU, multiple instructions are processed simultaneously at different stages. The hazards are handled carefully in the pipeline. The explanations of the hazards are detailed in the [Hazards](#hazards) section.

### Hazards

#### Structural Hazards
Structural hazards occur when multiple instructions simultaneously compete for the same physical hardware resource. In a 5-stage pipelined CPU, these conflicts primarily affect memory access and register file operations. Need to address the hardware limitation in the register file, where concurrent read and write operations from different pipeline stages can create access conflicts. For standardized testing purposes, assuming that **register file and data memory write at the falling edge of the clock signal, while the other registers and memory write at the rising edge.**

#### Data Hazards
Data hazards emerge when an instruction depends on data produced by a previous instruction which is still in the pipeline and not updated in the register file. These dependencies should be efficiently resolved through **data forwarding**. It routes computed values directly from the pipeline register to an earlier stage of the pipeline where they are needed.
However, data forwarding cannot completely resolve the case of a load delay slot as shown in the below example with `x1`. Ignore this kind of data hazard because one bubble or other instructions are inserted  between the two instructions in the test cases.
```assembly
    lw x1, 0(x3)
    add x7, x1, x0
```

#### Control Hazards
Control hazards occur when the processor cannot determine which instruction to fetch next until the branch or jump result is obtained. Implement **static branch prediction** as follows:

1.  Initially assume that all branches or jumps are **not taken** and continue executing subsequent instructions, despite the branch condition being met or it being an unconditional jump instruction.
2.  When a branch or jump **should be taken**, flush the pipeline registers except for those storing the instructions fetched before the branch/jump instruction.

For standardized testing purposes, assuming that branch/jump resolution occurs in the **EX stage**, but the branch/jump taken signal (i.e., `control_pc_mem` in the TOP I/O) is fetched from the **MEM stage** for final determination.

### TOP I/O
The TOP circuit incorporates predefined input/output ports for testing purposes, where most output signals are treated as "probes". 
The inputs and outputs of the top level are fixed in the TOP circuit. The naming scheme of the output is `{signal}_{stage}`, which means the `{signal}` from `{stage}`.

| Type   | signal           | bit width | description                               |
| :----- | :--------------- | :-------- | :---------------------------------------- |
| input  | `clk`            | 1         | clock                                     |
| input  | `rst`            | 1         | reset                                     |
| input  | `inst`           | 32        | RV32I instruction from inst memory        |
| output | `current_pc_if`  | 32        | current instruction address from IF stage |
| output | `mem_wen_mem`    | 1         | memory write enable from MEM stage        |
| output | `mem_din_mem`    | 32        | data written to memory from MEM stage     |
| output | `mem_dout_mem`   | 32        | data from memory from MEM stage           |
| output | `mem_addr_mem`   | 32        | address of memory from MEM stage          |
| output | `control_en_mem` | 1         | enable writing value to pc from MEM stage |
| output | `control_pc_mem` | 32        | value written to pc from MEM stage        |
| output | `wb_en_wb`       | 1         | regfile write enable from WB stage        |
| output | `wb_addr_wb`     | 5         | address written to regfile from WB stage  |
| output | `wb_data_wb`     | 32        | data written to regfile from WB stage     |

### Test
For the convenience of calibrating each cycle, an additional counter has been added to the `testbench`, which you can ignore.
For the purpose of ensuring smooth testing, NOP instructions (i.e., `addi x0, x0, 0`) are inserted in the test cases. Don't care about the outputs for NOP instructions.

#### Local Test
Provide two local tests containing all instruction patterns, but the reference output does not explicitly mark don't-care signals. `localtest_0` has no data and control hazards, while `localtest_1` includes some hazard cases.

To run the tests:
```bash
chmod +x test.sh
./test.sh
```
This script is suitable for Linux systems. If you want to run it on other operating systems, you need to make some modifications.