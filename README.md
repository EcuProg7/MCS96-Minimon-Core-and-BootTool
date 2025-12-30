# MCS96-Minimon-Core-and-BootTool

********** This is for educational and research use only!!! ***********
license : free under GPLv3

Perschl Minimon Core (K-Line) adapted to MCS96 with Python BootTool , ext flash and I2C 24C0x eeprom Driver

Compiler for MCS96 IC96 Tools: https://www.njohnson.co.uk/zip/Roland/96tools.zip
Batch files for compiler are written with absolut paths (you need to adapt it yourself)
IC96 needs to be in E:\temp\MCS96\
to compile run batch file RunCompile.bin in folder of project like for EEP driver:
e:\temp\MCS96\C96EEP\RunCompile.bat

You need Dosbox or other dos emulator to run this compiler

As hardware platform an ancient Motronic M3.8.2 was used with 87C196KR , SCL4402-V04 Asic, 8k ext Ram , 29F400BB or bigger on an adapter board (schematic and pcb will follow)

The Minimon Core starting at address 0x2000 needs to be in internal rom of 196KR or in external Flash  (with additional hardware paging on adapter board) 

SCL4402:
the asic needs some init values written to specific addresses to access ram and do paging on ext flash. Init is retreived from firmware binary.
I don´t know if all are needed or what they are doing, it works. The asic is memory mapped starting at address 0xe000 with inst pin P5.1 low
#scl44 init

    ret = SetByteAtAddress(ser, 0xe02f, 0x41)
    ret &= SetByteAtAddress(ser, 0xe030, 0xa)
    ret &= SetByteAtAddress(ser, 0xe031, 0xf)
    ret &= SetByteAtAddress(ser, 0xe032, 0x88)
    ret &= SetByteAtAddress(ser, 0xe042, 0x1)
    ret &= SetByteAtAddress(ser, 0xe040, 0x34)
    ret &= SetByteAtAddress(ser, 0xe01a, 0x3)
    ret &= SetByteAtAddress(ser, 0xe02c, 0x3d)
after a read or write ext flash operation something goes weird and the borad needs to be restarted ?!?!

Memory map with external flash adapter board:
  normal operation (boot pin low) and inst pin set:
  
    µc                    flash              page reg in asic 0xE040(write) / 0xE041(read):
    
    0x0000 - 0xBFFF      0x40000 - 0x4BFFF      x
    0xc000 - 0xfFFF      0x40000 - 0x43FFF      0x30
    0xc000 - 0xfFFF      0x44000 - 0x47FFF      0x31
    0xc000 - 0xfFFF      0x48000 - 0x4bFFF      0x32
    0xc000 - 0xfFFF      0x4C000 - 0x4FFFF      0x33
    0xc000 - 0xfFFF      0x50000 - 0x53FFF      0x34
    0xc000 - 0xfFFF      0x54000 - 0x57FFF      0x35
    0xc000 - 0xfFFF      0x58000 - 0x5BFFF      0x36
    0xc000 - 0xfFFF      0x5C000 - 0x5FFFF      0x37
    0xc000 - 0xfFFF      0x60000 - 0x63FFF      0x38
    0xc000 - 0xfFFF      0x64000 - 0x67FFF      0x39
    0xc000 - 0xfFFF      0x68000 - 0x6BFFF      0x3A
    0xc000 - 0xfFFF      0x6C000 - 0x6FFFF      0x3B
    0xc000 - 0xfFFF      0x70000 - 0x73FFF      0x3C
    0xc000 - 0xfFFF      0x74000 - 0x77FFF      0x3D
    0xc000 - 0xfFFF      0x78000 - 0x7BFFF      0x3E
    0xc000 - 0xfFFF      0x7C000 - 0x7FFFF      0x3F
    
  both operations and inst pin reset:
  
      µc
      
    0x0000 - 0xBFFF      0x40000 - 0x4BFFF    flash   
    0xc000 - 0xDFFF      0x0000 - 0x1FFF      ext RAM
    0xE000 - 0xFFFF                          ASIC (?)

  boot operation (boot pin high) and inst pin set:
  
    µc                    flash              page reg in asic 0xE040(write) / 0xE041(read):
    
    0x0000 - 0x7FFF      0x00000 - 0x07FFF      x
    0x8000 - 0xBFFF      0x48000 - 0x4BFFF      x
    0xc000 - 0xfFFF      0x40000 - 0x43FFF      0x30
    0xc000 - 0xfFFF      0x44000 - 0x47FFF      0x31
    0xc000 - 0xfFFF      0x48000 - 0x4bFFF      0x32
    0xc000 - 0xfFFF      0x4C000 - 0x4FFFF      0x33
    0xc000 - 0xfFFF      0x50000 - 0x53FFF      0x34
    0xc000 - 0xfFFF      0x54000 - 0x57FFF      0x35
    0xc000 - 0xfFFF      0x58000 - 0x5BFFF      0x36
    0xc000 - 0xfFFF      0x5C000 - 0x5FFFF      0x37
    0xc000 - 0xfFFF      0x60000 - 0x63FFF      0x38
    0xc000 - 0xfFFF      0x64000 - 0x67FFF      0x39
    0xc000 - 0xfFFF      0x68000 - 0x6BFFF      0x3A
    0xc000 - 0xfFFF      0x6C000 - 0x6FFFF      0x3B
    0xc000 - 0xfFFF      0x70000 - 0x73FFF      0x3C
    0xc000 - 0xfFFF      0x74000 - 0x77FFF      0x3D
    0xc000 - 0xfFFF      0x78000 - 0x7BFFF      0x3E
    0xc000 - 0xfFFF      0x7C000 - 0x7FFFF      0x3F

Flash Driver:
the F29xx Driver is split into 3 parts: GetState, Erase, Write
Each of this parts is loaded into internal ram at 0x400 - 0x4FF. Erase and Write Driver have Parameters from 0x400 to 0x407
Internal Ram at 0x100 - 0x1ff  is used for data copy.

Eeprom Driver:
This driver is bigger than the free internal ram so i decided to use the external ram. To do this the inst pin (P5.1) needs to be reset(low). 
Driver is loaded into ext Ram starting at 0xC000. Int ram at 0x400 is used for parameters and data copy.

Pics from my first prototype ( the adapter board will get better)
![IMG_0493](https://github.com/user-attachments/assets/07e74fcf-1ac5-4606-9d6e-72d49c77db1b)

