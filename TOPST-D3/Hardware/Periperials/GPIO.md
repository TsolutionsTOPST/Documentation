<h1>
  GPIO
</h1>


## GPIO Interface  

The TOPST D3 (Open platform board) provides 28 GPIOs via 40 pin header that is backwards compatible with other open platform boards and TOPST board series.  


## GPIO 40 pin Header
Figure 1.1 shows the 2.54 mm Pitch 40 pin IDC Cable.  
<p align="center"><img src="https://github.com/Topst-Dev/Documentation/assets/161264431/78110a46-b1a2-4d70-8450-e2141f44036c"></p>
<p align="center"><strong>Figure 1.1 2.54-mm Pitch 40-pin IDC Cable</strong></p>

Figure 1.2 shows the pin number 1 of the GPIO Header and how to connect an IDC cable to the GPIO Header.  
<p align="center"><img src="https://github.com/Topst-Dev/Documentation/assets/161264431/e8ca78e0-0c1b-4071-87eb-21374536c584"></p>
<p align="center"><strong>Figure 1.2 Connect 40Pin GPIO Header</strong></p>


## GPIO Pin Assignments  

Figure 1.3 shows the GPIO connector pin out.  
All GPIOs are available as inputs and outputs, and some GPIO pins can be switched to function as I2C, I2S, SPI and PWM.  
<p align="center"><img src="https://github.com/Topst-Dev/Documentation/assets/161264431/159778aa-beae-49e6-9a3c-143d61858b82"></p>  
<p align="center"><strong>Figure 1.3 GPIO Connector Pinout</strong></p>

Table 1.1 details the default pin fuctions and available alternative GPIO functions.  

**Table 1.1 Alternative GPIO Functions:**
| GPIO     | Header Pin No. | ALT0    | ALT1          | ALT2         | ALT3       | ALT4        | ALT5        | ALT6        |
|:--------:|:--------------:|:-------:|:-------------:|:------------:|:----------:|:-----------:|:-----------:|:-----------:|
| GPIO_C00 | 23             | GPIOC00 | TS0_CLK       | GSCK(15)     | UT_TXD(15) | -           | -           | -           |
| GPIO_C01 | 24             | GPIOC01 | TS0_VAILD     | GSCMD(15)    | UT_RXD(15) | -           | -           | -           | 
| GPIO_C02 | 19             | GPIOC02 | TS0_SYNC      | GSDO(15)     | UT_RTS(15) | PDM_OUT(48) | -           | -           | 
| GPIO_C03 | 21             | GPIOC03 | TS0_DATA      | GSDI(15)     | UT_CTS(15) | PDM_OUT(49) | -           | -           |  
| GPIO_C04 | 18             | GPIOC04 | TS1_DCLK      | GSCK(16)     | UT_TXD(16) | -           | -           | -           |  
| GPIO_C05 | 22             | GPIOC05 | TS1_VALID     | GSCMD(16)    | UT_RXD(16) | -           | -           | -           |  
| GPIO_C06 | 26             | GPIOC06 | TS1_SYNC      | GSDO(16)     | UT_RTS(16) | PDM_OUT(50) | CLK_OUT(00) | -           |  
| GPIO_C07 | 28             | GPIOC07 | TS1_DATA      | GSDI(16)     | UT_CTS(16) | PDM_OUT(51) | CLK_OUT(01) | RESERVED    |  
| GPIO_C20 | 5              | GPIOC20 | MI2S_BCLK(01) | GSCK(19)     | UT_TXD(21) | I2C_SCL(15) | PDM_OUT(60) | -           |  
| GPIO_C21 | 3              | GPIOC21 | MI2S_LRCK(01) | GSCMD(19)    | UT_RXD(21) | I2C_SDA(15) | PDM_OUT(61) | -           |  
| GPIO_C22 | 7              | GPIOC22 | MI2S_DAI0(01) | GSDO(19)     |      -     | -           | -           | -           |  
| GPIO_C23 | 11             | GPIOC23 | MI2S_DAI1(01) | GSDI1(19)    | UT_CTS(21) | -           | -           | -           |  
| GPIO_C24 | 13             | GPIOC24 | MI2S_DAI2(01) | TCO(4)       | I2C_SCL(16)| PDM_OUT(62) | CLK_OUT(04) | -           |  
| GPIO_C25 | 15             | GPIOC25 | MI2S_DAI3(01) | I2S_MCLK(00) | TCO(5)     | I2C_SDA(16) | PDM_OUT(63) | -           |  
| GPIO_C26 | 8              | GPIOC26 | MI2S_DAO0(01) | I2S_BCLK(00) | GSCK(20)   | UT_TXD(22)  | -           | -           |  
| GPIO_C27 | 10             | GPIOC27 | MI2S_DAO1(01) | I2S_LRCK(00) | GSCMD(20)  | UT_RXD(22)  | -           | -           |  
| GPIO_C28 | 12             | GPIOC28 | MI2S_DAO2(01) | I2S_DAO(00)  | GSDO(20)   | UT_RTS(22)  | I2C_SCL(17) | PDM_OUT(64) |  
| GPIO_C29 | 16             | GPIOC29 | MI2S_DAO3(01) | I2S_DAI(00)  | GSDI(20)   | UT_RTS(22)  | I2C_SDA(17) | PDM_OUT(65) |  
| GPIO_G00 | 27             | GPIOG00 | MI2S_MCLK(02) | I2S_MCLK(01) | GSCK(21)   | UT_TXD(23)  | CLK_OUT(00) | RESERVED    |  
| GPIO_G01 | 29             | GPIOG01 | MI2S_BCLK(02) | I2S_BCLK(01) | GSCMD(21)  | UT_RXD(23)  | PDM_OUT(66) | -           |  
| GPIO_G02 | 31             | GPIOG02 | MI2S_LRCK(02) | I2S_LRCK(01) | GSDO(21)   | UT_RTS(23)  | PDM_OUT(67) | -           |  
| GPIO_G03 | 33             | GPIOG03 | MI2S_DAO0(02) | I2S_DAO0(01) | GSDI(21)   | UT_CTS(23)  | PDM_OUT(68) | -           |  
| GPIO_G04 | 32             | GPIOG04 | MI2S_DAI0(02) | I2S_DAI0(01) | PCIE0_CLK  | I2C_SCL(18) | PDM_OUT(69) | -           |  
| GPIO_G06 | 37             | GPIOG06 | MI2S_DAI1(02) | I2S_MCLK(02) | PCIE0_CLK  | CLK_OUT(01) | -           | -           |  
| GPIO_G07 | 40             | GPIOG07 | MI2S_DAO2(02) | I2S_BCLK(02) | GSCK(22)   | UT_TXD(24)  | CLK_OUT(02) | -           |  
| GPIO_G08 | 36             | GPIOG08 | MI2S_DAI2(02) | I2S_LRCK(02) | GSCMD(22)  | UT_RXD(24)  | CLK_OUT(03) | -           |  
| GPIO_G09 | 38             | GPIOG09 | MI2S_DAO3(02) | I2S_DAO(02)  | GSDO(22)   | UT_RTS(24)  | PDM_OUT(71) | CLK_OUT(04) |  
| GPIO_G10 | 35             | GPIOG10 | MI2S_DAI3(02) | I2S_DAI(02)  | GSDI(22)   | UT_CTS(24)  | PDM_OUT(72) | CLK_OUT(05) |  

- GSCK, GSCMD, GSDO, GSDO, GSDI: SPI CLK, CS, MOSI, MISO
- PDM_OUT: PWM



