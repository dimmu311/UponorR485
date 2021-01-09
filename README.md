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

Now TeachIn is done. The Master goes back to 1. and stays there untill the next Slave is in TeachIn Mode or the Master cancels TeachIn.

**Basic Protocoll Informations**
In Both modes the Telegram follows the same Rules.
I'm not sure about all the following Inforamtions. Please see also the Section with the detailed Protocoll Informations.

The Telegrams can have different length.
The Telegram did not have a delimeter.

Telegram structure:

4 Byte Header  
* 2 Byte Device Type  
* 1 Byte Slave ID  
* 1 Byte Device Type  


x Byte Payload
* 1 Byte at Poll Message or  
* Multiple of 3 Bytes at answer Message or ACK Message  


2 Byte modbus crc16 checksum, high and low byte switched.  

The Payload:  
The Payload is either a Poll Message from the Master. The Poll Message is always 1 Byte
The answer and ACK Message are always a Multiple of 3 Bytes long. This 3 Bytes are 1 Byte Function Type and 2 Byte Value. See at the Bottom what i found out.
There are 3 possible ways the value can be interpreted.
* as Temperatur (you need this formula to convert the Value to °C: (Value-320)/18)
* as Integer
* as Bitmask
The Function Type Byte defines how to interpret the Value.

## one small example
see first this example sniffing from the communication between the Smatrix Base and the T-145 Roomthermostat.
1. 10	04 21 77 FF D3 49
2. 10	04 21	77 40	03 04	3E 00	00 3F	0C 08	3B 02	B3 53	D0
3. 10	04 21	77 2D	7F FF	3D 00	00 0C	00 24	37 01	9A 38	03 B6	3B 02	B3 3C	00 48	35 80	00 4A	31

split this down:
1. This is the Poll Message from the Smatrix Base.
2. Is the answer Message from the Roomthermostat.
3. is the ACK Message from the Smatrix Base.

## digging deeper in the messages
at first lets remove the uninteresting parts of the example Messages.  
Remove the 4 Byte Header at the Beginning and the 2 Byte CRC at the End.
1. ~~10	04 21 77~~ FF ~~D3 49~~
2. ~~10	04 21	77~~ 40	03 04	3E 00	00 3F	0C 08	3B 02	B3 ~~53	D0~~
3. ~~10	04 21	77~~ 2D	7F FF	3D 00	00 0C	00 24	37 01	9A 38	03 B6	3B 02	B3 3C	00 48	35 80	00 ~~4A	31~~

next try to understand the Message:  
1. FF -> Poll  
2. Split down the paylod to 3 Byte long segments:  
Remember, 1 Byte Function Type, 2 Bytes Value  
* 40 03 04	-> Function 40: Actual Temperatur = 0x0304 = 772 ---> (772-320)/18 = 25,1°C  
* 3E 00 00 -> Function 3E:   
* 3F 0C 08 -> Function 3F: Operating Mode = Bitmask = 0000 1100 0000 1000 ---> Bit 3 Eco Enabled, Bit 10 ? Bit 11 ?  
* 3B 02 B3 -> Function 3B: Set Point Temperatur = 0x02B3 = 691 ---> (691-320)/18 = 20,6°C  
3.


