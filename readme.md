# VaDeRetro EMS Option ROM (`vaderetro.bin`)

`vaderetro.bin` is a 32 KB PC Option ROM designed for x86 vintage computing architectures (8088/8086/80286+). It provides automated hardware initialization, RAM verification, and visual reporting during POST for compatible EMS (Expanded Memory Specification) ISA expansion boards.

## 🛠️ Project Origin & Acknowledgments

This ROM binary was specifically developed to test and validate the **[ISA_EMS_4Mb](https://github.com/hkzlab/ISA_EMS_4Mb)** expansion card created by **[hkzlab](https://github.com/hkzlab)**.

Special thanks to **hkzlab** for designing and open-sourcing this amazing 4 MB EMS ISA board for vintage PC enthusiasts. This ROM provides an immediate, hardware-level diagnostic screen during boot to verify that the board's I/O decoding, RAM pages, and Upper Memory page frames are fully operational without requiring any OS drivers.

---

## 📐 Technical Specifications

| Parameter | Specification | 
| ----- | ----- | 
| **ROM Size** | 32,768 Bytes (32 KB / 64 blocks of 512B) | 
| **Target Architecture** | Real Mode x86 (8088, 8086, 80286, 80386, 80486) | 
| **Bus Interface** | 8-bit / 16-bit ISA | 
| **Header Signature** | `55 AA 40` (Option ROM standard) | 
| **Checksum Type** | 8-bit Modulo 256 | 
| **Max EMS RAM Supported** | 4,096 KB (256 pages × 16 KB) | 
| **Inter-process Storage** | Physical address `0000:04FEh` (System Scratch Area) | 

---

## ⚡ POST Initialization Sequence

During system boot (POST), the motherboard BIOS scans the Upper Memory Area (UMA) for the `55 AA` Option ROM signature, validates the byte checksum, and executes `vaderetro.bin`.

```
               ┌────────────────────────────────────────┐
               │    Option ROM Execution (FAR CALL)     │
               └───────────────────┬────────────────────┘
                                   │
                                   ▼
               ┌────────────────────────────────────────┐
               │  1. I/O Port & Page Frame Discovery    │
               └───────────────────┬────────────────────┘
                                   │
                                   ▼
               ┌────────────────────────────────────────┐
               │   2. Non-Destructive Memory Test       │
               │      (Validates up to 256 pages)       │
               └───────────────────┬────────────────────┘
                                   │
                                   ▼
               ┌────────────────────────────────────────┐
               │  3. Cache Total RAM Size @ 0000:04FEh  │
               └───────────────────┬────────────────────┘
                                   │
                                   ▼
               ┌────────────────────────────────────────┐
               │  4. Render Dynamic ASCII Info Box      │
               └───────────────────┬────────────────────┘
                                   │
                                   ▼
               ┌────────────────────────────────────────┐
               │  5. Wait for Keypress & Return (RETF)  │
               └────────────────────────────────────────┘
```

---

## 🔍 Hardware Detection & Scan Ranges

### 1. I/O Base Address Probe

The ROM scans candidates across standard EMS port addresses:
* `0x0258` to `0x027C` (Step 4)
* `0x0280` to `0x029C` (Step 4)
* `0x0300`

### 2. Page Frame Probe

The ROM tests segment bases within the Upper Memory Area:
* `C000h`, `C400h`, `C800h`, `CC00h`
* `D000h`, `D400h`, `D800h`, `DC00h`
* `E000h`, `E400h`, `E800h`, `EC00h`

When a valid hardware match is found, the original RAM byte at offset `0010h` is preserved, test patterns (`0x5A5A` and `0x3C3C`) are written and verified, and the original value is restored non-destructively.

---

## 💾 Deployment & EPROM Flashing

### Physical Hardware

Burn `vaderetro.bin` directly onto compatible EPROM/EEPROM chips for your Option ROM socket:
* **27C256** / **28C256** (32 KB)
* **27C512** / **28C512** (64 KB - requires padding or mirroring)

### Emulators

To use `vaderetro.bin` with **86Box** or **PCem**:
1. Copy `vaderetro.bin` into your emulator's `roms/` directory or ISA expansion card manager.
2. Configure the Option ROM address to match your board's ROM socket (e.g., `D8000h`).