<h1 style="color:red">
  Key Connector (J10D1)
</h1>


J10D1 is a standard 20-pin/2 mm connector to connect an external key function.  
If you want to use other switch functions, a key sub-board should be developed first.  

Figure 1.1 shows the location of key connector (J10D1).
<p align="center"><img src="https://github.com/Topst-Dev/Documentation/assets/161264431/217d800b-11e6-4dd2-a244-1e3c2cce6602"></p> 
<p align="center"><strong>Figure 1.1 Key Connector</strong></p>

Figure 1.2 shows the schematic of key connector (J10D1).
<p align="center"><img src="https://github.com/Topst-Dev/Documentation/assets/161264431/ab405c1f-f588-4a52-9a00-a6bfab0515dc"></p>  
<p align="center"><strong>Figure 1.2 Schematic of Key Connector</strong></p>


Table 1.1 shows the pin description of key header connector (J10D1).  

**Table 1.1 J10D1 Pin Description:**    

|  Pin Number | TCC7045 Port Name | Signal Name | Dir(MCU◀▶J20D1)| Description            |
|:-----------:|:-----------------:|:-----------:|:----------------:|------------------------|
| 1           |         -         | B_UP        |       -          | B_UP Power             |
| 2           |         -         | B_UP        |       -          | B_UP Power             |
| 3           |         -         | DGND        |       -          | Ground                 |
| 4           |         -         | DGND        |       -          | Ground                 |
| 5           |         -         | SYS_3P3     |       -          | SYS_3P3V               |
| 6           |         -         | SYS_3P3     |       -          | SYS_3P3V               |
| 7           | GPIO_B07          | GPIO_B07    |  ◀              | Jog signal for VOL_DN  |
| 8           | GPIO_AC01         | GPIO_AC01   |  ◀              | Jog signal for KEY_DN  |
| 9           | GPIO_A28          | GPIO_A28    |  ◀              | Jog signal for VOL_UP  |
| 10          | GPIO_AC00         | GPIO_AC00   |  ◀              | Jog signal for KEY_UP  |
| 11          | AD008             | AD008       |  ◀              | ADC Key                |
| 12          | AD009             | AD009       |  ◀              | ADC Key                |
| 13          | AD010             | AD010       |  ◀              | ADC Key                |
| 14          | AD001             | AD001       |  ◀              | ADC Key                |
| 15          | STR_EN            | STR_EN      |  ◀              | STR_EN signal          |
| 16          | KEY0              | KEY0        |  ◀              | Interrupt Gpio Key     |
| 17          | ACCB              | ACCB        |  ◀              | ACC Signal             |
| 18          | ILL+              | ILL+        |  ◀              | Illumination + Signal  |
| 19          |         -         | DGND        |       -          | Ground                 |
| 20          |         -         | DGND        |       -          | Ground                 |


Table 1.2 shows the pin description of key header connector (J10D1). 

**Table 1.2 J10D1 Pin Fuction:**  

|  No         | TCC7045 Port Name | Signal Name | IO Level | FUNC0        | FUNC1       | FUNC2       | FUNC3          | FUNC4(ANALOG) |
|:-----------:|:-----------------:|:-----------:|:--------:|:------------:|:-----------:|:-----------:|:--------------:|:-------------:|
| 1           |         -         | B_UP        | 3.3V     |      -       |      -      |      -      |        -       |       -       |
| 2           |         -         | B_UP        | 3.3V     |      -       |      -      |      -      |        -       |       -       |
| 3           |         -         | DGND        | 3.3V     |      -       |      -      |      -      |        -       |       -       |
| 4           |         -         | DGND        | 3.3V     |      -       |      -      |      -      |        -       |       -       |
| 5           |         -         | SYS_3P3     | 3.3V     |      -       |      -      |      -      |        -       |       -       |
| 6           |         -         | SYS_3P3     | 3.3V     |      -       |      -      |      -      |        -       |       -       |
| 7           | GPIO_B07          | GPIO_B07    | 3.3V     | GPIO_B[07]   | GSDI0_CH0   | PWM_OUT[01] | MFIO_1_CH1[03] |       -       | 
| 8           | GPIO_AC01         | GPIO_AC01   | 3.3V     | GPIO_AC[01]  |      -      | UT2_RXD_CH1 |        -       | AD1[01]       |
| 9           | GPIO_A28          | GPIO_A28    | 3.3V     |  GPIO_B[28]  | UT2_CTS_CH0 | PWM_OUT[08] |        -       |       -       | 
| 10          | GPIO_AC00         | GPIO_AC00   | 3.3V     | GPIO_AC[00]  |      -      | UT2_TXD_CH1 |        -       | AD1[00]       | 
| 11          | AD008             | AD008       | 3.3V     |      -       |      -      |      -      |        -       | AD0[08]       |
| 12          | AD009             | AD009       | 3.3V     |      -       |      -      |      -      |        -       | AD0[09]       |
| 13          | AD010             | AD010       | 3.3V     |      -       |      -      |      -      |        -       | AD0[10]       |
| 14          | AD001             | AD001       | 3.3V     |      -       |      -      |      -      |        -       | AD0[11]       |
| 15          | STR_EN            | STR_EN      | 3.3V     |      -       |      -      |      -      |        -       |       -       |
| 16          | KEY0              | KEY0        | 3.3V     |      -       |      -      |      -      |        -       |       -       |
| 17          | ACCB              | ACCB        | 3.3V     |      -       |      -      |      -      |        -       |       -       |
| 18          | ILL+              | ILL+        | 3.3V     |      -       |      -      |      -      |        -       |       -       |
| 19          |         -         | DGND        | 3.3V     |      -       |      -      |      -      |        -       |       -       |
| 20          |         -         | DGND        | 3.3V     |      -       |      -      |      -      |        -       |       -       |
