<h1 style="color:re">
  External CPU Interface Connector
</h1>


J14D1 is connector for connecting External CPU Board.  
However, a separate cable for connection is not provided.  

Figure 1.1 shows the location of External CPU Interface Connector (J14D1).
<p align="center"><img src="https://github.com/Topst-Dev/Documentation/assets/161264431/bd36977b-a30d-49fc-bf69-0069ef321471"></p>
<p align="center"><strong>Figure 1.1 External CPU Interface Connector (J14D1)</strong></p>


Figure 1.2 shows the schematic of External CPU Interface Connector (J14D1).
<p align="center"><img src="https://github.com/Topst-Dev/Documentation/assets/161264431/b6b15a5e-ca36-4d29-b53f-a282fc981757"></p>
<p align="center"><strong>Figure 1.2 Schematic of External CPU Interface Connector (J14D1)</strong></p>

Table 1.1 shows the pin description of J14D1.  

**Table 1.1 J14D1 Pin Description:**  

|  Pin Number | TCC7045 Port Name | Signal Name | Dir(MCU◀▶J14D1)| Description                                |
|:-----------:|:-----------------:|:-----------:|:--------------:|----------------------------------------------|
| 1           |         -         | B_UP        |  -             | Power 12V                                    |
| 2           |         -         | B_UP        |  -             | Power 12V                                    |
| 3           |         -         | DGND        |  -             | Ground                                       |
| 4           |         -         | DGND        |  -             | Ground                                       |
| 5           | GPIO_A15          | GPIO_A15    |  ◀            | CPU_RSTOUT# : PMIC RSTOUT Control Signal     |
| 6           | GPIO_B10          | GPIO_B10    |  ▶            | CPU_PWR_EN  : CPU_PWR_EN Control Signal      |
| 7           | GPIO_B28          | GPIO_B28    |  ▶            | CPU_SYS_PWR_EN:CPU_SYS_PWR_EN Control Signal |
| 8           | GPIO_A29          | GPIO_A29    |  ▶            | CPU_RST# : CPU Reset Control                 |
| 9           | GPIO_K10          | GPIO_K10    |  ▶            | CPU_ALIVE_PWR_CTL : ALIVE_PWR Control Signal |
| 10          | GPIO_K14          | GPIO_K14    |  ▶            | CPU_STR_Module : CPU_STR_Mode Control        |
| 11          | GPIO_K13          | GPIO_K13    |  ◀            | GPIO Control Signal                          |
| 12          | GPIO_K03          | GPIO_K03    |  ◀            | GPIO Control Signal                          |
| 13          | GPIOA_C09         | SPI0_DO     |  ▶            | SPI Data input                               |
| 14          | GPIOA_C07         | SPI0_CLK    |  ▶            | SPI Clock                                    |
| 15          | GPIOA_C10         | SPI0_DI     |  ◀            | SPI Data Output                              |
| 16          | GPIOA_C08         | SPI0_CS     |  ▶            | SPI Chip Selection                           |
| 17          |        -          | DGND        |  -             | Ground                                       |
| 18          |        -          | DGND        |  -             | Ground                                       |
| 19          | GPIO_K15          | GPIO_K15    |  ◀            | GPIO Control Signal                          |
| 20          | GPIO_K12          | GPIO_K12    |  ◀            | GPIO Control Signal                          |
| 21          | GPIO_A23          | GPIO_A23    |  ▶            | GPIO Control Signal                          |
| 22          | GPIO_A22          | GPIO_A22    |  ◀            | GPIO Control Signal                          | 
| 23          | GPIO_A20          | UT4_TX      |  ▶            | UART Transmit                                |
| 24          | GPIO_A21          | UT4_RX      |  ◀            | UART Receive                                 |
| 25          | GPIO_B27          | GPIO_B27    |  ◀            | GPIO Control Signal                          |
| 26          | GPIO_K08          | GPIO_K08    |  ▶            | CAN0_TX                                      |                         
| 27          | GPIO_A14          | GPIO_A14    |  ▶            | LCD Backlight Enable Control                 |
| 28          | GPIO_K01          | GPIO_K01    |  ◀            | CAN0_RX                                      |

Table 1.2 shows the pin description of J14D1. 

**Table 1.2 J14D1 Pin Fuction:**  

|  No         | TCC7045 Port Name | Signal Name | IO Level | FUNC0        | FUNC1       | FUNC2       | FUNC3          | FUNC4(ANALOG) |
|:-----------:|:-----------------:|:-----------:|:--------:|:------------:|:-----------:|:-----------:|:--------------:|:-------------:|
| 1           |         -         | B_UP        | 3.3V     |       -      |       -     |      -      |        -       |       -       |
| 2           |         -         | B_UP        | 3.3V     |       -      |       -     |      -      |        -       |       -       |
| 3           |         -         | DGND        | 3.3V     |       -      |       -     |      -      |        -       |       -       |
| 4           |         -         | DGND        | 3.3V     |       -      |       -     |      -      |        -       |       -       |
| 5           | GPIO_A15          | GPIO_A15    | 3.3V     | GPIO_A[15]   | MAC_TXER    | PWM_OUT[05] |        -       |       -       |
| 6           | GPIO_B10          | GPIO_B10    | 3.3V     | GPIO_B[10]   | UT1_RTS_CH0 | PWM_OUT[04] | MFIO_2_CH1[02] |       -       |
| 7           | GPIO_B28          | GPIO_B28    | 3.3V     | GPIO_B[28]   | UT2_CTS_CH0 | PWM_OUT[08] |        -       |       -       | 
| 8           | GPIO_A29          | GPIO_A29    | 3.3V     | GPIO_A[29]   | UT0_RXD_CH0 | SNOR1_RST#  |        -       |       -       |
| 9           | GPIO_K10          | GPIO_K10    | 3.3V     | GPIO_K[10]   | CAN2_TX     | PWM_OUT[02] |        -       |       -       |
| 10          | GPIO_K14          | GPIO_K14    | 3.3V     | GPIO_K[14]   |       -     | PWM_OUT[06] | MFIO_0_CH3[03] |       -       | 
| 11          | GPIO_K13          | GPIO_K13    | 3.3V     | GPIO_K[13]   |       -     | PWM_OUT[05] | MFIO_0_CH3[02] |       -       |
| 12          | GPIO_K03          | GPIO_K03    | 3.3V     | GPIO_K[03]   | CAN2_RX     |      -      |        -       |       -       |
| 13          | GPIOA_C09         | SPI0_DO     | 3.3V     | GPIO_AC[09]  | GSDO0_CH1   | PWM_OUT[03] | MFIO_1_CH2[01] | AD1[09]       |
| 14          | GPIOA_C07         | SPI0_CLK    | 3.3V     | GPIO_AC[07]  | GSCK0_CH1   | PWM_OUT[01] | MFIO_0_CH2[03] | AD1[07]       |
| 15          | GPIOA_C10         | SPI0_DI     | 3.3V     | GPIO_AC[10]  | GSDI0_CH1   | PWM_OUT[04] | MFIO_1_CH2[02] | AD1[10]       |
| 16          | GPIOA_C08         | SPI0_CS     | 3.3V     | GPIO_AC[08]  | GSCMD0_CH1  | PWM_OUT[02] | MFIO_1_CH2[00] | AD1[08]       |
| 17          |        -          | DGND        | 3.3V     |       -      |       -     |      -      |        -       |       -       |
| 18          |        -          | DGND        | 3.3V     |       -      |       -     |      -      |        -       |       -       |
| 19          | GPIO_K15          | GPIO_K15    | 3.3V     | GPIO_K[15]   | WDI         | PWM_OUT[07] |        -       |       -       |
| 20          | GPIO_K12          | GPIO_K12    | 3.3V     | GPIO_K[12]   |       -     | PWM_OUT[04] | MFIO_0_CH3[01] |       -       |
| 21          | GPIO_A23          | GPIO_A23    | 3.3V     | GPIO_A[23]   | MAC_TXD[5]  |      -      | MFIO_1_CH0[03] |       -       |
| 22          | GPIO_A22          | GPIO_A22    | 3.3V     | GPIO_A[22]   | MAC_TXD[4]  |      -      | MFIO_1_CH0[02] |       -       | 
| 23          | GPIO_A20          | UT4_TX      | 3.3V     | GPIO_A[20]   | MAC_RXD[6]  |      -      | MFIO_1_CH0[00] |       -       |
| 24          | GPIO_A21          | UT4_RX      | 3.3V     | GPIO_A[21]   | MAC_RXD[7]  |      -      | MFIO_1_CH0[01] |       -       |
| 25          | GPIO_B27          | GPIO_B27    | 3.3V     | GPIO_B[27]   | UT2_RTS_CH0 | PWM_OUT[07] |        -       |       -       |
| 26          | GPIO_K08          | GPIO_K08    | 3.3V     | GPIO_K[08]   | CAN0_TX     | PWM_OUT[00] |        -       |       -       |
| 27          | GPIO_A14          | GPIO_A14    | 3.3V     | GPIO_A[14]   | MAC_RXER    | PWM_OUT[04] |        -       |       -       |
| 28          | GPIO_K01          | GPIO_K01    | 3.3V     | GPIO_K[01]   | CAN0_RX     |      -      |        -       |       -       |
