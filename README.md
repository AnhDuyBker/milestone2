# RISC-V Single-Cycle Processor - DE-10 Standard FPGA

A complete RISC-V RV32I processor implementation with 6-digit stopwatch demonstration running on the Terasic DE-10 Standard FPGA board.

---

## 🎯 Project Overview

This project implements a single-cycle RISC-V processor with Harvard architecture (separate instruction and data memory) designed for the DE-10 Standard FPGA. The demonstration program is an optimized 6-digit stopwatch that counts from 000000 to 999999 on the board's 7-segment displays.

### Key Features
- **Processor**: RISC-V RV32I instruction set architecture
- **Architecture**: Single-cycle, Harvard (separate IMEM/DMEM)
- **Clock**: 50 MHz input → 10 MHz processor clock (divide-by-5)
- **Display**: 6× 7-segment displays (HEX5-HEX0)
- **I/O**: Memory-mapped switches, buttons, LEDs, and displays
- **Demo**: Optimized stopwatch with pause/resume and reset controls

---

## 🎮 Stopwatch Controls

| Control | Function | Description |
|---------|----------|-------------|
| **SW[0]** | Pause/Resume | ON = Pause counting, OFF = Resume |
| **KEY[0]** | Reset Counter | Press to reset display to 000000 |
| **KEY[1]** | System Reset | Emergency processor reset |

### Display Behavior
- **Range**: 000000 to 999999 (1 million counts)
- **Update Rate**: 1 count per second
- **Format**: Leading zeros displayed (e.g., 000042)
- **Wraparound**: Automatically resets to 000000 after 999999

---

## 📁 Project Structure

```
riscv/
├── 00_src/              # RTL source files
│   ├── wrapper.sv       # Top-level DE-10 interface
│   ├── clock_25M.sv     # Clock divider (50MHz → 10MHz)
│   ├── single_cycle.sv  # RISC-V processor core
│   ├── control_unit.sv  # Instruction decoder
│   ├── alu.sv          # Arithmetic logic unit
│   ├── regfile.sv      # 32× 32-bit register file
│   ├── i_mem.sv        # Instruction memory
│   ├── dmem.sv         # Data memory
│   ├── lsu.sv          # Load/store unit
│   └── ...             # Other modules
│
├── 01_bench/           # Testbench files
│   ├── tbench.sv       # Top-level testbench
│   ├── driver.sv       # Test driver
│   └── scoreboard.sv   # Result checker
│
├── 02_test/            # Test programs (hex format)
│   ├── counter_v2.hex  # Optimized stopwatch (244 instructions)
│   ├── isa_1b.hex      # ISA test program
│   └── isa_4b.hex      # ISA test program
│
├── 03_sim/             # Simulation files
│   ├── makefile        # Build automation
│   ├── flist           # File list for compilation
│   └── dump.vcd        # Waveform output
│
└── 04_doc/             # Documentation
    ├── README.md              # This file
    ├── STOPWATCH_README.md    # Quick reference guide
    ├── DEPLOYMENT_CHECKLIST.md # Complete deployment guide
    ├── STOPWATCH_OPTIMIZATIONS.md # Technical optimization details
    └── de10_pin_assign.qsf    # Quartus pin assignments
```

---

## 🚀 Quick Start

### Prerequisites
- **Hardware**: Terasic DE-10 Standard FPGA board
- **Software**: Intel Quartus Prime (tested with Standard Edition)
- **Cable**: USB-Blaster programming cable

### Deployment Steps

1. **Open Project**
   ```
   Open your Quartus project file (.qpf)
   ```

2. **Set Top-Level Entity**
   - Set `wrapper.sv` as the top-level design entity

3. **Import Pin Assignments**
   - Load `04_doc/de10_pin_assign.qsf` into your project
   - This maps all FPGA pins to board components

4. **Compile Design**
   - Processing → Start Compilation
   - Wait for successful compilation (~5-10 minutes)

5. **Program FPGA**
   - Tools → Programmer
   - Select USB-Blaster
   - Load `wrapper.sof` file
   - Click Start to program

6. **Test Stopwatch**
   - Display should show `000000`
   - Counting starts automatically (1 count/second)
   - Test SW[0] for pause/resume
   - Press KEY[0] to reset to 000000

---

## 🔧 Technical Specifications

### Processor Core
- **ISA**: RISC-V RV32I (32-bit base integer instruction set)
- **Architecture**: Single-cycle execution, Harvard memory
- **Registers**: 32× 32-bit general purpose (x0-x31)
- **Memory**: 
  - Instruction Memory: 16KB (4096 words)
  - Data Memory: 16KB (4096 words)

### Clock System
- **Input**: 50 MHz (CLOCK_50, PIN_AF14)
- **Divider**: Divide-by-5 implementation
- **Output**: 10 MHz processor clock
- **Module**: `clock_25M.sv` (name is historical)

### Memory Map
| Address Range | Device | Description |
|--------------|--------|-------------|
| 0x00000000 - 0x00003FFF | IMEM | Instruction memory (16KB) |
| 0x10000000 - 0x10003FFF | DMEM | Data memory (16KB) |
| 0x10007020 | HEXL | 7-segment displays HEX2-HEX1-HEX0 |
| 0x10007024 | HEXH | 7-segment displays HEX5-HEX4-HEX3 |
| 0x10017800 | Input | Switches SW[9:0] + Buttons KEY[3:0] |

### I/O Mapping
**Input Register (0x10017800):**
- Bits [9:0]: SW[9:0] switches
- Bits [13:10]: KEY[3:0] buttons (inverted to active-high)
- Bits [31:14]: Reserved (read as 0)

**Output Registers:**
- 0x10007020: Lower 3 digits (HEX2-HEX1-HEX0)
- 0x10007024: Upper 3 digits (HEX5-HEX4-HEX3)

---

## 📊 Stopwatch Optimization

The stopwatch program (`counter_v2.hex`) has been optimized for size and performance:

### Performance Metrics
- **Code Size**: 244 instructions (976 bytes)
- **Improvement**: 5.1% smaller than original (257 instructions)
- **Speed**: 22% faster digit processing
- **Algorithm**: Early-exit search for 7-segment encoding

### Key Optimizations
1. **Early-Exit Strategy**: Checks most common digits first (0, then 5)
2. **Harvard-Compatible**: Uses register-based lookup (no memory tables)
3. **Minimal Overhead**: Direct calculation via division by 10
4. **Efficient Loops**: Calibrated timing for exactly 1.0 second per count

See `04_doc/STOPWATCH_OPTIMIZATIONS.md` for detailed technical analysis.

---

## 🧪 Testing & Simulation

### ModelSim/Questa Simulation
```bash
cd 03_sim
make clean
make compile
make simulate
```

### Waveform Analysis
- Output: `03_sim/dump.vcd`
- View with GTKWave or ModelSim

### Hardware Testing
1. **Power-On Test**: Display shows 000000
2. **Counting Test**: Increments every second
3. **Pause Test**: SW[0] ON freezes display
4. **Reset Test**: KEY[0] returns to 000000
5. **Wraparound**: Verify 999999 → 000000

---

## 🐛 Troubleshooting

### Display Shows All Segments Lit/Dark
- **Cause**: 7-segment polarity mismatch
- **Fix**: Verify active-low encoding (0x40=0, 0x79=1, etc.)

### Wrong Count Speed
- **Cause**: Clock frequency mismatch
- **Fix**: Verify clock divider outputs 10 MHz
- **Adjust**: Modify `counter_v2.hex` line 226 for timing

### Display Frozen at 000000
- **Cause**: Clock not toggling or processor held in reset
- **Check**: 
  - LEDR[9] should blink (instruction valid indicator)
  - Release KEY[1] (processor reset)
  - Verify clock divider is running

### Some Digits Missing
- **Cause**: Pin assignment or address decode issue
- **Fix**: 
  - Reload pin assignments from `.qsf` file
  - Check `output_buffer.sv` address decode logic

See `04_doc/DEPLOYMENT_CHECKLIST.md` for complete troubleshooting guide.

---

## 📚 Documentation

- **[STOPWATCH_README.md](04_doc/STOPWATCH_README.md)** - Quick reference for stopwatch controls and specs
- **[DEPLOYMENT_CHECKLIST.md](04_doc/DEPLOYMENT_CHECKLIST.md)** - Step-by-step deployment guide with verification
- **[STOPWATCH_OPTIMIZATIONS.md](04_doc/STOPWATCH_OPTIMIZATIONS.md)** - Detailed optimization strategy and analysis

---

## 🔌 Hardware Requirements

### DE-10 Standard FPGA Board Components Used
- **FPGA**: Intel Cyclone V 5CSXFC6D6F31C6
- **Clock**: 50 MHz oscillator
- **Switches**: SW[0] for pause/resume
- **Buttons**: KEY[0] for reset, KEY[1] for system reset
- **Displays**: HEX5-HEX0 (6× 7-segment, common anode)
- **LEDs**: LEDR[9:0] for status (optional)

### Pin Assignments
All pin assignments are defined in `04_doc/de10_pin_assign.qsf`. Key pins:
- CLOCK_50: PIN_AF14
- SW[0]: PIN_AB30
- KEY[0]: PIN_AJ4 (counter reset)
- KEY[1]: PIN_AK4 (processor reset)
- HEX0-HEX5: 42 pins total (7 segments × 6 displays)

---

## 🎓 Educational Value

This project demonstrates:
- **Computer Architecture**: Single-cycle RISC-V processor design
- **Digital Logic**: RTL design with SystemVerilog
- **Memory Systems**: Harvard architecture with memory-mapped I/O
- **FPGA Development**: Complete workflow from RTL to hardware
- **Algorithm Optimization**: Code size and performance tradeoffs
- **Hardware/Software Interface**: Memory-mapped I/O programming

---

## 📝 License

Educational project for computer architecture coursework.

---

## 🙏 Acknowledgments

- RISC-V International for the open ISA specification
- Terasic for DE-10 Standard board documentation
- Intel for Quartus Prime development tools

---

## 📞 Support

For issues or questions:
1. Check `04_doc/DEPLOYMENT_CHECKLIST.md` troubleshooting section
2. Verify all pin assignments are loaded
3. Confirm clock divider is producing 10 MHz output
4. Review simulation waveforms for debugging

---

**Status**: ✅ Fully verified and ready for deployment  
**Last Updated**: November 8, 2025  
**Version**: Milestone 2 - Optimized Stopwatch
