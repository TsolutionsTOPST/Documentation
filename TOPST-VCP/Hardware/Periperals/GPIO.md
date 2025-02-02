<h1 style="color:red">
  Standard 40-pin GPIO header
</h1>


J20D1 is a standard 40-pin GPIO header.  

Figure 1.1 shows the location of External CPU Interface Connector (J20D1).
<p align="center"><img src="https://github.com/Topst-Dev/Documentation/assets/161264431/6e00142a-4a69-4585-b0a8-2ff8d758e20c"></p>
<p align="center"><strong>Figure 1.1 Standard 40-pin GPIO header</strong></p>

Figure 1.2 shows the schematic of a standard 40-pin GPIO header.
<p align="center"><img src="https://github.com/Topst-Dev/Documentation/assets/161264431/22966921-c736-4a83-9d38-ced61475e9e7"></p>
<p align="center"><strong>Figure 1.2 Schematic of a standard 40-pin GPIO header</strong></p>

Table 1.1 shows the pin description of J20D1.  

**Table 1.1 40-Pin Description:**  

|  Pin Number | TCC7045 Port Name | Signal Name | Dir(MCU◀▶J20D1)| Description                                   |
|:-----------:|:-----------------:|:-----------:|:--------------:|--------------|
| 1           |         -         | SYS_3P3     |       -        | Power 3.3V      |
| 2           |         -         | B_5P0       |       -        | Power 5.0V |
| 3           | GPIO_B01          | I2C0_SDA    |      ◀▶      | I2C data       |
| 4           |         -         | B_5P0       |       -        | Power 5.0V       |
| 5           | GPIO_B00          | I2C0_SCL    |  ▶            | I2C Clock     |
| 6           |         -         | DGND        |       -        | Ground        |
| 7           | GPIO_B04          | GPIO_B04    |  ◀            | GPIO Signal  |
| 8           | GPIO_B25          | UT2_Tx      |  ▶            | UART Transmit      |
| 9           |         -         | DGND        |       -        | Ground      |
| 10          | GPIO_B26          | UT2_Rx      |  ▶            | UART Receive        |
| 11          | GPIO_B06          | GPIO_B06    |  ◀            | GPIO Signal     |
| 12          | GPIO_A16          | GPIO_A16    |  ◀            | GPIO Signal      |
| 13          | GPIO_B23          | GPIO_B23    |  ◀            | GPIO Signal       |
| 14          |         -         | DGND        |       -        | Ground     |
| 15          | GPIO_K11          | GPIO_K11    |  ◀            | GPIO Signal        |
| 16          | GPIO_AC04         | GPIO_AC04   |  ◀            | GPIO Signal       |
| 17          |         -         | SYS_3P3     |       -        | Power 3.3V    |
| 18          | GPIO_AC05         | GPIO_AC05   |  ◀            | GPIO Signal        |
| 19          | GPIO_A24          | SPI3_SDO    |  ▶            | SPI Data Output         |
| 20          |         -         | DGND        |       -        | Ground         |
| 21          | GPIO_A25          | SPI3_SDI    |  ◀            | SPI Data Input          |
| 22          | GPIO_K09          | GPIO_K09    |  ◀            | GPIO Signal          | 
| 23          | GPIO_A27          | SPI3_CLK    |  ▶            | SPI Clock           |
| 24          | GPIO_A26          | SPI3_CS     |  ▶            | SPI Chip Selection               |
| 25          |         -         | DGND        |       -        | Ground              |
| 26          | GPIO_C13          | SPI1_CS     |  ▶            | SPI Chip Selection           |
| 27          | GPIO_B03          | I2C1_SDA    |  ◀▶          | I2C data            |
| 28          | GPIO_B02          | I2C1_SCL    |  ▶            | I2C Clock          |
| 29          | GPIO_A29          | GPIO_A29    |  ◀            | GPIO Signal          |
| 30          |         -         | DGND        |       -        | Ground          |
| 31          | GPIO_B19          | GPIO_B19    |  ◀            | GPIO Signal          |
| 32          | GPIO_B11          | GPIO_B11    |  ◀            | GPIO Signal          |
| 33          | GPIO_C11          | GPIO_C11    |  ◀            | GPIO Signal          |
| 34          |         -         | DGND        |       -        | Ground          |
| 35          | GPIO_C15          | SPI1_DI     |  ◀            | SPI Data Input         |
| 36          | GPIO_B18          | GPIO_B18    |  ◀            | GPIO Signal          |
| 37          | GPIO_A18          | GPIO_A18    |  ◀            | GPIO Signal          |
| 38          | GPIO_C14          | SPI1_DO     |  ▶            | SPI Data Output          |
| 39          |         -         | DGND        |       -        | Ground           |
| 40          | GPIO_C12          | SPI1_CLK    |  ▶            | SPI Clock          |

Table 1.2 shows the pin description of J20D1. 

**Table 1.2 40-Pin PinFuction:**  

|  No         | TCC7045 Port Name | Signal Name | IO Level | FUNC0        | FUNC1       | FUNC2       | FUNC3          | FUNC4(ANALOG) |
|:-----------:|:-----------------:|:-----------:|:--------:|:------------:|:-----------:|:-----------:|:--------------:|:-------------:|
| 1           |         -         | SYS_3P3     | 3.3V     |      -       |      -      |      -      |        -       |       -       |
| 2           |         -         | B_5P0       | 3.3V     |      -       |      -      |      -      |        -       |       -       |
| 3           | GPIO_B01          | I2C0_SDA    | 3.3V     | GPIO_B[01]   | I2C0_SDA_CH0| I2S_BCLK_CH2| MFIO_0_CH1[01] |       -       |
| 4           |         -         | B_5P0       | 3.3V     |      -       |      -      |      -      |        -       |       -       |
| 5           | GPIO_B00          | I2C0_SCL    | 3.3V     | GPIO_B[00]   | I2C0_SCL_CH0| I2S_MCLK_CH2| MFIO_0_CH1[00] |       -       |
| 6           |         -         | DGND        | 3.3V     |      -       |      -      |      -      |        -       |       -       |
| 7           | GPIO_B04          | GPIO_B04    | 3.3V     | GPIO_B[04]   | GSCK0_CH0   | I2S_DAI_CH2 | MFIO_1_CH1[00] |       -       | 
| 8           | GPIO_B25          | UT2_Tx      | 3.3V     | GPIO_B[25]   | UT2_TXD_CH0 | TCO[05]     |        -       |       -       |
| 9           |         -         | DGND        | 3.3V     |      -       |      -      |      -      |        -       |       -       |
| 10          | GPIO_B26          | UT2_Rx      | 3.3V     | GPIO_B[26]   | UT2_RXD_CH0 | PWM_OUT[06] |        -       |       -       | 
| 11          | GPIO_B06          | GPIO_B06    | 3.3V     | GPIO_B[06]   | GSDO0_CH0   | PWM_OUT[00] | MFIO_1_CH1[02] |       -       |
| 12          | GPIO_A16          | GPIO_A16    | 3.3V     | GPIO_A[16]   | MAC_COL     | PWM_OUT[06] | MFIO_0_CH0[00] |       -       |
| 13          | GPIO_B23          | GPIO_B23    | 3.3V     | GPIO_B[23]   | I2S_DAI_CH0 | TCO[03]     |        -       |       -       |
| 14          |         -         | DGNDK       | 3.3V     |       -      |      -      |      -      |        -       |       -       |
| 15          | GPIO_K11          | GPIO_K11    | 3.3V     | GPIO_K[11]   |      -      | PWM_OUT[03] | MFIO_0_CH3[00] |       -       |
| 16          | GPIO_AC04         | GPIO_AC04   | 3.3V     | GPIO_AC[04]  |      -      | I2C2_SCL_CH1| MFIO_0_CH2[00] | AD1[04]       |
| 17          |         -         | SYS_3P3     | 3.3V     |      -       |      -      |      -      |        -       |       -       |
| 18          | GPIO_AC05         | GPIO_AC05   | 3.3V     | GPIO_AC[05]  |      -      | I2C2_SDA_CH1| MFIO_2_CH0[01] | AD1[05]              |
| 19          | GPIO_A24          | SPI3_SDO    | 3.3V     | GPIO_A[24]   | MAC_TXD[6]  | I2C2_SCL_CH0| MFIO_2_CH0[00] |       -       |
| 20          |         -         | DGND2       | 3.3V     |      -       |      -      |      -      |        -       |       -       |
| 21          | GPIO_A25          | SPI3_SDI    | 3.3V     | GPIO_A[25]   | CAN1_TX     | I2S_MCLK_CH1| MFIO_2_CH0[01] |       -       |
| 22          | GPIO_K09          | GPIO_K09    | 3.3V     | GPIO_K[09]   | MAC_TXRSTN  | PWM_OUT[01] |        -       |       -       | 
| 23          | GPIO_A27          | SPI3_CLK    | 3.3V     | GPIO_A[27]   | MAC_RXRSTN  | I2S_BCLK_CH1| MFIO_2_CH0[03] |       -       |
| 24          | GPIO_A26          | SPI3_CS     | 3.3V     | GPIO_A[26]   | MAC_RXD[7]  | I2C2_SDA_CH0| MFIO_2_CH0[02] |       -       |
| 25          |         -         | DGND        | 3.3V     |      -       |      -      |      -      |        -       |       -       |
| 26          | GPIO_C13          | SPI1_CS     | 3.3V     | GPIO_C[13]   | GSCMD1_CH1  | PWM_OUT[07] | MFIO_2_CH2[01] |       -       |
| 27          | GPIO_B03          | I2C1_SDA    | 3.3V     | GPIO_B[03]   | I2C1_SDA_CH0| I2S_DAO_CH2 | MFIO_0_CH1[03] |       -       |
| 28          | GPIO_B02          | I2C1_SCL    | 3.3V     | GPIO_B[02]   | I2C1_SCL_CH0| I2S_LRCK_CH2| MFIO_0_CH1[02] |       -       |
| 29          | GPIO_A29          | GPIO_A29    | 3.3V     | GPIO_A[29]   | UT0_RXD_CH0 | SNOR1_RST#  |        -       |       -       |
| 30          |         -         | DGND        | 3.3V     |      -       |      -      |      -      |        -       |       -       |
| 31          | GPIO_B19          | GPIO_B19    | 3.3V     | GPIO_B[19]   |      -      | TCO[08]     |        -       |       -       |
| 32          | GPIO_B11          | GPIO_B11    | 3.3V     | GPIO_B[11]   | UT1_CTS_CH0 | PWM_OUT[05] | MFIO_2_CH1[03] |       -       |
| 33          | GPIO_C11          | GPIO_C11    | 3.3V     | GPIO_C[11]   |      -      | PWM_OUT[05] | MFIO_1_CH2[03] |       -       |
| 34          |         -         | DGND        | 3.3V     |      -       |      -      |             |        -       |       -       |
| 35          | GPIO_C15          | SPI1_DI     | 3.3V     | GPIO_C[15]   | GSDI1_CH1   | TCO[09]     | MFIO_2_CH2[03] |       -       |
| 36          | GPIO_B18          | GPIO_B18    | 3.3V     | GPIO_B[18]   | SNOR1_DQS   | TCO[07]     |        -       |       -       |
| 37          | GPIO_A18          | GPIO_A18    | 3.3V     | GPIO_A[18]   | MAC_RXD[4]  | PWM_OUT[08] | MFIO_0_CH0[02] |       -       |
| 38          | GPIO_C14          | SPI1_DO     | 3.3V     | GPIO_C[14]   | GSDO1_CH1   | PWM_OUT[08] | MFIO_2_CH2[02] |       -       |
| 39          |         -         | DGND        | 3.3V     |      -       |      -      |      -      |        -       |       -       |
| 40          | GPIO_C12          | SPI1_CLK    | 3.3V     | GPIO_C[12]   | GSCK1_CH1   | PWM_OUT[06] | MFIO_2_CH2[00] |       -       |
