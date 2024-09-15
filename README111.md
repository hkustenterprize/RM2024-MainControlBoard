# RM2024 G473-Main-Control-Board Manual


G473 Main Control Board (the "G" Board/ G板) consists of one **Core Board** and one **Connector Board** / **Extension Board** (for ENGNEER). Two PCBs are connected via a 80-Pin BTB Connector.

HW related bugs/problem:
zxianaa@connect.ust.hk

Use [G Bank](https://docs.google.com/spreadsheets/d/1zWNblgsTIIUWQSXFcsPFqQVMJEhKg5JTl5KiXP_lMfQ/edit?usp=drive_link) for inventory management.
> building, take F Bank for reference

Comment in [G Feedback](https://docs.google.com/spreadsheets/d/1CWj03PfoRyROP8zgPTzhCLMOIUZqd73VYrf7h7KjqA0/edit?usp=drive_link) for any problem feedback, idea and recommendation.
> building

See [Release](https://github.com/hkustenterprize/RM2024-G-Board-Hardware/releases) for kicad files and detailed changelog of each version. The version can be found on the top of PCBs. 
> Though it's not likely to cause any problem, it's not recommend to use different versions of PCBs together unless there's "HW is same as Vxx,xx" remark.

| Version | Status | Description |
| ------- | ------- | ------- |
| V24.02 | Active (Sentry, Balanced infantry) | First useable CShape Version |
| V24.03 | Active (Sentry, Balanced infantry) | HW is same as V24.02, except SMT |
| V24.04 | Testing | change design of buzzer, add RGB for debugging, add Flash for logging data, etc |
> This table only lists the CShape Version. Older versions (V1, V2, FShape, etc.) are not listed here.


## Overview

MCU: STM32G473VET6

IMU: ICM-42688P (SPI1)

Magnetometer: LIS3MDLTR (SPI1)

3x FDCAN for motor control 
> interboard function (pending)

1x RS485 for interboard

2x UART-TTL for communicating with MiniPC

2x External UART for referee system and transmission

1x DR16 connector connected to two UART

## UART Allocation
Since STM32G473's limited UART peripherals (only 5), the G Board uses internal MUX to multiplex UARTs.
> e.g. UART1 can be accessed from PC4/PC5 and PE0/PE1

Pinout:
<div style="margin-left: 20px">

|        | TX   | RX   | DE   | CTS  | RTS  | Function          |
| -----  | ---- | ---- | ---- | ---- | ---- | ----              |
| UART1A | PC4  | PC5  | -    | -    | -    | Referee / Video   |
| UART1B | PE0  | PE1  | -    | -    | -    | General           |
| UART1C | -    | PA10 | -    | -    | -    | DR16 receiver     |
| UART2A | PA2  | PA3  | -    | PA0  | PA1  | UART-TTL, MiniPC1 |
| UART2B | PD5  | PD6  | -    | -    | -    | General           |  
| UART3A | PD8  | PD9  | -    | PD11 | PD12 | UART-TTL, MiniPC2 |
| UART3B | PB10 | PE15 | -    | -    | -    | General           |  
| UART3C | -    | PB11 | -    | -    | -    | DR16 receiver     | 
| UART4  | PC10 | PC11 | PA15 | -    | -    | RS485             |   
| UART5  | PC12 | PD2  | -    | -    | -    | Referee / Video   |
</div>

Allocation:
>not necessarily need to strictly follow

<div style="margin-left: 20px">

|                   | UART1 | UART2 | UART3 | UART4 | UART5 |
| -----             | ----  | ----  | ----  | ----  | ----  |
| Standard Infantry | [C] DR16 | [A] MiniPC | RESV | RS485 | Video |
| Balanced Infantry | [C] DR16 | [A] MiniPC | RESV | RS485 | Video |
| Hero              | [C] DR16 | [A] MiniPC | RESV | RS485 | Video |
| Sentry (RM24)     | [C] DR16 | [A] MiniPC | [A] MiniPC | RS485 | Video |
| *Sentry (RM25)     | [C] DR16 | [A] MiniPC | [A] MiniPC | *Referee | Video |
| Engineer          |  //
| General Chassis   | RESV | RESV | RESV | RS485 | Referee |

> "RESV" = reserved, can be used for bus servo, etc.

> *Sentry (RM25) draft plan. After V24.04 UART4 can be accessed from connector board. 
</div>

## Peripherals //
> All marks is in MCU side (i.e. same as ioc)
### CAN (x3)
<div style="margin-left: 20px">

|       | TX   | RX   |
| ----- | ---- | ---- |
| CAN1  | PD1  | PD0  |
| CAN2  | PB6  | PB5  |
| CAN3  | PB4  | PB3  |
</div>

- 3* SIT1042T/3
- 1* 2Pin-GH1.25 connector in core board for testing only and 2* 2Pin-GH1.25 connector in connector board for normal use. 
- Each TX & RX signal wire has 82R resistor in series. Each CAN_H &CAN_L signal wire has ESD protection and 120R resistor between differential wires.

### RS485//
<div style="margin-left: 20px">

|       | TX/D | RX/R | DE   |
| ----- | ---- | ---- | ---- |
| RS485 | PC10 | PC11 | PA15 |
</div>

- SN75176AD//
- 1* 2Pin-GH1.25 connector in core board for testing only and 2* 2Pin-GH1.25 connector in connector board for normal use. 
- Hardware flow control available.
- TX & RX & DE signal wire has 82R resistor in series. RS485_A & RS485_B signal wire has ESD protection and 120R resistor between differential wires

### UART-TTL (x2)
<div style="margin-left: 20px">

|       | TX   | RX   | CTS  | RTS  |
| ----- | ---- | ---- | ---- | ---- |
| TTL1 (UART2A)  | PA2  | PA3  | PA0  | PA1  |
| TTL2 (UART3A)  | PD8  | PD9  | PD11 | PD12 |
</div>

- CH343P
- 1* 4Pin-PH2.0 connector on connector board.
- Full flow control available.
- LED (Purple) for indicating status.
- Powered by 3V3_IO network. Enabled by 5V network.
- TX & RX & CTS & RTS signal wire has 82R resistor in series. USB_DP & USB_DN signal wire has ESD protection.

### QSPI FLASH
<div style="margin-left: 20px">

| FLASH | NCS   | CLK   | IO0  | IO1  | IO2  | IO3  |
| ----- | ---- | ---- | ---- | ---- | ---- | ---- |
| MCU   | PD3  | PF10  | PD4  | PC2  | PD3  | PC7  |

</div>

### DR16//
<div style="margin-left: 20px">

| GPIO |  UART  |
| ---- | ----   | 
| PB11 | UART3C  |
| PA10 | UART1C  |
</div>

- Readable from both UARTs
- No hardware inverter, remember to enable software input inverter:
    ~~~
    //
    ~~~

### Power ADC//
<div style="margin-left: 20px">

|       | ADC GPIO   | Description |
| ----- | ---- | ---- | 
| ADC_PWR | PD3  | Senses the voltage of XT30 input with a 10k:1k voltage divider |
| ADC_5V  | PD10 | Senses the voltage of 5V network with a 1k:1k voltage divider |
</div>

#### Power status calculation:


### IMU Heater (testing)
- Set PE8 (V24.03) / PE7 (V24.04&After) to high to enable
- 10k pull-down resistor
- Output power
    $$ P = \frac{{(5.2V)}^2}{36\Omega} $$ 
### Debug
- Debug button (PD15)
    - 10k pull-up resistor, active low
- LED (PE3)
    - Set PE3 to low to enable
- RGB [V24.04&After]
    - //
### ID Config
<div style="margin-left: 20px">

| ID0(PC13) | ID1(PC14) | ID2(PC15) | Description |
| -----     | ----      | ----      | ----        |
| 0 | 0 | 0 | Standard Connector Board            |
| 0 | 0 | 1 | Engineer Extension Board            |
</div> 

### Master/Slave Config [V24.04&After] //
- Check for Master/Slave role.
- Prevent RS485 transceiver damage in case two board are both running Master code.
  - Send warning message and close RS485 transmission if the role is not right.

## Power tree // 
| Net Name	| Source	| Connected to	| Description |
| -------   | -------   | -------       | -------      |
|24V_VIN	|XT30 with Fuse&TVS|	RY8411 input	|Main power input   |
|VINP	    |XT30	    |Voltage divider to MCU ADC	|Just for voltage sense
|5V	        |1. RY8411 output with diode & TVS <br> 2. VBUS with diode: only when 24V is not connected <br> 3. 3V3_SWD with diode & TVS: only when 24V and VBUS is not connected|RY3410 input, <br>Buzzer, Heater, <br>2* RS3236’s input, <br>2* CH343P EN, <br>3* CAN transceiver, <br>1* RS485 transceiver, <br>Connector Board	|    Current limit 1.2A <br> 5.2V   |   
|VBUS       |MiniPC USB |   5V network after diode & TVS|"Backup" power <br> Normally shounldn't be used|
|3V3_MCU    |RY3410 output	|MCU VDD and VREF <br>IMU and MAG VDDIO |Power for digital device   |
|3V3_IMU    |RS3236 output  |IMU and MAG VDD    |Power for sensors|	
|3V3_IO     |RS3236 output  |2* CH343P, <br>3* CAN transceiver, <br>IO Pullup/Pulldown, <br>LEDs	|Power for IO related things|
|3V3_SWD    |SWD Connector  |	5V network after diode & TVS    |Auxiliary power when burning code	|
## Software testing //
## ioc Configuration //
## Soldering Precautions //
1.	Solder the DC-DC parts (RY8411 & RY3410) and input TVS & Fuse (Green Highlight Components), but not XT30.
2.	Test 5V and 3V3_MCU output
> 5V network should be around 5.2V and 3V3_MCU network should be around 3.42V
3.	Solder UART-TTL (CH343P*2), CAN Transceiver (SIT1402*3), RS485 Transceiver (SN65HVD1176) and LDOs (RS3236*2)
4.	Test 3V3_IO and 3V3_IMU
5.	Solder the rest of the board except the BTB connector and IMU (ICM-42688) and Magnetometer (LIS3MDLTR)
6.	Wash the board. Check if the MCU is alive, then test CAN*3 and RS485 (ask software).
7.	Solder the BTB connector and IMU (ICM-42688) and Magnetometer (LIS3MDLTR)
*Do not use ultrasonic washer after soldering IMU and Magnetometer
8.	Solder the Connector Board
*Cut short the pins of the PH2.0 4pin connectors to prevent intervene. 
9.	Full function test (ask software).
10.	3D print a shell (ask mech).
