# Simple C++ based assembler for multicycle processor that is used in ECE352 course in the University of Toronto

Credit: R.Willenberg and F.Martin del Campo for the initial version of this assembler. 

1. You can compile and use the assembler for the multicycle processor using G++:
	`> make`
2. You can now assemble files by calling
	`> ./asm YOURCODE.s`
3. The assembler will produce a file 'data.mif' to use for inclusion in your Quartus bitstream, and a file 'data.mif.mem' that is used for ModelSim simulation
4. You can either copy the two files into your Quartus project folder, or copy the asm.exe file into your project folder and build the files right there, whatever you preference is
5. If you haven't changed anything about your processor hardware, but want to test a new assembly program on the board  you can update and download your system with a new *.mif file by clicking on the 'UPDATE_MIF.bat' batch file in the Quartus project folder. It is much faster than re-starting the whole Quartus compile. Alternatively, you can call it directly after assembling from the NIOS II console with
	`> cmd /C UPDATE_MIF.bat`
	
REMEMBER: If you try out hardware changes, SIMULATE them first before trying them on the board. Much easier and faster to debug.

## NOTES:
- IMM3 is a 3-bit immediate using sign/magnitude representation:
  - bit 2 = sign, value = bits 1 and 0
- IMM4: 4-bit immed using 2's complement representation
- IMM5: 5-bit unsigned
- EXT(X): sign-extend X to 8 bits
  - eg: 101 = 5; EXT(101) = 00000101
  - eg: 1101 = -3; EXT(1101) = 11111101
- R1, R2: registers variables
- TMP = temporary variable

## Register Transfer Notation:
`
	LOAD R1 (R2):
	     TMP = MEM[[R2]]
	     R1 = TMP
	     [PC] = [PC] + 1
	
	STORE R1 (R2):
	     MEM[[R2]] = [R1]
	     [PC] = [PC] + 1
	
	ADD R1 R2
	     TMP = [R1] + [R2]
	     R1 = TMP
	     IF (TMP == 0) Z = 1; ELSE Z = 0;
	     IF (TMP < 0) N = 1; ELSE N = 0;
	     [PC] = [PC] + 1
	
	SUB R1 R2
	     TMP = [R1] - [R2]
	     R1 = TMP
	     IF (TMP == 0) Z = 1; ELSE Z = 0;
	     IF (TMP < 0) N = 1; ELSE N = 0;
	     [PC] = [PC] + 1
	
	NAND R1 R2
	     TMP = [R1] bitwise-NAND [R2]
	     R1 = TMP
	     IF (TMP == 0) Z = 1; ELSE Z = 0;
	     IF (TMP < 0) N = 1; ELSE N = 0;
	     [PC] = [PC] + 1
	
	ORI IMM5
	    TMP = [K1] bitwise-OR IMM5
	    [K1] = TMP
	    IF (TMP == 0) Z = 1; ELSE Z = 0;
	    IF (TMP < 0) N = 1; ELSE N = 0;
	    [PC] = [PC] + 1
	
	SHIFT R1 IMM3
	    IF (IMM3 > 0) TMP = [R1] << IMM3
	    ELSE TMP = [R1] >> (-IMM3)
	    R1 = TMP
	    IF (TMP == 0) Z = 1; ELSE Z = 0;
	    IF (TMP < 0) N = 1; ELSE N = 0;
	    [PC] = [PC] + 1
	
	BZ IMM4
	     IF (Z == 1) PC = [PC] + EXT(IMM4)
	     [PC] = [PC] + 1
	
	BNZ IMM4
	     IF (Z == 0) PC = [PC] + EXT(IMM4)
	     [PC] = [PC] + 1
	
	BPZ IMM4
	     IF (N == 0) PC = [PC] + EXT(IMM4)
	     [PC] = [PC] + 1
`

## Instruction Encodings:
Legend:
Rx = 2 bit encoding of register
I  = immediate value
`
	Bit [MSB]76543210[LSB]
	
	LOAD:    R1R20000
	STORE:	 R1R20010
	ADD:	 R1R20100
	SUB:	 R1R20110
	NAND:	 R1R21000
	ORI:	 IIIII111
	SHIFT:	 R1III011
	BZ:	 IIII0101
	BNZ:	 IIII1001
	BPZ:	 IIII1101
	
	VADD:  V1V21110
	VLOAD  V1R21010
	VSTORE V1R21110
	
	STOP:	 00000001
	NOP:	 10000001
`
