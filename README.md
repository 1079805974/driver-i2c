# I2C (Inter-Integrated Circuit)
This driver can be used to communicate via I2C between microcontrollers

## Summary

- Includes

```
#include <compat/twi.h>                   // must always be included for the definitions of TWI states
```

- Events

```
output (on/off,u8?) I2C; // on/off of I2C with optional variable address

// master:
output u8 I2C_MASTER_SEND;                // master requests to send size of tx_buf number of bytes to slave with the address mentioned in the parameter
output (u8,u8) I2C_MASTER_RECEIVE;        // master requests certain amount of data from slave with address equal to the second parameter.

//slave:
input none I2C_SLAVE_SEND_ACKED;          // emitted when the master addresses this device with a send request.
output none I2C_SLAVE_SEND;               // emit this output to send data that's in the tx_buf.
input (u8,u8) I2C_SLAVE_RECEIVE;          // await this input to receive data from the master, returns number of bytes and errorcode.

//input to await the completion of transfer
input  (u8,u8) I2C_DONE;                  // this input is emitted once the transfer of data is complete. The values returned are number of bytes and errorcode respectively.


```

- Buffers

```
#define BUF_MAX 20                        // maximum buffer length
var[BUF_MAX*] byte tx_buf;                // transmit buffer
var[BUF_MAX] byte rx_buf;                 // receive buffer

```

## Includes

To use I2C communication, a program must always include `i2c.ceu`:

```
#include "i2c.ceu"
```

## Events

Céu-Arduino provides declarations to communicate between devices via I2C. The API for the ceu driver for I2C is as follows:

- Include the driver after cloning this repository in libraries with #include "i2c.ceu".
- emit the I2C to on to begin the Two Wire Interface. It has a second optional parameter for address which is necessary for a slave and optional for a master.

```
emit I2C(on,_);
```
- Set address of the device by emitting I2C with the desired address as the second parameter.

```
emit I2C(on,8);
```

- To stop using I2C communication you can turn it off by emitting the I2C to off

```
emit I2C(off,_);
```

### MASTER SENDER

- Master sending data can be acheived by setting the tx_buf and then emitting I2C_MASTER_SEND to the desired address.
- The tx_buf contains the data we will send and I2C_MASTER_SEND has a parameter of slave address.
- We can await the completion of the transfer with the input I2C_DONE.
- Following is an example of master send.

```
tx_buf = [1,2,3,4,5];                     // transmit buffer contains values to transfer and is of size 5
emit I2C_MASTER_SEND(4);                  // send the data to the slave with address 4
await I2C_DONE;                           // await for the request to be completed
```

- Refer [master_sender01.ceu](https://github.com/ceu-arduino/driver-i2c/blob/pre-v0.40/examples/master/master_sender01.ceu) for example.


### MASTER RECEIVER

- Master receiving data can be acheived by emitting I2C_MASTER_RECEIVE to the desired address and specifying the number of bytes.
- The output I2C_MASTER_RECEIVE has parameters slave address and the number of bytes requested.
- We can await the completion of the transfer with the input I2C_DONE.
- Following is an example of master receive.

```
emit I2C_MASTER_RECEIVE(4,2); // request 2 bytes data from the slave with address 4
await I2C_DONE; // await for the request to be completed
```

- The received data is then stored in the vector rx_buf which can be accessed for the data.
- refer [master_receiver01.ceu](https://github.com/ceu-arduino/driver-i2c/blob/pre-v0.40/examples/master/master_receiver01.ceu) for example where the data received is printed in serial monitor.

### SLAVE SENDER

- Slave requests require awaiting of the request I2C_SLAVE_SEND_ACKED which returns 3 for read and 2 for write. The enums defined for them are TWI_MODE_READ_SLAVE and TWI_MODE_WRITE_SLAVE respectively.. This await is complete when the master addressed the slave with it's address.
- Slave sending data can be acheived by setting the tx_buf and then emitting I2C_SEND.
- The tx_buf contains the data we will send.
- We can await the completion of the transfer with the input I2C_DONE.
- Following is an example of slave sender.

```
await I2C_SLAVE_SEND_ACKED; // await for master to address the slave
tx_buf = [4,5,6,7]; // transmit buffer contains values to transfer and is of size 4
emit I2C_SEND(); // send the data to master
await I2C_DONE; // await for the request to be completed
```

- Refer [slave_sender01.ceu](https://github.com/ceu-arduino/driver-i2c/blob/pre-v0.40/examples/slave/slave_sender01.ceu) for example.


### SLAVE RECEIVER

- Slave requests require awaiting of the request I2C_SLAVE_SEND_ACKED which returns 3 for read and 2 for write. The enums defined for them are TWI_MODE_READ_SLAVE and TWI_MODE_WRITE_SLAVE respectively. This await is complete when the master addressed the slave with it's address.
- Slave receiving data can be acheived by emitting I2C_RECEIVE.
- The output I2C_RECEIVE has no parameters.
- We can await the completion of the transfer with the input I2C_DONE.
- Following is an example of slave receive.

```
await I2C_SLAVE_RECEIVE();
```

- The received data is then stored in the vector rx_buf which can be accessed for the data.
- refer [slave_receiver01.ceu](https://github.com/ceu-arduino/driver-i2c/blob/pre-v0.40/examples/slave/slave_receiver01.ceu) for example where the data received is printed in serial monitor.


## Comparison with the Arduino API

### Master Sender

- The Arduino API
    - The master sends data using the following syntax :

    ```
    Wire.beginTransmission(8); // transmit to device #8
    Wire.write(7);              // sends one byte with value 7
    Wire.endTransmission();    // stop transmitting
    ```

    The ISR 


- The Céu API
    - The master sends data using the following syntax