# UponorRS485
This is a small collection about the RS485 protocol used by uponor between Smatrix Base and eg. the Roomthermostat.

I have collectet this information by sniffing the packets between Smatrix Base and Roomthermstat T-145. I'm sure there are some more Informations in the protocoll, but till now i don't know them.

## Operating Modes

There are 2 operating modes.
- Normal Mode
- TeachIn Mode.

The TeachIn Mode is used while adding a new Thermostat or changing the outputs of a know Thermostat.
See the origingal dokumentation of the Smatrix Base to see how to get to TeachIn Mode.

## Hardware level

The Bus is RS485 at 19200 Baud 8N1.

## Protocoll

The Protocoll is a simple Master Slave Protocoll and folws some easy rules.
The Smatrix Base is Master. The connected and Teached in Roomthermosats are Slave.

**In Normal Mode**
1. The Master sends round robin a Poll Message to all the Slaves.
2. The Slave which gets polled answers a Message within all his values.
3. The Master sends ack Message within the given values from the Slave and some additinal informations.

**In TeachIn Mode**
during TeachIn, the Protocoll is a bit more complicated, but still easy.
1. The Master sends search Telegrmm as long as it is in TeachIn Mode.
2. The Slave which should Teached in answers with first Handshake Message.
3. The Master answers with a ack Message.
4. The Slave which should Teached in ansers with second Handshake Message.

Now TeachIn is done. The Master goes back to 1. and stays there untill the next Slave is in TeachIn Mode.

**Basic Protocoll Informations**
In Both modes the Telegram follows the same Rules.
I'm not sure about all the following Inforamtions. Please see also the Section with the detailed Protocoll Informations.

The Telegrams can have different length.
The Telegram did not have a delimeter.

Telegram structure:

4 Byte Device Type
2 Byte Slave ID
x Byte Payload
2 Byte modbus crc16 checksum, high and low byte switched.

