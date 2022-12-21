This first thing that I've founde about the connection of __9XTend__ is that we need to use the module that Mr.Rezvani developed to be able to communication the data with the Computer. From Computer I mean PC or raspberry pi board.

I have tried to connect the RF module to raspberry pi directly with the pi pinout but I could not receive the data comming from the mate module which is connected into pixhawk board. On the other hand when we use the Mr.Rezvani usb module the data nicely transfers even with raspberry pi board and we can infer that the power supply is not a problem.

So as I understood the problem is about the protocol of connection. consequently, I tested this connection with __PL2303__ module which we had bought in our shoppings. but it does not work. I will test this module more in next tries.

Mr.Rezvani said that you must also use the pin 7 of the 9XTend module to enable the module to work. This pin is __shutdown__ pin and during operation must be high. Also, If you have looked at the pinout in datasheet, you would be informed that this pin is a __"must use pin"__ for the module. So read the datasheet with more concentration!

Some Electrical concepts that are good to know:
- [[TTL and RS232]]: somethings releated to voltage level of logical electronic parts and modules.
- Working Modes of 9XTend Module:
	- Transparent Operation: [Some good information in this link](https://www.digi.com/resources/documentation/Digidocs/90001942-13/concepts/c_transparent_mode_detailed.htm?tocpath=XBee%20transparent%20mode%7CXBee%20transparent%20mode%20in%20detail%7C_____0) 
	-  API Operation: [Some good information in this link](https://www.digi.com/resources/documentation/Digidocs/90001942-13/Default.htm#concepts/c_api_mode_detailed.htm?TocPath=API%2520mode%257C_____1)

Air Module:
	Firmware version: 206F
	SH: 3852
	SL: 19CB
Ground Module:
	Firmware version: 206F
	SH: 5761
	SL: 12CB

Now let's see how XBee python library can work for us!

