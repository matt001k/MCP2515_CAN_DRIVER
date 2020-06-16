# MCP2515_CAN_DRIVER
Generic CAN Bus Controller Driver Made to Interface with Any MCU

The idea behind this was to make the driver easily compatible with any MCU. Tests were done with the ESP8266 and STM32F031K6. The following  are an explanation and test results of the driver.

At the beginning of the header file, there are configurable variables used in order to determine items such as the Oscillator's frequency, CAN Bus baud rate, the number of time quanta, whether RX Buffer interrups will be enabled and filter etc. These should be changed in accordance to the user's preference and needs. 



## Functions

```C
mcp2515_status_t mcp2515_init(void);
```
- Used to initialize the prescale (in accordance to the user's settings) along with the control registers for transmission, and reception of messages as well as any masks or filtering for the received CAN messages


```C
mcp2515_status_t mcp2515_operation_mode(mcp2515_operation_mode_t op_mode);
```
- Used in order to change the operation mode of the controller
- mcp2515_operation_mode_t values are as follows:
  - NORMAL_MODE
  - SLEEP_MODE
  - LOOPBACK_MODE
  - LISTEN_ONLY_MODE
  - CONFIGURATION_MODE
  
 ```C
mcp2515_status_t mcp2515_fast_tx_message(uint32_t identifier, uint32_t *data, uint8_t rtr_bit, uint32_t data_len);
```
- Quickly transmits a message through transmission buffer zero
- Can transmit the message identifier, whether the frame is an Remote Frame as well as up to 8 bytes of data(no data in the case of RTR frame)

 ```C
mcp2515_status_t mcp2515_tx_fill_buffer_0(uint32_t identifier, uint32_t *data, uint8_t rtr_bit, uint32_t data_len);       
mcp2515_status_t mcp2515_tx_fill_buffer_1(uint32_t identifier, uint32_t *data, uint8_t rtr_bit, uint32_t data_len);        
mcp2515_status_t mcp2515_tx_fill_buffer_2(uint32_t identifier, uint32_t *data, uint8_t rtr_bit, uint32_t data_len);
```
- Fills the buffer signified in the function

 ```C
mcp2515_status_t mcp2515_tx_send_buffer(uint32_t tx_buf_num);
```
- Sends the buffer dignified over the CAN Bus
- Buffers are of value:
  - 0
  - 1
  - 2
  
 ```C
mcp2515_status_t mcp2515_rx_buffer_read(uint32_t rx_buf_num, uint32_t *identifier, uint32_t *data);
```
- Reads the respective buffer dignified within rx_bug_num
- Buffers are of value:
  - 0
  - 1
- The identifier and data will be stored within the respective pointer locations, function will dignify between a Remote Frame and a Data Frame as well as whether the buffer is standard or extended frame



## Testing
The MCP2515 was connected with an MCP2562 CAN Transceiver in order to turn the TX lines from the controller into CANH and CANL lines on the CAN Bus.
The following are results from testing in Loopback mode, which sends and receives data to the same controller(only need one controller in order to test), all testing was done using a logic analyzer with Pulse View software, a serial terminal along with an Oscilloscope:

### Design Circuit
<p align="center">
  <img width="460" height="300" src="https://github.com/matt001k/MCP2515_CAN_DRIVER/blob/master/MCP2515/photos/Design%20Circuit.PNG">
</p>

### SPI Testing
Testing was done on the written ESP8266 SPI driver to ensure that communication was working the following is random data being written to the MCP2515:
![Alt text](https://github.com/matt001k/MCP2515_CAN_DRIVER/blob/master/MCP2515/photos/SPI%20Readings.PNG)

Next tests were done to ensure data could be received from the Controller via MISO. The data read was from the prescale configuration registers. These tests were stored into pointers and displayed on a serial monitor:
![Alt text](https://github.com/matt001k/MCP2515_CAN_DRIVER/blob/master/MCP2515/photos/CNF_1%2C2%2C3_TEST.PNG)

### SPI Communication to CAN Controller for Data Transmission
The following are waveforms indicating the standard and extended identifier frames being sent to the TX buffer zero and then sent over the CAN bus:

***STANDARD IDENTIFIER DATA FRAME***
![Alt text](https://github.com/matt001k/MCP2515_CAN_DRIVER/blob/master/MCP2515/photos/STA_IDENTIFIER_TEST_WAVEFORMS.PNG "STANDARD DATA FRAME") 

***STANDARD IDENTIFIER REMOTE FRAME***
![Alt text](https://github.com/matt001k/MCP2515_CAN_DRIVER/blob/master/MCP2515/photos/STA_IDENTIFIER_TEST_RTR_WAVEFORMS.PNG "STANDARD REMOTE FRAME") 

***EXTENDED IDENTIFIER DATA FRAME***
![Alt text](https://github.com/matt001k/MCP2515_CAN_DRIVER/blob/master/MCP2515/photos/EXT_IDENTIFIER_TEST_WAVEFORMS.PNG "EXTENDED DATA FRAME") 

### SPI Communication to CAN Controller for Data Reception
The following are waveforms indicating the standard and extended identifier frames being received over RX buffer zero, and analyzed with Pulse View:

***STANDARD IDENTIFIER DATA FRAME***
![Alt text](https://github.com/matt001k/MCP2515_CAN_DRIVER/blob/master/MCP2515/photos/CONTROLLER_RECV_BUFF_WAVEFORM.PNG "STANDARD DATA FRAME") 

***STANDARD IDENTIFIER REMOTE FRAME***
![Alt text](https://github.com/matt001k/MCP2515_CAN_DRIVER/blob/master/MCP2515/photos/STA_IDENTIFIER_TEST_RTR_WAVEFORMS.PNG "STANDARD REMOTE FRAME") 

***EXTENDED IDENTIFIER DATA FRAME***
![Alt text](https://github.com/matt001k/MCP2515_CAN_DRIVER/blob/master/MCP2515/photos/CONTROLLER_RECV_EXT_ID_TEST_WAVEFORM.PNG "EXTENDED DATA FRAME")

###CAN Bus CANH and CANL Readings
Lastly, in Normal Mode on the controller, the data being sent out was analyzed on an oscilloscope to ensure the CANH and CANL lines were working with the transmitted messages from the controller. The following is a reading of a data fram on a Oscilloscope:
![Alt text](https://github.com/matt001k/MCP2515_CAN_DRIVER/blob/master/MCP2515/photos/CAN0.png)

A data bit of "1" was analyzed in order to determine if the prescaled baud rate was initialized properly. The results are as follows:
![Alt text](https://github.com/matt001k/MCP2515_CAN_DRIVER/blob/master/MCP2515/photos/CAN1.png)

With a time difference of 8 microseconds, the frequency of the CAN lines will be at 128000 bit/s, which is precisely what was configured in the initialization of the CAN controller. 


Items still needed to be tested:
- Filters and Masks
- Interface between multiple controllers
