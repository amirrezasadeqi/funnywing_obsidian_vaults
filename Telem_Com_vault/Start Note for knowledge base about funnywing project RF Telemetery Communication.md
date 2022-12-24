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
	module address: module SH (3852)
 
Another profile for air module:
	Firmware version: 8064
	SH: 13A200
	SL: 404A9EAB
	DH: 13A2FF
	DL: FFF112CB

Ground Module:
	Firmware version: 206F
	SH: 5761
	SL: 12CB
	module address: module SH (5761)

Another profile for Ground Module:
	Firmware version: 8064
	SH: 13A2FF
	SL: FFF112CB
	DH: 13A200
	DL: 404A9EAB

Now let's see how XBee python library can work for us!

I think we can use the rf modules!

### Is there a way to access a shell using ssh over a serial port?
Actually you can do this by forwarding/piping the ssh port to serial port or vice versa. The other ways of doing this are, using programs like __screen__ or __minicom__ (I think the XCTU also does the job), but I did not test them yet!

TOOD: Access to a bash shell using screen or minicom program!

### How to set the default baud-rate of a serial port in linux?
You can do this by using `stty` command like bellow:
```
stty -F /dev/ttyUSB0 ...
```
you should check more information on how to use this command on the internet, but I think this is the right command to do this. I also used it on raspberry pi for changing the `ttyAMA0` port baud rate, but actually I was not successful.

Note that some guy on the [internt](https://forums.raspberrypi.com/viewtopic.php?t=149927) has tried to use raspberry pi(I think rpi 3) GPIO pins for serial connection, but he could not to do this, so he has done what I have done, I mean connecting RF module to rpi usb ports using a usb to TTL serial cable(I have used usb to TTL module).

In future search and work on usign GPIO for RF module connection to RPI board and ...

### Is there any ROS package for Connecting ROS hosts/nodes/topics over serial ports?
Actually the only package I found about serial connection in ROS, is __rosserial__ package but it is more for connection of MCUs to a host running ROS. Note that in rosserial package is a library named __rosserial_embeddedlinux__ that I think can help for doing our job, I mean connecting two hosts running ROS(in my case GCS computer and unboard RPI board).
The other package which may be helpful is __rosserial_xbee__, but I'm not confident!
[This tutorial](https://sonictl.github.io/from_cnblogs/2016/02/01/p20160202062400.html) also may be helpful.
There is also a [question in ROS](https://answers.ros.org/question/328832/how-to-talk-to-two-ros-devices-over-serial/) forums which can be helpful. The guy asking the question, wants to do something to what we want to do, Connecting to RPIs running ROS over wireless serial port(XBee modules). He said that he has managed to do this by creating an ethernet bridge over serial connection(XBee module) and he provided a [tutorial](https://forums.raspberrypi.com/viewtopic.php?t=149927) which can be so helpful.

To sum the subject of proper method of connecting GCS and RPI using XTend RF modules up, I should state that although some above mentioned methods and tutorials may help to do this, but I am not confident about them and in future I will test them and ... 
For now to do the job, I will use XBee python library (if it did not help,  use pySerial instead) for RF module communication and send/recieve data between GCS and RPI board, since I think using this method I could be more to the point, I mean I can get what exactly I want and nothing more or less, and furthermore at the moment I have no more time to test different methods which may not help to do the job, so I will use the more probable suitable method, __XBee python library with sending some predefined string like commands to RPI for doing some tasks like simple goto commands and also receiving sensor datas like funnywing GPS data over RF connection__. In future I will test more methods and ... 

Now, Let's Code using the selected method!



