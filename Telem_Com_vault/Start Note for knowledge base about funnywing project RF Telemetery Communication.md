This first thing that I've founde about the connection of __9XTend__ is that we need to use the module that Mr.Rezvani developed to be able to communication the data with the Computer. From Computer I mean PC or raspberry pi board.

I have tried to connect the RF module to raspberry pi directly with the pi pinout but I could not receive the data comming from the mate module which is connected into pixhawk board. On the other hand when we use the Mr.Rezvani usb module the data nicely transfers even with raspberry pi board and we can infer that the power supply is not a problem.

So as I understood the problem is about the protocol of connection. consequently, I tested this connection with __PL2303__ module which we had bought in our shoppings. but it does not work. I will test this module more in next tries.

Mr.Rezvani said that you must also use the pin 7 of the 9XTend module to enable the module to work. This pin is __shutdown__ pin and during operation must be high. Also, If you have looked at the pinout in datasheet, you would be informed that this pin is a __"must use pin"__ for the module. So read the datasheet with more concentration!
[XTend moduel documentation](https://www.digi.com/resources/documentation/digidocs/pdfs/90002166.pdf) 

Some Electrical concepts that are good to know:
- [[TTL and RS232]]: somethings releated to voltage level of logical electronic parts and modules.
- Working Modes of 9XTend Module:
	- Transparent Operation: [Some good information in this link](https://www.digi.com/resources/documentation/Digidocs/90001942-13/concepts/c_transparent_mode_detailed.htm?tocpath=XBee%20transparent%20mode%7CXBee%20transparent%20mode%20in%20detail%7C_____0) 
	-  API Operation: [Some good information in this link](https://www.digi.com/resources/documentation/Digidocs/90001942-13/Default.htm#concepts/c_api_mode_detailed.htm?TocPath=API%2520mode%257C_____1)
Air Module MAC address: 0013A200404A9EAB
Ground Module MAC address: 0013A2FFFFF112CB
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

#### Now let's see how XBee python library can work for us!
I think we can't use the XBee python library, when the module is in transparent operating mode and if we insist on using transparent mode, I think we should use python standard serial module. For more information check [this part of xbee python library documentation](https://xbplib.readthedocs.io/en/latest/user_doc/working_with_xbee_classes.html#device-operating-mode).
So now, we should check if we can use API operating mode for XTend modules for connecting into pixhawk.
I've checked the 

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

#### List of TODOs for adding support for RF communication
- For RPI board implement these:
	- [X] Server
	- [ ] Subscriber node for sensor data -> this sends data to GCS, like GPS data
	- [ ] Client node -> this recives commands from GCS, deserializes the commands, calls corresponding services, serializes the answer and sends it back to GCS
 
- For GCS PC we should do these:
	- [x] Adding seperate page for RF communication in GUI app.
	- [X] Add subscription to sensor data and visualize it in GUI app.
	- [ ] Test the sensor subscription(previous task.)
	- [x] Add publisher to the GUI app for sending commands to the corresponding topic. 
	- [ ] Test the pulisher which sends packed navigation commands from GUI app to the corresponding topic for feeding RF link interface node.
	- [ ] Add a Publisher node which recieves data from RF link and publishes to corresponding topics to feed the GUI app.
	- [ ] Add a subscriber node subscribing to the command topics, serializing them and sending them to RPI via RF link. This node should also get the command answers back via RF link and provide them to GUI app. This last task may also be done in the previous publisher task(data from RF link), but I'm not confident about it yet, but I think here is better for implementing it, because in this way we gather same contexts together!

__NOTE__: It may be ok to have seperate nodes with interface to RF module or in another words serial port. But I think at the moment(that I have no time and want to be faster) or even at any other situation, it is better to have something like mavros. I mean having only one interface to the link and all the data transfers being passed through this interface. I think this way have some advantages as below:
- Thinking about the communication is easier and developement and maintanace would be easier.
- Prevents wierd behaviors of busy port and threading problems and if there were some errors that would be more managable and easier to solve.
- so on
I don't know that the other solution(multiple RF interfaces) how would be, but I think it is not a good one, but anyway I will think about and check it or even implement it in the future and verify its quallity. So in the future ...
Now let's do the job as fast as possible with one single ROS node as the single or only RF/serial link interface!
Now let's implement these tasks!

### How the structure of Guidance commands should be?
In the fourth task for GCS PC, we need to send commands into the RPI via RF link. But how the structure of the sending data should be?
Is it enough to use something like below for Simple GOTO command(space as delimeter):
SG lat lon alt
Or we need something more complex, like using pickle, protobuf and so on. Regardless of selected method, In the RPI side, we need to parse the recieved data to be able to call the corresponding service.

After some little thinking, I infered that using a de/serialization library is a better way, because of below reasons:
- The library converts the data to an usable format or data structure in your language and so you should just use it. But in the case of implementing the serialization by yourself, you need to provide the de/serialization function of each command type you add to your system. So your life would be easier.
- Some advantages like encrypting and ...
Also, at the moment I'm not 100 percent confident about the fact that using de/serialization causes the better performance and creation of extra values, But I've decided to use a library for that.
Note that the usage of library for de/serialization does not the task of parsing the command and I must do it manaully. The probable solutions could be the handler dictionary, Inheritance of command classes handling the corresponding service call, simple `if else` statements and so on. I will decide for this in future ...
Now let's choose the de/serialization library and and create the publisher of guidance commands.
__Note__: a nice note that I've found is the fact that(I think), using de/serialization you can connect different programs with different languages.

### De/Serialization libraries
[This article](https://medium.com/@shmulikamar/python-serialization-benchmarks-8e5bb700530b) is a good place to be familiar with different de/serialization python libraries and 2 or 3 terminologies in this area. The article also provides a benchmark for performance comparision between the introduced packages. The most appealing libraries for me are elaborated in below list:
- pickle: [The pickle module is not secure. Only unpickle data you trust.](https://docs.python.org/3/library/pickle.html). This is stated in the documentation. Since we are not very expert to handle this unsecurity, so we don't use it.
- Google Protobuf package: This is very attractive for me because this library is very common in machine learning and TensorFlow. Also, among introduced libraries, this library's serialization output size is the second least size and this is good feature for us because we need to transfer this output over RF connection. I think the drawback of this library is its speed(which in our case is not a problem, because we don't de/serialize too many messages) which is relativly lower than the other introduced packages. Also, we need to compile `.proto` files into corresponding python codes, which I don't like this fact, since it can make the project structure too complicated and is not necessary in our case. The other drawback of this library is that(I think) it only supports simple data types and in future of our project we may need to send more complex data type over RF port and I don't want to become restricted. Consequently, I don't use this package for this project, but I will study and use it for my next projects, specially ML projects. So in future ...
- JSON and Ultra-JSON: The json library is in python standard libraries but is comparitativly slow and produces large serialized objects. so Ultra-JSON is its faster alternative. The drawback of UJSON is its less accuracy in floating points and the fact that some options of JSON isn't implemented in it.
- msgpack: This is one of the fastest de/serialization libraries producing relativly small sized output(based on the represented benchmark, larger than protobuf). There are many different implementation for this library in different languages like python, C++ and so on. It does not need any seperate message description file like protobuf and also handles complex data type serialization like [dataclass](https://realpython.com/python-data-classes/)(also refer to [this link](https://docs.python.org/3/library/dataclasses.html)) in [__aviramha__](https://github.com/aviramha/ormsgpack) implementation. Furthermore, it supports data types like `numpy` and so on. It also very recommended by the author of the medium article.
There are also some more packages introduced in the article and over the internet, but for now I don't want to spend more time on this. I think the most powerful packages which are worth to learn and use in future(may be more than enough in this project, but why not use a powerful one for future extendability) are, Google Protobuf and msgpack. Because of previously stated reasons, we don't use protobuf here(I will use it later and specially in ML projects). Consequently, the only choice is [msgpack](https://msgpack.org/) and the [aviramha](https://github.com/aviramha/ormsgpack) implementation of it, because (I think) this implementation is more compelete. From more compelete, I mean that (I think) among python implementation, only this one implements the serialization of dataclasses which actually provide the ability to serialize the class member functions and also some powerful inheritance features. For more information about data class go to [real python tutorial](https://realpython.com/python-data-classes/).

In future I will work more on this subject and in future ...

Now let's construct funnywing with __msgpack__!
__Note__: At the moment, I could not install the ormsgpack on RPI, So I've decided to work with the standard/official msgpack and I will try the installation of ormsgpack on RPI later. I think the problem can be bad internet connection. So I will check that!



#### TODO:
- [x] Check the RF module profiles with Pixhawk board.
	Thanks to GOD! It seems that pixhawk has no problem with the setting profile which is used for 9XTend RF module connecting to the RPI/CUAV v5+ pixhawk(air module). So I think we can use the air module profile for pixhawk too and also for tests which will be performed/conducted in the field.

Let's go ...


#### Python and Software new things which I learned:
- GIL: Global Interpreter Lock, is a mutex or lock which allows only one thread to be executable, even in multithreaded programs. This can be a bottleneck of the program!
- dataclass: It is like class, but it is intended to be used for data. For more info about it go to realpython tutorial about [dataclasses](https://realpython.com/python-data-classes/).
- Backward compatibility: when a newer version of a system works with older version interfaces of the same system, we call that system backward compatible. for example consider the case of microsoft word 2010 which can read and open older versions back to word 2007. Also 80486 processor can work with programs for 80386, so it is backward compatible.
- Simple but nice way to have something like C++ structure in python: To create a template for my commands I need something like structures in C++. note that I could use python class or dataclass, but they are not supported by official msgpack implementation. So I need to use dictionaries. This [stackoverflow question](https://stackoverflow.com/a/19673465/10243689) provides a nice and simple way. we just need to write a function returning the dictionary with our tempelate and name it like a data type. you can also use the [`@staticmethod`](https://www.geeksforgeeks.org/class-method-vs-static-method-python/) decorator in a class for defining these structure creator functions. so we have a simple data structure in python!

## Now I think we should work bese on facts and be reasonable
There are two problems which I can't spend more time on that:
- __RF module operating mode__: pixhawk can't work with API mode and on the other hand in transparent mode we can't use the xbee python library. so because I can't spend more time and I want to be agile in developement, I will use the transparent mode and python serial library. In future I will work on learning API mode and XBee python library, so in future ...
- __Can't install ormsgpack on RPI, so we use msgpack__: I don't know if the problem is because of bad network connection or something else, but I have no more time to spend on that so I wil use official/standard msgpack implementation. In future I will work on learning/installation and work with ormsgpack or implement my own or extend msgpack for things like python dataclass and now just do the work using python msgpack. So in future I will ...

Now Let's do the job with ==__official msgpack python implementation__== and ==RF module __transparent operating mode__==. In future ... . Be fast! Let's go ...
To do this I will provide [[important notes about msgpack python library]] in a seperate node.
for working with RF module in transparent operating mode, we can use the python standart module `serial`. If there were any important note about this library I will provide it in the [[important notes about python standart serial library]].

I moved the vaults to github account!

__Note about commands topic__
I think we should have one command topic containing the serialized commands. This is because of two resons, as below:
- firstly, to prevent making the ROS network very crowd and dirty.
- secondly, if we want add more commands, in this way we don't need to add more topics and message types.
- thridly, I think in this way the commands will be sent/served sequentially and not in parrallel and in this way we will not be confused by serving the commands in parallel which may cause in complex scenarios(may be!). In future, If we will need sending/creating/serving commands in parallel we would think on that and implement it in a nice way. For now just use this simple method(sequentially send/serve commands) and do the job. In future ...

Now let's do the job ...

### Publish and Subscribe an msgpack-ed encoded/decoded python object in ROS network
Note that you can't do this using python `str` and in the unpacking of the encoded data, the deserializer does not accept `str` and wants `bytes-like` object. This fact made it difficult to figure out how we should send such a data or what ros message type is compatible with this data. So after some researchs, [WHOI(ros-acomms)](https://git.whoi.edu/acomms/ros_acomms) project and specifically [Packet.msg](https://git.whoi.edu/acomms/ros_acomms/-/blob/master/msg/Packet.msg#L14) solved the problem. Actually we can use [UInt8MultiArray](http://docs.ros.org/en/api/std_msgs/html/msg/UInt8MultiArray.html) standard ROS message for this task. The documentation about this type of message is so confusing, so I searched about how we should set the the fields of such a message, especially the `layout` field. In python(and I think for our simple case of a data buffer) we don't need to set the data field and it is enough to just set the `data` field of the message as it is illustrated in this [stackoverflow question](https://stackoverflow.com/a/31378414/10243689) and this [robot ignition academy discussion session](https://get-help.robotigniteacademy.com/t/publishing-multi-dimensional-array/8895/4). For C++ and more complex data types, like images and so on, I think we also must set the `layout` field. So in the future ...

__NOTE__: I think it is not so good to use UInt8MultiArray for sending general messages(I think specially such a message which can contain anything, because of security things and I think we should add some security layer to compensate this problem), But at the moment I have not any other idea in my head about how to fastly implement publisher/subscriber for the funnywing commands topic(Note that type of commands can evolve or change as time passes). So In the future ...

Now Let's progress the implementation of the funnywing project!

I think my serial ports connecting the RF modules to the GCS PC and RPI are [[half-duplex]], so at a specific time, we can only write or only read to/from serial port. In another words, the data flow is uni-directional at any specific time. This [stackoverflow answer](https://stackoverflow.com/a/74182813/10243689) can be helpful in implementing RF module codes.

#### Setting Up the old RF module(4wire air module)
The wiring that collegues have done was very funny. So I checked it and used my own standard for it by adding some wire extensions as below:
	Black -> GND
	Red -> VCC (5v)
	White -> RXD for air module and TXD for USB to TTL module
	Gray -> TXD for air module and RXD for USB to TTL module
Using screen on RPI and GTKTerm on PC I tried to test different settings for ports and finally it worked using `/dev/ttyUSBX 9600-8-N-1`. __Note__ that now I try to implement my codes using this module but the major module (9XTend) may have different behaviors and I hope my code also work on that! but if not this different behavior can be the potential source of issue!
[[Some Notes about reading and writing on serial connection(RF link)]]

==Let's do the job!==:
[[RF communication kanban board]]






