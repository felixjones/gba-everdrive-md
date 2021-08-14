# <a name="reference"></a>Everdrive GBA Reference

## Overview

- [Memory Map](#memorymap)
- [I/O Map](#iomap)

## Programming

- [BIOS](#bios)
- [FPGA](#fpga)
- [SD](#everdrivesd)
- [Everdrive key](#everdrivekey)

# <a name="memorymap"></a>Memory Map

## Everdrive Memory (Game Pak)

Table below is for the Wait State 0 GamePak address range.

```
  09FC0000-09000003   BIOS
  09FC000A-0900000B   FPGA
  09FC0010-09000017   SD
  09FC00B4-090000B5   KEY
```

# <a name="iomap"></a>I/O Map

## BIOS Registers

```
  9FC0000h  2    W    CFG       Device configuration
  9FC0002h  2    R    STATUS    Device status
```

## FPGA Registers

```
  9FC000Ah  2    R    FPGA_VER  FPGA version
```

## SD Registers

```
  9FC0010h  1    R/W  SD_CMD    SD command
  9FC0011h  1    -    -         Not used
  9FC0012h  2    R/W  SD_DAT    SD data
  9FC0014h  2    W    SD_CFG    SD configuration
  9FC0016h  2    W    SD_RAM    SD RAM
```

## Key Registers

```
  9FC00B4h  2    W    KEY       Everdrive unlock key
```

# <a name="bios"></a>BIOS

## Registers

- [Configuration & Status](#configurationandstatus)

# <a name="configurationandstatus"></a>Configuration & Status

## 9FC0000h - CFG - Configuration (Write)

```
  Bit   Expl.
  0     Map ROM registers      (0=Unmapped, 1=Mapped)
  1     Map ROM PSRAM          (0=Boot ROM, 1=PSRAM)
  2     PSRAM write enable     (0=Off, 1=On)
  3     Auto write enable???   (0=Off, 1=On) Used for DMA'ing from ROM???
  4-6   Save type              (0=None, 1=EEPROM, 2=SRAM, 4=Flash 64K, 5=Flash 128K)
  7-8   SRAM bank              (0-3) Select which bank of 64K SRAM is mapped to E000000h
  9     RTC enable             (0=Off, 1=On)
  10    ROM bank???            (0-1) Select which bank of ROM is mapped???
  11    Big ROM                (0=Off, 1=On) Enable EEPROM mapping for 32M games
  12-15 Not used
```

## 9FC0002h - STATUS - Status (Read)

```
  Bit   Expl.
  0     SD busy                (0=No, 1=Yes)
  1     SD card timeout        (0=No, 1=Yes)
  2-15  Not used
```

# <a name="fpga"></a>FPGA

## Registers

- [Version](#version)

# <a name="version"></a>Version

## 9FC000Ah - FPGA_VER - Status (Read)

```
  Bit   Expl.
  0-15  FPGA version code
```

# <a name="everdrivesd"></a>SD

## Registers

- [Command](#command)
- [Data](#data)
- [Configuration](#configuration)
- [RAM](#ram)

# <a name="command"></a>Command

## 9FC0010h - SD_CMD - Command (Read/Write)

Single byte commands can be written to this register.

The response of the last command is retrieved by reading this register.

Command arguments are sent in big-endian order.

```
  Bit   Expl.
  0-7   Command/response
```

### <a name="commandsummary"></a>SD Command Summary

```
  Code  Hex     Mnemonic                Basic Functions
  CMD0  40h     GO_IDLE_STATE           Software reset
  CMD1  41h     SEND_OP_COND            Brings card out of idle state
  CMD2  42h     ALL_SEND_CID            Reads the "card identification register" (CID)
  CMD3  43h     SEND_RELATIVE_ADDR      Reads the "relative card address register" (RCA)
  CMD6  46h                             Reserved
  CMD7  47h     SELECT/DESELECT_CARD    Toggles card between stand-by and transfer states
  CMD8  48h     SEND_IF_COND            Tell the SD card which voltages it must accept
  CMD9  49h     SEND_CSD                Reads the "card specific data" (CSD)
  CMD12 4Ch     STOP_TRANSMISSION       Stop transmission on multiple block read
  CMD17 51h     READ_SINGLE_BLOCK       Reads single block
  CMD18 52h     READ_MULTIPLE_BLOCK     Reads multiple blocks
  CMD24 58h     WRITE_BLOCK             Writes single block
  CMD25 59h     WRITE_MULTIPLE_BLOCK    Writes multiple blocks
  CMD41 69h                             Reserved
  CMD55 77h     APP_CMD                 Next command set to be an application command
  CMD58 7Ah     READ_OCR                Reads the "operation condition register" (OCR)
```

CMD55 sets the SD to expect an application specific command. Once an application specific command is sent the SD returns to its normal operation mode.

### <a name="appcommandsummary"></a>Application Specific Command Summary

```
  Code      Hex     Mnemonic            Basic Functions
  ACMD6     46h     SET_BUS_WIDTH       Set 4-bit operation mode
  ACMD41    69h     SD_SEND_OP_COND     When accepted, the SD card will have left the idle-state
```

# <a name="data"></a>Data

## 9FC0012h - SD_DAT - Data (Read/Write)

Data is written as big-endian and received as big-endian. 8-bit reads requires endian re-ordering.

8-bit writes requires bits 8-15 to be set to FFh.

```
  Bit   Expl.
  0-7   "High" byte
  8-15  "Low" byte
```

## DMA write

Prepare 512 byte buffer.

Write 0 to SD_RAM.

Set SD_CFG mode to 4.

DMA transfer 512 bytes into SD_DAT.

## DMA read

Set SD_CFG mode to 4 with WAIT_F0 and STRT_F0 set.

Read SD_DAT.

Read STATUS until busy is unset.

If STATUS timeout is set: repeat. Else: transfer 512 bytes from SD_DAT.

# <a name="configuration"></a>Configuration

## 9FC0014h - SD_CFG - Configuration (Write)

```
  Bit   Expl.
  0     Speed                  (0=Low, 1=High)
  1-2   Mode                   (0-3) Card operating mode
  3     Await F0               (0=No, 1=Yes) Wait for "F0" to complete
  4     Start F0               (0=Off, 1=On) Begin "F0"
  5-15  Not used
```

# <a name="ram"></a>RAM

## 9FC0016h - SD_RAM - RAM (Write)

Writing 0 to this register is required to enable writing via DMA.

# <a name="everdrivekey"></a>Everdrive Key

## Registers

- [Key](#key)

# <a name="key"></a>Key

## 9FC00B4h - KEY - Key (Write)

The value 00A5h must be written to this register to unlock all Everdrive GBA functionality.

```
  Bit   Expl.
  0-15  Everdrive unlock key
```

00A5h is the Unicode for the Yen currency symbol. This can be interpreted as the EASCII string "¥", with the high-byte being the NULL terminator.
