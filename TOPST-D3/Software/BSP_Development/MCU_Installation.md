#  MCU (CR5) Installation

## 1.1 Software Architecture

Figure 1.1 shows software components in TCC8050 MCU BSP. The TCC8050 MCU BSP is composed of device driver, Real Time Operating System (RTOS), System Adaptation Layer (SAL), and sample application. The MCU BSP is categorized into three types (Sample, 3rd party, and TC Support) by usage. The sample is a sample code for testing device driver. You can refer to the sample code while implementing your own solution, but quality is not guaranteed. The MCU BSP has the 3rd party software such as RTOS, but this MCU BSP does not provide technical support for the RTOS itself. The device driver and SAL interface are supported by Telechips (TC Support).

<p align="center">
    <img src="https://github.com/Topst-Dev/Documentation/assets/144076415/793aa74a-652f-4797-9b56-18579ac6d367" width="700" height="550">
</p>
<p align="center"><strong>Figure 1.1 Software Components in TCC8050 MCU BSP</strong></p>

## 1.2 Directory Structure

<p align="center">
    <img src="https://github.com/Topst-Dev/Documentation/assets/144076415/0bc19621-a3d6-4494-acaa-d901d0a83b57" width="300" height="350">
</p>
<p align="center"><strong>Figure 1.2 Directory Structure</strong></p>

① documents: All documents for TCC8050 MCU BSP

② scripts: Debug script for JTAG debugger such as Trace32

③ sources: Source code

1. app.drivers: Application drivers. These drivers do not access hardware directly.
2. app.sample: Sample applications
3. build: Build environment & utilities
4. core: Assembly core files
5. dev.drivers: Device drivers
6. os: Operating System
7. sal: System Adaptation Layer

④ tools: Tools for FWDN & FW upgrade

## 1.3 Build Information

The following describes how to build an MCU source.

After building the MCU source, use Firmware Downloader (***FWDN***) to create an image that can be loaded onto the TOPST D3 (Open platform board).

```bash
$ autolinux -c build cr5 clean

$ autolinux -c build cr5

Read configuration from autolinux.config

~/TOPST/cr5-bsp/sources/build/tcc805x/gcc ~/TOPST
make: Nothing to be done for 'all'.
~/Prj_TOPST/TOPST-20221005
 building on tcc8050-sub
=================================================================================
Built Path : /home/TOPST/cr5-bsp/sources/build/tcc805x/gcc/tcc805x-freertos-debug
=================================================================================
~/TOPST/boot-firmware/tools/snor_mkimage ~/TOPST
Get r5_fw.rom
make snor image ..................
MCERT file: (../../prebuilt/mcert.bin)
HSM binary file: (../../prebuilt/hsm.cs.bin)
R5-BL1 binary file: (../../prebuilt/r5_bl1-snor.rom)
update binary file: (../../prebuilt/r5_sub_fw.rom)
MICOM binary file: (./r5_fw.rom)
SNOR ROM size: (4 MByte)
R5 UART port enable: (1)
R5 UART port for debug: (46)
Output file: (../../../build/tcc8050-main/tmp/deploy/fwdn/cr5_snor.rom)

<<SNOR_MAP: 0x00002000 ++0x00002000>>
[Make SFMC Read CMD Header...]
Header Size: 8192 byte
(0) FAST READ CMD Set for Chipboot (SPI)
        Code:        0x000000E0
        Timing:      0x00140300
        Delay_s:     0x00000500
        Dc_clk:      0x00000000
        Run_mode:    0x00000011
        Read_cmd:    0x840000EB 0x4A000001 0x86000000 0x46002000 0x2A000000
        CMD CRC:     0x8E83D588
        CMD address: 0x00002000
write_sfmc_init_header - success

<<SNOR_MAP: 0x00004000 ++0x00002000>>
[Write Master Certificate...]
write_master_certificate - success

<<SNOR_MAP: 0x00006000 ++0x00010000>>
[Write HSM F/W Image ...]
write_hsm_image - success

<<SNOR_MAP: 0x00016000 ++0x00100000>>
[Write R5-BL1 Image ...]
write_r5_bl1_image - success

<<SNOR_MAP: 0x00116000 ++0x00010000>>
[Write Micom SubFW Image ...]
MICOM Sub f/w size: 0x7a00
MICOM Sub f/w target addr: 0x00116200
write_micom_sub_fw_image - success

<<SNOR_MAP: 0x001ff000 ++0x00200000>>
[Write Secure Micom FW Image ...]
MICOM ROM size: 0x6fdc0

<<SNOR_MAP: 0x00000000 ++0x00002000>>
[Make SNOR ROM Header...]
SNOR ROM Header CRC: 0x99A624CA
write_rom_header - success
SNOR ROM Header CRC: 0x5B90D383
write_rom_header - success
Total Image Size: 4194304 byte
create_final_rom - success

```


## 1.4 Feature Instructions

### 1.4.1 UART Support

The TCC8050 MICOM sub-system has a 5-channel UART, and each UART channel consists of one ARM® PrimeCell™ UART (PL011) controller and one ARM® PrimeCell™ DMA controller (PL081). Data transmitted/received via UART communication can be transferred from/to the internal memory (SRAM-0 or SRAM-1) by the UART dedicated DMA controller.

<p align="center">
    <img src="https://github.com/Topst-Dev/Documentation/assets/144076415/45c50597-0f5f-49f4-b5f9-028fac162f37" width="1000" height="850">
</p>
<p align="center"><strong>Figure 1.3 Block Diagram of UART</strong></p>


### 1.4.1.1 Features

### 1.4.1.1.1 UART Controller

The features of the UART controller are as follows:

- Use of programmable UART input/output
- Separate 32×8 transmits and 32×12 receives First-In, First-Out (FIFO) memory buffers to reduce CPU interrupts.
- Programmable FIFO disabling for 1-byte depth.
- Programmable baud rate generator. This enables pision of the reference clock frequency > 3.6864 MHz.
- Standard asynchronous communication bits (start, stop, and parity).
- Independent masking of transmit FIFO, receive FIFO, receive timeout, modem status, and error condition interrupts.
- Support Direct Memory Access (DMA).
- False start bit detection
- Line break generation and detection.
- Programmable hardware flow control: Modem control functions of CTS and RTS
- Fully-programmable serial interface characteristics:
- Data can be programmed in 5, 6, 7, or 8 bits
- Even, odd, stick, or no-parity bit generation and detection
- 1 or 2 stop bit generation
- Baud rate generation, up to UARTCLK/16
- Identification registers that uniquely identify the UART. These can be used by an operating system to automatically configure itself.

### 1.4.1.1.2 UART DMA Controller

The features of the UART DMA controller are as follows:

- Memory-to-memory, memory-to-UART, UART-to-memory transfers
- DMA controller has two channels.
- Scatter or gather DMA support through the use of linked lists. This means that the source and destination areas do not have to occupy contiguous areas of memory.
- One AHB bus master for transferring data. This interface transfers data when a DMA request goes a
- 32-bit AHB master bus width.
- Incrementing or non-incrementing addressing for source and destination.
- Programmable DMA burst size. The DMA burst size can be programmed to transfer data more efficiently. Usually, the burst size is set to half the size of the FIFO in the peripheral.
- Internal four-word FIFO per channel.
- Big-endian and little-endian support. The DMA controller defaults to little-endian mode on reset.
- Combined DMA error and DMA count interrupt requests. You can generate an interrupt to the processor on a DMA error or when a DMA count has reached 0. This is usually used to indicate that a transfer has finished. Three interrupt request signals do this:
- INTTC signals when a transfer is completed.

**Note**: The INTTC signal is not input to the interrupt controller, but status can be checked through the status register.

- INTERR signals when an error occurred.

**Note**: The INTERR signal is not input to the interrupt controller, but status can be checked through the status register.

- INTR combines both the INTTC and INTERR interrupt request signals. The INTR signal is input to the interrupt controller.
- Interrupt masking: You can mask the DMA error and DMA terminal count interrupt requests.
- Raw interrupt status: You can read the DMA error and DMA count raw interrupt status prior to masking.

### 1.4.1.2 Debug UART Initialize

The following source code is the basic information on how to set UART for MCU debugging.

<p align="center">
    <img src="https://github.com/Topst-Dev/Documentation/assets/144076415/672d844c-389c-4274-9370-4a3242d9373b" width="350" height="200">
    <img src="https://github.com/Topst-Dev/Documentation/assets/144076415/662f4f9a-c34d-4ba1-85ac-e4eab6caa8a2" width="600" height="500">
</p>

```bash
$ cat cr5-bsp/sources/dev.drivers/uart/tcc805x/uart.c

void UART_Init
(
    void
)
{
    // For LOG output
    (void)UART_Open(UART_CH0, GIC_PRIORITY_NO_MEAN, 115200UL, UART_POLLING_MODE, UART_CTSRTS_OFF,
                    46U, WORD_LEN_8, DISABLE_FIFO, TWO_STOP_BIT_OFF, PARITY_SPACE, NULL_PTR);
}

sint32 UART_Open
(
    uint8                               ucCh,
    uint32                              uiPriority,     // Interrupt Priority
    uint32                              uiBaudrate,     // Baudrate
    uint8                               ucMode,         // polling or interrupt or udma
    uint8                               ucCtsRts,       // on/off
    uint8                               ucPortCfg,      // port selection
    uint8                               ucWordLength,   // 5~6 bits
    uint8                               ucFIFO,         // on/off
    uint8                               uc2StopBit,     // on/off
    uint8                               ucParity,       // space, even, odd, mark
    GICIsrFunc                       fnCallback      // callback function
)
{
    SNORRomInfo_t   *rom_header;
    uint32          header_addr;
    uint8           port_cfg;   //config of debug port
    sint32          ret = -1;

    UART_StatusInit(ucCh);

    rom_header  = (SNORRomInfo_t *)MCU_BSP_SNOR_BASE;
    port_cfg  = 0;

    if (ucCh == UART_DEBUG_CH)
    {
        // Find Debug port from special location of SNOR Flash.
        if (rom_header->riRomId != SNOR_ROM_ID)
        {
            /* Secondary SNOR ROM Header */
            header_addr = (MCU_BSP_SNOR_BASE + 0x1000UL);
            rom_header  = (SNORRomInfo_t *)(header_addr);

            if (rom_header->riRomId != SNOR_ROM_ID)
            {
                port_cfg = ucPortCfg;
            }
            else
            {
                port_cfg = (uint8)rom_header->riDebugPort;
            }
        }
        else
        {
            port_cfg = (uint8)rom_header->riDebugPort;
        }
    }
    else if (ucCh < UART_CH_MAX)
    {
        port_cfg = ucPortCfg;
    }
    else
    {
        port_cfg = 0xFF;    //Error
    }

cr5-bsp/dev.drivers/uart/tcc805x/uart.c

static sint32 UART_SetPortConfig
(
    uint8                               ucCh,
    uint32                              uiPort
)
{
    uint32  idx;
    sint32  ret;
    static const UartBoardPort_t board_serial[UART_PORT_TBL_SIZE] =
    {
#if 0
        /* Coretex-R5 is not allowed to use the GPIO_SD0-2 ports */
        { 0UL,  TCC_GPSD0(11UL), TCC_GPSD0(12UL), TCC_GPSD0(13UL), TCC_GPSD0(14UL), GPIO_FUNC(7UL) },
        { 1UL,  TCC_GPSD1(0UL),  TCC_GPSD1(1UL),  TCC_GPSD1(2UL),  TCC_GPSD1(3UL),  GPIO_FUNC(7UL) },
        { 2UL,  TCC_GPSD1(6UL),  TCC_GPSD1(7UL),  TCC_GPSD1(8UL),  TCC_GPSD1(9UL),  GPIO_FUNC(7UL) },
        { 3UL,  TCC_GPSD2(0UL),  TCC_GPSD2(1UL),  TCC_GPSD2(2UL),  TCC_GPSD2(3UL),  GPIO_FUNC(7UL) },
        { 4UL,  TCC_GPSD2(6UL),  TCC_GPSD2(7UL),  TCC_GPNONE,    TCC_GPNONE,    GPIO_FUNC(7UL) },
#endif
        { 5UL,  GPIO_GPA(0UL),    GPIO_GPA(1UL),    GPIO_GPA(2UL),    GPIO_GPA(3UL),    GPIO_FUNC(7UL) },
        { 6UL,  GPIO_GPA(6UL),    GPIO_GPA(7UL),    TCC_GPNONE,       TCC_GPNONE,       GPIO_FUNC(7UL) },
        { 7UL,  GPIO_GPA(12UL),   GPIO_GPA(13UL),   TCC_GPNONE,       TCC_GPNONE,       GPIO_FUNC(7UL) },
        { 8UL,  GPIO_GPA(18UL),   GPIO_GPA(19UL),   TCC_GPNONE,       TCC_GPNONE,       GPIO_FUNC(7UL) },
        { 9UL,  GPIO_GPA(24UL),   GPIO_GPA(25UL),   GPIO_GPA(26UL),   GPIO_GPA(27UL),   GPIO_FUNC(7UL) },
        { 10UL, GPIO_GPB(0UL),    GPIO_GPB(1UL),    GPIO_GPB(2UL),    GPIO_GPB(3UL),    GPIO_FUNC(7UL) },
        { 11UL, GPIO_GPB(6UL),    GPIO_GPB(7UL),    TCC_GPNONE,       TCC_GPNONE,       GPIO_FUNC(7UL) },
        { 12UL, GPIO_GPB(12UL),   GPIO_GPB(13UL),   TCC_GPNONE,       TCC_GPNONE,       GPIO_FUNC(7UL) },
        { 13UL, GPIO_GPB(19UL),   GPIO_GPB(20UL),   GPIO_GPB(21UL),   GPIO_GPB(22UL),   GPIO_FUNC(7UL) },
        { 14UL, GPIO_GPB(25UL),   GPIO_GPB(26UL),   GPIO_GPB(27UL),   GPIO_GPB(28UL),   GPIO_FUNC(7UL) },
        { 15UL, GPIO_GPC(0UL),    GPIO_GPC(1UL),    GPIO_GPC(2UL),    GPIO_GPC(3UL),    GPIO_FUNC(7UL) },
        { 16UL, GPIO_GPC(4UL),    GPIO_GPC(5UL),    GPIO_GPC(6UL),    GPIO_GPC(7UL),    GPIO_FUNC(7UL) },
        { 17UL, GPIO_GPC(8UL),    GPIO_GPC(9UL),    GPIO_GPC(10UL),   GPIO_GPC(11UL),   GPIO_FUNC(7UL) },
        { 18UL, GPIO_GPC(10UL),   GPIO_GPC(11UL),   GPIO_GPC(12UL),   GPIO_GPC(13UL),   GPIO_FUNC(8UL) },
        { 19UL, GPIO_GPC(12UL),   GPIO_GPC(13UL),   GPIO_GPC(14UL),   GPIO_GPC(15UL),   GPIO_FUNC(7UL) },
        { 20UL, GPIO_GPC(16UL),   GPIO_GPC(17UL),   GPIO_GPC(18UL),   GPIO_GPC(19UL),   GPIO_FUNC(7UL) },
        { 21UL, GPIO_GPC(20UL),   GPIO_GPC(21UL),   GPIO_GPC(22UL),   GPIO_GPC(23UL),   GPIO_FUNC(7UL) },
        { 22UL, GPIO_GPC(26UL),   GPIO_GPC(27UL),   GPIO_GPC(28UL),   GPIO_GPC(29UL),   GPIO_FUNC(7UL) },
        { 23UL, GPIO_GPG(0UL),    GPIO_GPG(1UL),    GPIO_GPG(2UL),    GPIO_GPG(3UL),    GPIO_FUNC(7UL) },
        { 24UL, GPIO_GPG(7UL),    GPIO_GPG(8UL),    GPIO_GPG(9UL),    GPIO_GPG(10UL),   GPIO_FUNC(7UL) },
        { 25UL, GPIO_GPE(5UL),    GPIO_GPG(6UL),    GPIO_GPG(7UL),    GPIO_GPG(8UL),    GPIO_FUNC(7UL) },
        { 26UL, GPIO_GPE(11UL),   GPIO_GPE(12UL),   GPIO_GPG(13UL),   GPIO_GPG(14UL),   GPIO_FUNC(7UL) },
        { 27UL, GPIO_GPE(16UL),   GPIO_GPE(17UL),   GPIO_GPG(18UL),   GPIO_GPG(19UL),   GPIO_FUNC(7UL) },
        { 28UL, GPIO_GPH(4UL),    GPIO_GPH(5UL),    TCC_GPNONE,       TCC_GPNONE,       GPIO_FUNC(7UL) },
        { 29UL, GPIO_GPH(6UL),    GPIO_GPH(7UL),    TCC_GPNONE,       TCC_GPNONE,       GPIO_FUNC(7UL) },
        { 30UL, GPIO_GPH(0UL),    GPIO_GPH(1UL),    GPIO_GPH(2UL),    GPIO_GPH(3UL),    GPIO_FUNC(7UL) },
        { 31UL, GPIO_GPMA(0UL),   GPIO_GPMA(1UL),   GPIO_GPMA(2UL),   GPIO_GPMA(3UL),   GPIO_FUNC(7UL) },
        { 32UL, GPIO_GPMA(6UL),   GPIO_GPMA(7UL),   GPIO_GPMA(8UL),   GPIO_GPMA(9UL),   GPIO_FUNC(7UL) },
        { 33UL, GPIO_GPMA(12UL),  GPIO_GPMA(13UL),  GPIO_GPMA(14UL),  GPIO_GPMA(15UL),  GPIO_FUNC(7UL) },
        { 34UL, GPIO_GPMA(18UL),  GPIO_GPMA(19UL),  GPIO_GPMA(20UL),  GPIO_GPMA(21UL),  GPIO_FUNC(7UL) },
        { 35UL, GPIO_GPMA(24UL),  GPIO_GPMA(25UL),  GPIO_GPMA(26UL),  GPIO_GPMA(27UL),  GPIO_FUNC(7UL) },
        { 36UL, GPIO_GPK(9UL),    GPIO_GPK(10UL),   GPIO_GPK(11UL),   GPIO_GPK(12UL),   GPIO_FUNC(2UL) },
        { 37UL, GPIO_GPMB(0UL),   GPIO_GPMB(1UL),   GPIO_GPMB(2UL),   GPIO_GPMB(3UL),   GPIO_FUNC(7UL) },
        { 38UL, GPIO_GPMB(6UL),   GPIO_GPMB(7UL),   GPIO_GPMB(8UL),   GPIO_GPMB(9UL),   GPIO_FUNC(7UL) },
        { 39UL, GPIO_GPMB(12UL),  GPIO_GPMB(13UL),  GPIO_GPMB(14UL),  GPIO_GPMB(15UL),  GPIO_FUNC(7UL) },
        { 40UL, GPIO_GPMB(18UL),  GPIO_GPMB(19UL),  GPIO_GPMB(20UL),  GPIO_GPMB(21UL),  GPIO_FUNC(7UL) },
        { 41UL, GPIO_GPMB(24UL),  GPIO_GPMB(25UL),  GPIO_GPMB(26UL),  GPIO_GPMB(27UL),  GPIO_FUNC(7UL) },
        { 42UL, GPIO_GPMC(0UL),   GPIO_GPMC(1UL),   GPIO_GPMC(2UL),   GPIO_GPMC(3UL),   GPIO_FUNC(7UL) },
        { 43UL, GPIO_GPMC(6UL),   GPIO_GPMC(7UL),   GPIO_GPMC(8UL),   GPIO_GPMC(9UL),   GPIO_FUNC(7UL) },
        { 44UL, GPIO_GPMC(12UL),  GPIO_GPMC(13UL),  GPIO_GPMC(14UL),  GPIO_GPMC(15UL),  GPIO_FUNC(7UL) },
        { 45UL, GPIO_GPMC(18UL),  GPIO_GPMC(19UL),  GPIO_GPMC(20UL),  GPIO_GPMC(21UL),  GPIO_FUNC(7UL) },
        { 46UL, GPIO_GPMC(24UL),  GPIO_GPMC(25UL),  GPIO_GPMC(26UL),  GPIO_GPMC(27UL),  GPIO_FUNC(7UL) }
    };

...

}

```


### 1.4.1.3 Configuration Debug Port

Refer to Chapter 4 for information on how to set the PIN.

```bash
cr5-bsp/dev.drivers/common/debug.h

#define mcu_printf                          (DBG_SerialPrintf)

cr5-bsp/dev.drivers/common/debug.c

__attribute__ ((naked)) void DBG_SerialPrintf
(
    const int8 *                        format,
    ...
) 
{
    (void)format;
__asm volatile (
            "stmdb  sp!, {r1-r3}    \n"
            "mov    r1, r0          \n"     /* r1 = format string */
            "mov    r2, sp          \n"     /* r2 = arguments */
            "ldr    r0, =DBG_Putc     \n"     /* r0 = function pointer which will put character */
            "str    lr, [sp, #-4]!  \n"     /* save r14 */
            "bl     DBG_Printfi       \n"
            "ldr    lr, [sp], #4    \n"     /* restore r14 */
            "add    sp, sp, #12     \n"     /* restore stack */
            "mov    pc, lr          \n"
            :::"r0", "r1", "r2", "r3" \
    );
}

cr5-bsp/dev.drivers/common/debug.c

sint32 DBG_Putc
(
    sint8                               putChar
)
{
    if (putChar == (sint8)'\n')
    {
        (void)UART_PutChar((uint32)UART_DEBUG_CH, (uint8)'\r');
    }

    (void)UART_PutChar((uint32)UART_DEBUG_CH, (uint8)putChar);

    return 0;
}

```


### 1.4.1.4 External Configuration Debug Port

You can change the debug port without modifying the source by using the settings below.

```bash
$ cat boot-firmware/tools/snor_mkimage/tcc8050.cs.cfg

# R5 UART Port number for debug
DEBUG_ENABLE=1
DEBUG_PORT=46

```


The follwoing is the debug log for MCU (CR5).

```bash
[PMIO][PMIO_STR_SetTimer:1722] pmio Set Timer 600000000, reason 1
[PMIO][PMIO_STR_OpenMsg:1301] STR Msg Open : A72
[PMIO][PMIO_STR_OpenMsg:1307] STR Msg Open : A53

Initialize System done
Welcome to Telechips MCU BSP

MCU BSP Version: V1.0.4
(D)[SAL  ][FR_OSStart:481] ~ Done to initialize Free RT Operating System ~

[PMIO][PMIO_STR_RecvMsgTask:1476] recv: wait (PMIO_MSG_ACK for AppReady:1)

-------TCC_EVM_BD_VERSION == TCC8050_BD_VER_0_1----------
KEY_AppIPCHandleForSubAP
KEY_AppIPCHandleForMainAP

> help

================================================
============== How to use command ==============
================================================
 [command_string] [arg1] ... [arg10]

 1. display alive message : [alive] [on/off]

>

```


### 1.4.2 CAN Application

The Controller Area Network (CAN) performs communication according to ISO 11898-1 (identical to Bosch CAN protocol specification 2.0 part A, B). In addition, the CAN supports communication according to the CAN Flexible Data-rate (FD) protocol specification 1.0.

All functions handling messages are implemented by the Rx Handler and the Tx Handler.

The Rx Handler manages message acceptance filtering, the transfer of received messages from the CAN Core to the Message RAM as well as providing receive message status information.

The Tx Handler is responsible for the transfer of transmit messages from the Message RAM to the CAN Core as well as providing transmit status information. It implements all functions concerning the time schedule and the global system time.

Acceptance filtering is implemented by a combination of up to 128 filter elements where each one can be configured as a range, as a bit mask, or as a dedicated ID filter.

A total of three CAN FD controllers are integrated in the TCC8050 MICOM sub-system. The CPU can configure each CAN controller through AHB to APB host I/F. Each CAN FD controller accesses SRAM-0 or SRAM-1 integrated in MICOM sub-system through AHB bus matrix and uses SRAM-0 or SRAM-1 as message memory.

<p align="center">
    <img src="https://github.com/Topst-Dev/Documentation/assets/144076415/48657fe6-0d65-47a3-a437-80a9309f0742" width="1000" height="700">
</p>
<p align="center"><strong>Figure 1.4 Block Diagram of CAN FD Controller</strong></p>

### 1.4.2.1 Features

The features of the CAN application are as follows:

- Conform with CAN protocol version 2.0 part A, B (ISO 11898-1), Bosch CAN FD specification V1.0, ISO 16845
- CAN FD with up to 64 data bytes supported
- CAN Error Detection and Logging (Stuff/Bit/Format/CRC/ACK)
- Improved acceptance filtering
- Two configurable Receive FIFOs
- Separate signaling on reception of High Priority Messages
- Up to 64 dedicated Receive Buffers
- Up to 32 dedicated Transmit Buffers
- Configurable Transmit FIFO
- Configurable Transmit Queue
- Configurable Transmit Event FIFO
- Programmable loopback test mode
- Maskable module interrupts
- Two clock domains (CAN clock and bus clock)

<p align="center">
    <img src="https://github.com/Topst-Dev/Documentation/assets/144076415/816cf63d-fb5e-4ef0-9ebb-86304da9019c" width="350" height="350">
    <img src="https://github.com/Topst-Dev/Documentation/assets/144076415/a961f6b3-4fa6-482d-9014-8a923cf19907" width="600" height="400">
</p>

### 1.4.2.2 Create Task

```bash
cr5-bsp/app.sample/app.can.demo/can_demo.c

void CAN_DemoCreateApp
(
    void
)
{
    static uint32 uiCanDemoAppTaskID;
    static uint32 uiCanDemoAppTaskStk[CAN_DEMO_TASK_STK_SIZE];

    ( void ) SAL_TaskCreate( &uiCanDemoAppTaskID,
                   ( const uint8 * ) "Can Demo Task",
                   ( SALTaskFunc ) &CAN_DemoTask,
                   ( uint32 * const ) &uiCanDemoAppTaskStk[0],
                   CAN_DEMO_TASK_STK_SIZE,
                   SAL_PRIO_CAN_DEMO,
                   NULL_PTR);
}

static void CAN_DemoTask
(
    void *                              pArg
)
{
    uint8 ucCh;

    ( void ) pArg;

    while( 1 )
    {
        if( sTestInfo.tiRecv == TRUE )
        {
            for( ucCh = 0 ; ucCh < CAN_CONTROLLER_NUMBER ; ucCh++ )
            {
                CAN_DemoReceive( ucCh );
            }
        }

        ( void ) SAL_TaskSleep( 100 );
    }
}

```


### 1.4.2.3 CAN Driver

As shown in the CAN GPIO image above, apply GPIO PINs to CAN0, CAN1, and CAN2 port in the code below.

```bash
cr5-bsp/sources/dev.drivers/can/can_porting.h

    #define CAN_0_FS                        (PMIO_GPK(14)|PMIO_GPK(13)|PMIO_GPK(8)|PMIO_GPK(6))
    #define CAN_0_TX                        (GPIO_GPK(13UL))
    #define CAN_0_RX                        (GPIO_GPK(14UL))
    #define CAN_0_STB                       (GPIO_GPK(8UL))
    #define CAN_0_INH                       (GPIO_GPK(6UL))

    #define CAN_1_FS                        (PMIO_GPK(16)|PMIO_GPK(15))
    #define CAN_1_TX                        (GPIO_GPK(15UL))
    #define CAN_1_RX                        (GPIO_GPK(16UL))
    #define CAN_1_STB                       (GPIO_GPMC(0UL))
    #define CAN_1_INH                       (0UL) //No INH for CAN1

    #define CAN_2_FS                        (PMIO_GPK(18)|PMIO_GPK(17))
    #define CAN_2_TX                        (GPIO_GPK(17UL))
    #define CAN_2_RX                        (GPIO_GPK(18UL))
    #define CAN_2_STB                       (GPIO_GPMC(1UL))
    #define CAN_2_INH                       (0UL) //No INH for CAN2

cr5-bsp/dev.drivers/can/can_drv.c

CANErrorType_t CAN_DrvInitChannel
(
    CANController_t *                   psControllerInfo
)
{
    uint8               ucCh;
    uint32              uiMemSize;
    uint32              uiMemAddr;
    CANRegBaseAddr_t *  psConfigBaseRegAddr;
    CANErrorType_t      ret;

    uiMemAddr           = 0UL;
    ret                 = CAN_ERROR_NONE;

    if( ( psControllerInfo != NULL_PTR ) && ( psControllerInfo->cMode == CAN_MODE_NO_INITIALIZATION ) )
    {
        /* Set Config */
        ucCh = psControllerInfo->cChannelHandle;
        psControllerInfo->cRegister = ( CANControllerRegister_t * ) CAN_PortingGetControllerRegister( ucCh );

        /* Set HW_Init */
        ( void ) CAN_PortingInitHW( psControllerInfo );
        ( void ) CAN_PortingSetControllerClock( psControllerInfo, CAN_CONTROLLER_CLOCK );
        ( void ) CAN_PortingResetDriver( psControllerInfo );

        /* Set Memory */
        uiMemSize = CAN_DrvGetSizeOfRamMemory( psControllerInfo );

        if( 0UL < uiMemSize )
        {
            uiMemAddr = CAN_PortingAllocateNonCacheMemory( ucCh, uiMemSize );
        }

        if( uiMemAddr != 0UL )
        {
            psConfigBaseRegAddr = ( CANRegBaseAddr_t * ) CAN_PortingGetMessageRamBaseAddr( ucCh );

            if( psConfigBaseRegAddr != NULL_PTR )
            {
                psConfigBaseRegAddr->rFReg.rfBASE_ADDR = ( SALReg32 ) ( ( uint32 ) ( uiMemAddr >> ( uint32 ) 16UL ) & ( uint32 ) 0xFFFFUL );
            }

            ( void ) CAN_DrvRegisterParameterAll( psControllerInfo );

            ( void ) CAN_DrvStartConfigSetting( psControllerInfo );
            ( void ) CAN_DrvSetControlConfig( psControllerInfo );

            /* Set Buffer */
            ( void ) SAL_MemSet( ( void * ) uiMemAddr, 0, uiMemSize );
            ( void ) CAN_DrvInitBuffer( psControllerInfo, uiMemAddr, uiMemSize );
            ( void ) CAN_MsgInit( psControllerInfo );

            /* Set Filter */
            ( void ) CAN_DrvSetFilterConfig( psControllerInfo );

            /* Set Timing */
            ( void ) CAN_DrvInitTiming( psControllerInfo );

            /* Set Interrupt */
            ( void ) CAN_DrvSetInterruptConfig( psControllerInfo );

            /* Set TimeStamp */
            ( void ) CAN_DrvSetTimeStamp( psControllerInfo );

            /* Set TimeOut */
            ( void ) CAN_DrvSetTimeoutValue( psControllerInfo );

            /* Set WatchDog */
            ( void ) CAN_DrvSetWatchDog( psControllerInfo );

            ( void ) CAN_DrvFinishConfigSetting( psControllerInfo );

            psControllerInfo->cMode = CAN_MODE_OPERATION;
        }
        else
        {
            ret = CAN_ERROR_ALLOC;
        }
    }
    else
    {
        ret = CAN_ERROR_BAD_PARAM;
    }

    return ret;
}

cr5-bsp/dev.drivers/can/tcc805x/can_porting.c

CANErrorType_t CAN_PortingInitHW
(
    const CANController_t *             psControllerInfo
)
{
    CANErrorType_t ret;

    ret = CAN_ERROR_NONE;

    if( psControllerInfo != NULL_PTR )
    {
        switch( psControllerInfo->cChannelHandle )
        {
            case 0:
            {
                PMIO_SetGPK(CAN_0_FS); //Set to normal k-port

                ( void ) GPIO_Config( CAN_0_TX,    ( uint32 ) ( GPIO_FUNC( 2U ) | GPIO_OUTPUT ) ); //can0 tx
                ( void ) GPIO_Config( CAN_0_RX,    ( uint32 ) ( GPIO_FUNC( 2U ) | GPIO_INPUT ) ); //can0 rx
                ( void ) GPIO_Config( CAN_0_STB,   ( uint32 ) ( GPIO_FUNC( 0U ) | GPIO_OUTPUT ) ); //can0 stb
                ( void ) GPIO_Config( CAN_0_INH,   ( uint32 ) ( GPIO_FUNC( 0U ) | GPIO_INPUT ) ); //can0 INH

                ( void ) GPIO_Set( CAN_0_STB, 0UL ); //Set to Low

                break;
            }

            case 1:
            {
                PMIO_SetGPK(CAN_1_FS); //Set to normal k-port

                ( void ) GPIO_Config( CAN_1_TX,    ( uint32 ) ( GPIO_FUNC( 2U ) | GPIO_OUTPUT ) ); //can1 tx
                ( void ) GPIO_Config( CAN_1_RX,    ( uint32 ) ( GPIO_FUNC( 2U ) | GPIO_INPUT ) ); //can1 rx

                /* //CAN_1_STB is not used on EVB
                ( void ) GPIO_Config( CAN_1_STB,   ( uint32 ) ( GPIO_FUNC( 0U ) | GPIO_OUTPUT ) ); //can1 stb

                ( void ) GPIO_Set( CAN_1_STB, 0UL ); //Set to Low
                */

                break;
            }

            case 2:
            {
                PMIO_SetGPK(CAN_2_FS); //Set to normal k-port

                ( void ) GPIO_Config( CAN_2_TX,     ( uint32 ) ( GPIO_FUNC( 2U ) | GPIO_OUTPUT ) ); //can2 tx
                ( void ) GPIO_Config( CAN_2_RX,     ( uint32 ) ( GPIO_FUNC( 2U ) | GPIO_INPUT ) ); //can2 rx

                /* //CAN_2_STB is not used on EVB
                ( void ) GPIO_Config( CAN_2_STB,    ( uint32 ) ( GPIO_FUNC( 0U ) | GPIO_OUTPUT ) ); //can2 stb

                ( void ) GPIO_Set( CAN_2_STB, 0UL ); //Set to Low
                */

                break;
            }

            default:
            {
                ret = CAN_ERROR_BAD_PARAM;

                break;
            }
        }
    }
    else
    {
        ret = CAN_ERROR_NOT_INIT;
    }

    return ret;
}
```


### 1.4.2.4 Receive CAN

```bash
cr5-bsp/dev.drivers/can/can_drv.c

static void CAN_DemoReceive
(
    uint8                               ucCh
)
{

…

    if( 0UL < gReceiveFlag[ ucCh ] )
    {
        while( 1 )
        {
            uiRxMsgNum = CAN_CheckNewRxMessage( ucCh );

            if( 0UL < uiRxMsgNum )
            {
                ( void ) CAN_GetNewRxMessage( ucCh, &sRxMsg );

                 …

}
}
}
}
```


### 1.4.2.5 Send CAN

```bash
cr5-bsp/dev.drivers/can/can_drv.c

static void CAN_DemoSend
(
    uint8                               ucCh
)
{

…

    for( uiTxMsgCnt = 0UL ; uiTxMsgCnt < CAN_MAX_TEST_MSG_NUM ; uiTxMsgCnt++ )
    {
        uiTimeout = 10; //ms

        /* Fill data */
        for( ucTxMsgDlc = 0U ; ucTxMsgDlc < sTxMessageInfo[ uiTxMsgCnt ].mDataLength ; ucTxMsgDlc++ )
        {
            sTxMessageInfo[ uiTxMsgCnt ].mData[ ucTxMsgDlc ] = ucTxMsgDlc;
        }

        /* Send Tx message */
        psTxMsg = &sTxMessageInfo[ uiTxMsgCnt ];

        ( void ) CAN_SendMessage( ucCh, psTxMsg, &ucTxBufferIndex );
         
…

}
    }
}

cr5-bsp/sources/app.sample/app.can.demo/can_demo.c

static CANMessage_t sTxMessageInfo[CAN_MAX_TEST_MSG_NUM] =
{
#if 1  // TEST :: CAN Format
   /* BufferType,                 Index, ESI, ExtendedID, RTR, ID,    FD, BRS, MM,   EventFIFO, DLC, DATA */
    { CAN_TX_BUFFER_TYPE_DBUFFER, 0,     0,   0,          0,   0x11,  0,  0,   0xFF, 1,         1,   {0, /* Data is definded as much as DLC in sending function */} },
    { CAN_TX_BUFFER_TYPE_DBUFFER, 1,     0,   1,          0,   0x22,  0,  0,   0xFF, 1,         2,   {0, /* Data is definded as much as DLC in sending function */} },
    { CAN_TX_BUFFER_TYPE_DBUFFER, 2,     0,   0,          0,   0x33,  0,  0,   0xFF, 1,         3,   {0, /* Data is definded as much as DLC in sending function */} },
    { CAN_TX_BUFFER_TYPE_DBUFFER, 3,     0,   1,          0,   0x44,  0,  0,   0xFF, 1,         4,   {0, /* Data is definded as much as DLC in sending function */} },
    { CAN_TX_BUFFER_TYPE_FIFO,    0,     0,   0,          0,   0x55,  0,  0,   0xFF, 1,         5,   {0, /* Data is definded as much as DLC in sending function */} },
    { CAN_TX_BUFFER_TYPE_FIFO,    0,     0,   1,          0,   0x66,  0,  0,   0xFF, 1,         6,   {0, /* Data is definded as much as DLC in sending function */} },
    { CAN_TX_BUFFER_TYPE_FIFO,    0,     0,   0,          0,   0x77,  0,  0,   0xFF, 1,         7,   {0, /* Data is definded as much as DLC in sending function */} },
    { CAN_TX_BUFFER_TYPE_FIFO,    0,     0,   1,          0,   0x88,  0,  0,   0xFF, 1,         8,   {0, /* Data is definded as much as DLC in sending function */} },
    { CAN_TX_BUFFER_TYPE_FIFO,    0,     0,   0,          0,   0x99,  0,  0,   0xFF, 1,         8,   {0, /* Data is definded as much as DLC in sending function */} },
#else   // TEST :: CAN FD Format
   /* BufferType,                 Index, ESI, ExtendedID, RTR, ID,    FD, BRS, MM,   EventFIFO, DLC, DATA */
    { CAN_TX_BUFFER_TYPE_DBUFFER, 0,     0,   0,          0,   0x11,  1,  1,   0xFF, 1,         1,   {0, /* Data is definded as much as DLC in sending function */} },
    { CAN_TX_BUFFER_TYPE_DBUFFER, 1,     0,   1,          0,   0x22,  1,  1,   0xFF, 1,         2,   {0, /* Data is definded as much as DLC in sending function */} },
    { CAN_TX_BUFFER_TYPE_DBUFFER, 2,     0,   0,          0,   0x33,  1,  1,   0xFF, 1,         3,   {0, /* Data is definded as much as DLC in sending function */} },
    { CAN_TX_BUFFER_TYPE_DBUFFER, 3,     0,   1,          0,   0x44,  1,  1,   0xFF, 1,         4,   {0, /* Data is definded as much as DLC in sending function */} },
    { CAN_TX_BUFFER_TYPE_FIFO,    0,     0,   0,          0,   0x55,  1,  1,   0xFF, 1,         5,   {0, /* Data is definded as much as DLC in sending function */} },
    { CAN_TX_BUFFER_TYPE_FIFO,    0,     0,   1,          0,   0x66,  1,  1,   0xFF, 1,         6,   {0, /* Data is definded as much as DLC in sending function */} },
    { CAN_TX_BUFFER_TYPE_FIFO,    0,     0,   0,          0,   0x77,  1,  1,   0xFF, 1,         7,   {0, /* Data is definded as much as DLC in sending function */} },
    { CAN_TX_BUFFER_TYPE_FIFO,    0,     0,   1,          0,   0x88,  1,  1,   0xFF, 1,         8,   {0, /* Data is definded as much as DLC in sending function */} },
    { CAN_TX_BUFFER_TYPE_FIFO,    0,     0,   0,          0,   0x99,  1,  1,   0xFF, 1,         12,  {0, /* Data is definded as much as DLC in sending function */} },
#endif
};
```


### 1.4.3 Console Application

### 1.4.3.1 Create Task

```bash
cr5-bsp/app.sample/app.console/console.c

void CreateConsoleTask(void)
{
    static uint32   ConsoleTaskID = 0UL;
    static uint32   ConsoleTaskStk[CSL_TASK_STK_SIZE];

    (void)SAL_TaskCreate
    (
        &ConsoleTaskID,
        (const uint8 *)"Console Task",
        (SALTaskFunc)&ConsoleTask,
        &ConsoleTaskStk[0],
        CSL_TASK_STK_SIZE,
        SAL_PRIO_LOWEST,
        NULL
    );
}

cr5-bsp/app.sample/app.console/console.c

static void ConsoleTask(void *pArg)
{
    uint8   cmdBuffer[CSL_CMD_BUFF_SIZE + CSL_CMD_PROMPT_SIZE];
    uint32  cmdLength   = 0;
    uint8   cmdStatus   = (uint8)CSL_NOINPUT;
    sint8   getc_err;
    sint32  ret         =0;
    uint8   c           =0;
    uint32  idx;

    (void)pArg;

    for(;;)
    {
        ret = UART_GetChar(UART_CH0, 0, (sint8 *)&getc_err);

        if(ret > 0)
        {
            c   = ((uint8)ret & 0xFFUL);

            switch(c)
            {
                case ARRIAGE_RETURN:
                case LINE_FEED:
                case BACK_SPACE:
                case NAK_KEY:
                case ESC_KEY:
UART_Write(UART_CH0, (const uint8 *)"\b \b", 3UL);
                    break;
                      ….
}
```


### 1.4.3.2 Console Command

```bash
cr5-bsp/app.sample/app.console/console.c

static void CSL_HelpList(uint8 ucArgc, void *pArgv[])
{
    (void)  ucArgc;
    (void)  pArgv;

    UART_Write(UART_CH0, (const uint8 *)"\r\n================================================", 50);
    UART_Write(UART_CH0, (const uint8 *)"\r\n============== How to use command ==============", 50);
    UART_Write(UART_CH0, (const uint8 *)"\r\n================================================", 50);

    UART_Write(UART_CH0, (const uint8 *)"\r\n [command_string] [arg1] ... [arg10]", 38);
    UART_Write(UART_CH0, (const uint8 *)"\n", 1);
    UART_Write(UART_CH0, (const uint8 *)"\r\n 1. display alive message : [alive] [on/off]", 46);

    UART_Write(UART_CH0, (const uint8 *)"\n", 1);
}

static ConsoleCmdList_t ConsoleCmdList[CSL_CMD_NUM_MAX] =
{
    { CMD_ENABLE,   (const uint8 *)"help",     CSL_HelpList},
    { CMD_ENABLE,   (const uint8 *)"md",       CSL_ReadMemory},
    { CMD_ENABLE,   (const uint8 *)"mm",       CSL_WriteMemory},
    { CMD_ENABLE,   (const uint8 *)"alive",    CSL_SetAliveMessage},
    { CMD_ENABLE,   (const uint8 *)"i2c",      CSL_DeviceI2c},
    { CMD_ENABLE,   (const uint8 *)"uart",     CSL_DeviceUart},
    { CMD_ENABLE,   (const uint8 *)"can",      CSL_DeviceCan},
    { CMD_ENABLE,   (const uint8 *)"lin",      CSL_DeviceLin},
    { CMD_ENABLE,   (const uint8 *)"mbox",     CSL_DeviceMbox},
    { CMD_ENABLE,   (const uint8 *)"pdm",      CSL_DevicePdm},
    { CMD_ENABLE,   (const uint8 *)"ictc",     CSL_DeviceIctc},
    { CMD_ENABLE,   (const uint8 *)"adc",      CSL_DeviceAdc},
    { CMD_ENABLE,   (const uint8 *)"gpio",     CSL_DeviceGpio},
    { CMD_ENABLE,   (const uint8 *)"tmr",      CSL_DeviceTmr},
    { CMD_ENABLE,   (const uint8 *)"wdt",      CSL_DeviceWdt},
    { CMD_ENABLE,   (const uint8 *)"mpc",      CSL_DeviceMpu},
    { CMD_ENABLE,   (const uint8 *)"fmu",      CSL_DeviceFmu},
    { CMD_ENABLE,   (const uint8 *)"gpsb",     CSL_DeviceGpsb},
    { CMD_ENABLE,   (const uint8 *)"gic",      CSL_DeviceGic},
    { CMD_ENABLE,   (const uint8 *)"pmio",     CSL_DevicePmio},
    { CMD_ENABLE,   (const uint8 *)"ckc",      CSL_DeviceCkc},
    { CMD_ENABLE,   (const uint8 *)"gdma",     CSL_DeviceGdma},
    { CMD_ENABLE,   (const uint8 *)"sdm",      CSL_DeviceSdm},
    { CMD_ENABLE,   (const uint8 *)"dse",      CSL_DeviceDse},
    { CMD_ENABLE,   (const uint8 *)"ssm",      CSL_DeviceSsm},
    { CMD_ENABLE,   (const uint8 *)"midf",     CSL_DeviceMidf},
    { CMD_ENABLE,   (const uint8 *)"log",      CSL_EnableLog},
    { CMD_ENABLE,   (const uint8 *)"hsm",      CSL_DeviceHsm},
    { CMD_ENABLE,   (const uint8 *)"trvc",     CSL_DeviceTrvc},
    { CMD_ENABLE,   (const uint8 *)"pmu",      CSL_DevicePmu},
    { CMD_ENABLE,   (const uint8 *)"top",      CSL_DeviceTOP},
    { CMD_ENABLE,   (const uint8 *)"fwug",     CSL_DeviceFwug},
    { CMD_DISABLE,  (const uint8 *)"",         NULL}
};
```
