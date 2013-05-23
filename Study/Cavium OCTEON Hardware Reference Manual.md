Core Clock Frequency = PLL_REF_CLK × [C_MUL] = 50 MHz × [C_MUL]

Coprocessor Clock Frequency = PLL_REF_CLK × [PNR_MUL] = 50 MHz × [PNR_MUL]

Coherent Multicore and I/O L2/DRAM Sharing:
==========================================
	shared main memomry(implemented via the level-2 cache and the DRAM)is the prima-
ry communication media for bulk(大块) transfers in the CN63XX.

In-line Packet-Processing Hardware Acceleration:
===============================================
The CN63XX in-line packet-processing hardware units complete the follwing tasks before 
a core receives a packet:
	1. Allocate DRAM buffers to hold the packet bytes.
	2. Send packet data into these DRAM buffers via DMA operations in a format con-
	   venient to upper-level software.This can enable copy-free core software.		upper-level:上层
	3. Parse the layer-two/IP packet,checking for common exception conditions.This 
	   is done for every ingress packet.The layer-four TCP/UDP checksum checks are 
	   included,among many others.
	4. Perform optional mutual exclusion via a programmable 7-tuple classification.		classification:分类，分级
												mutual:共有的，共同的。
When software has finished processing the packet, CN63XX in-line hardware performs
the following tasks:
	1. DMA the packet-send command from the selected output queue(and free the
   	   available queue space)
	2. DMA packet data from L2/DRAM.This includes multiple modes to gather non-
	   contiguous packet data.
	3. Generate the TCP/UDP checksum.(CN63XX can perform this very efficiently;		efficiently:高效地
	   the packet data is read out of L2/DRAM memory only once to both calculate
	   the checksum and send the packet off-chip)
	4. Free the DRAM buffers

Essential Quality of Service (QoS) Functions Implemented in Hardware:
====================================================================
CN63XX has some essential QoS capabilities implemented directly in hardware:
	1. The hardware has eight separate input work queues.
	2. The in-line input packet processing hardware can classify packets into one
	   of the eight input work queues using default values,DSA/VlAN priorities and
	   IP Diffserv values,configurable on a per-port basis.					diffserv:差异式服务
	3. The in-line input packet processing hardware may discard an input packet be-
	   fore buffering and presenting it to a core.The hardware implements both a r-
	   andom early discard (RED) algorithm and a threshold algorithm to decide when		threshold:入口，门槛
	   or if to discard and input packet.The RED-like algorithm can be configured 
	   differently for each of the eight QoS levels,and the threshold algorithm can
	   be configured differently for different ports
	4. Each output port can be configured to have multiple queues.The queues can be
	   configured with different priorities.The hardware implements both static pr-		priority:优先级
	   iority and weighted round-robin. 							weigthed round-robin:加权循环

Coprocessor Accelerators:
========================
	1. Hyper Finite Automata(HFA)/Deterministic Finite Automata(DFA):string-matchi-
	   ng with the attached DRAM(NOTE:in the rest of the manual,these terms will be
	   used interchangeably).								interchangeable:可交替的，可互换的。
	2. Compression/Decompression(ZIP):hardware-acceleration based on standard infl-		inflate:使膨胀,膨胀。
	   ate/deflate algorithms supporting GZIP/PKZIP and variant protocols.			deflate:使缩小,缩小。
	3. Timer(TIM):timer support is particularly useful for TCP implementations.
	4. RAID/De-Dup(RAD):optional hardware acceleration support for RAID 3/4/5/6
	5. True Random Number Generator(RNG)

CMI:coherent memory interconnect	interconnect:使互相连接.

CPU Visible State Resident in each cnMIPS II Core:
	CRCIV<31:0> IV:interrupt vector

CSRs:Configuration and Status Registers
	CN63XX has many CSRs to control and configure the on-chip hardware.Each
	configuration register can be accessed by one or more of the following
	mechaisms:
	1. cores using MFC0/MTC0 instructions
	2. cores using MFC2/MTC2 instructions
	3. cores using ordinary load/store instructons to I/O physical addresses
	4. cores using indirect windowed accesses to PCIe configuration registe-
	   rs via the PEM*_CFG_WR and PEM*_CFG_RD CSRs.
	5. cores using indirect windowed accesses to SRIO maintenance registers
	   via the SRIO*_MAINT_OP and SRIO*_MAINT_RD_DATA CSRs.
	6. remote EJTAG/JTAG device using the EJTAG/JTAG TAP interface.
	7. remote PCIe/SRIO host using direct PCIe configuration accesses/SRIO 
	   maintenance accesses.
	8. remote host using direct PCIe/SRIO memory space BAR0 accesses.
	9. remote PCIe/SRIO host using indirect windowed accesses to I/O physic-
	   al addresses via the SLI_WIN_RD_ADDR,SLI_WIN_RD_DATA,SLI_WIN_WR_ADDR,
	   SLI_WIN_WR_DATA, and SLI_WIN_WR_MASK_CSRs.



























P88:instruction set summary

SEGBITS = 49, as per the MIPS specifications

KSEG3的范围0xFFFF FFFF FFFF 8000到0xFFFF FFFF FFFF BFFF称作CVMSEG段，两部分：
CVMSEG LM = 0xFFFF FFFF FFFF 8000 到 0xFFFF FFFF FFFF 9FFF
CVMSEG IO = 0xFFFF FFFF FFFF A000 到 0xFFFF FFFF FFFF BFFF
Chapter 3	Coherent Memory Interconnect and Level-2 Cache Controller
CMI Buses:
	The CMI is comprised of six buses:
		1.the address/command bus - ADD	
		2.the store data bus	- STORE
		3.the commit/response control bus - COMMIT
		4.the fill data bus	- FILL
		5.the I/O command/data bus - IOC
		6.the I/O response/data bus - IOR
	The CMI interface is 640bits wide,the LMC DRAM interface is 256 bits wi-
	de,and the internal L2 cache data interfaces are 256 bits in each quad,
	a total of 1024 bits including all quads.
	
	All data transfers between the L2C and the LMC move full 128-byte cache
	blocks.

Chapter 26	Boot Bus