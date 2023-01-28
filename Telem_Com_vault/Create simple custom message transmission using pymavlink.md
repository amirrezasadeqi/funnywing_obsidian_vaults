[Getting Started with mavlink](https://mavlink.io/en/getting_started/)
The workflow should be:
- Installing mavlink for custom messages. So we can't use `pip` to install `pymavlink` and we should fork from the desired mavlink fork and add custom messages to that. [You can find the proper way of doing this in mavlink documentation.](https://mavlink.io/en/mavgen_python/#generate-a-custom-mavlink-dialect) (I think we should do this in both RPI and GCS system). Note that I have used the [funnywing_mavlink](https://github.com/amirrezasadeqi/funnywing_mavlink) fork which I forked from [ArduPilot/mavlink](https://github.com/ArduPilot/mavlink).
- Generate the custom messages using [this tutorial](https://mavlink.io/en/getting_started/generate_libraries.html) from mavlink site. Note that for creating your custom messages you should write xml files called ==Dialects==. A good documentation to learn the contents of these files can be found [here](https://mavlink.io/en/guide/define_xml_element.html) and [here](https://mavlink.io/en/guide/xml_schema.html). Furthermore you can include the other dialects into your message set and here we will include the message sets of `ArduPilot` project which can be found in [this link](https://mavlink.io/en/messages/ardupilotmega.html) (`ardupilotmega.xml` file). The list of other dialects can be found [here](https://mavlink.io/en/messages/#dialects). A good example dialect to start writing your dialect is [test.xml](https://github.com/ArduPilot/mavlink/blob/master/message_definitions/v1.0/test.xml).
- Now you should use the generated codes for your message set in your code. You can find some useful documents for doing this in [here](https://mavlink.io/en/mavgen_python/#using-the-python-mavlink-libraries). Also pymavlink github containes [some good examples of pymavlink usage](https://github.com/ArduPilot/pymavlink/tree/master/examples), like mavtest.py and ... .

Now let's write the first code with pymavlink library! Let's Go ...

Consider adding your custom message mavlink project as a submodule to the funnywing project using [this atlassian tutorial](https://www.atlassian.com/git/tutorials/git-submodule).

## Procedure of Creating an example for sending and receiving custom mavlink messages using `pymavlink`
In this example we will create two ROS nodes that each of them has two thread, one for sending and another for receiving. To be able to send custom messages, the first step is generating the pymavlink library according to your custom dialect. For this task you can fork from the `ArduPilot/mavlink`(not `mavlink/mavlink`, since we want to work with ardupilot system.) to put your custom messages in there. Here Our fork is [`funnywing/mavlink`](https://github.com/amirrezasadeqi/funnywing_mavlink). Now you can follow [mavlink documentation on installation of the mavlink toolchain](https://mavlink.io/en/getting_started/installation.html#installation). Do not forget to set the `PYTHONPATH` variable to the root directory of your clone.
Now you have access to tools for generating the pymavlink libraries for your dialect, then create a `xml` file for your dialect in the `<fork_root>/message_defenitions/1.0/` and put your custom enums and messages in there(==create a branch for repository modification==). You should also include the `ardupilotmega` dialect in yours to have access to ardupilot specific messages. For details about the content of the file, refer to the documentations about dialects in the mavlink site. Finally, you can generate the the libraries using [GUI](https://mavlink.io/en/getting_started/generate_libraries.html#mavgenerate) or [command line](https://mavlink.io/en/getting_started/generate_libraries.html#mavgen) tools in the mavlink library. For example for command line, you can use below command:
```shell
python3 -m pymavlink.tools.mavgen --lang=Python --wire-protocol=2.0 --output=<fork_root>/include/<library_name>.py <fork_root>/message_definitions/v1.0/<your_custom_dialect>.xml
```

Note that the `<fork_root>/include` is in git ignore file!
`pymavlink` is a git submodule for the mavlink repository, so you can install `pymavlink` with your generated custom message classes. To do this you can follow [these instructions](https://mavlink.io/en/mavgen_python/#generate-a-custom-mavlink-dialect). So just copy the generated python file to the `<fork_root>/pymavlink/dialects/<v10_or_v20>/` directory(`v20` is for mavlink version 2). Now change directory to the `pyamavlink` submodule and install the pymavlink to your system using below command:
```shell
python setup.py install --user
```
Do not forget to uninstall the old pymavlink using `pip uninstall pymavlink`!
Now, you have `pymavlink` installed on your system and you can code with that. There is only one tricky thing! You can manage the connection(here the serial connection) by your self or by the library. if you want to handle the connection by yourself, you can import the dialect directly, like below:
```python
# Import ardupilotmega module for MAVLink 1
from pymavlink.dialects.v10 import ardupilotmega as mavlink1

# Import common module for MAVLink 2
from pymavlink.dialects.v20 import common as mavlink2
```
If not, you should use the `mavutil` module and `mavutil.mavlink_connection` function. Here is the tricky spot. By default the `pymavlink` uses the `ardupilotmega` dialect and MAVLink1 version. So [according to documentations](https://mavlink.io/en/mavgen_python/#dialect_file) you should set some environment variables(which I found the `MAVLINK20` necessary, 1 for mavlink2) and consequently the mavutil will use the correct dialect library and you'll be able to use the `<custom_message>_send()` function for sending your custom messages. For setting these environment variable you can use the python `os` module. Also note that in the `mavtuil.mavlink_connection` you should specify the dialect(just the name of the file as string without the `.py` and `.xml` ) if you don't want to set the corresponding environment variable. furthermore, you should set these variables before creating the connection. The corresponding code will be:
```python
# Note that we set to "1" not 1
os.environ["MAVLINK20"] = "1"  # MAVLINK20
rf_connection = mavutil.mavlink_connection(
    "/dev/ttyUSB0", baud=57600, dialect="funnywing")
```
For more information on how to use this library for send/receive data, refer to [corresponding parts in documentation](https://mavlink.io/en/mavgen_python/).
The code for GCS will be like below box:
```python
#!/usr/bin/env python

import rospy
from pymavlink import mavutil
import threading
import time
import os

# setting the mavlink version when using mavutil for connection management
os.environ["MAVLINK20"] = "1"  # MAVLINK20
rf_connection = mavutil.mavlink_connection(
    "/dev/ttyUSB0", baud=57600, dialect="funnywing")


def writer_worker():
    wc = 0

    while not rospy.is_shutdown():
        rospy.loginfo("GCS writer thread writes!")
        rf_connection.mav.hello_world_send(
            mavutil.mavlink.SENDER_ID_GCS, b"GCS", wc)
        wc += 1
        time.sleep(0.5)

    return


def reader_worker():
    while not rospy.is_shutdown():
        msg = rf_connection.recv_match(blocking=True)
        if not msg:
            return
        if msg.get_type() == "BAD_DATA":
            if mavutil.all_printable(msg.data):
                print(f"Bad Data from RPI: {msg.data}")
        else:
            # Valid message
            print(
                f"GCS reader thread reads: (SENDER_ID: {msg.sender_id}, SENDER_NAME: {msg.sender_name}, SEQUENCE: {msg.sequence})")
    return


if __name__ == "__main__":
    rospy.init_node("gcs_rf_node")

    writer_thread = threading.Thread(target=writer_worker)
    reader_thread = threading.Thread(target=reader_worker)

    writer_thread.start()
    reader_thread.start()

    # while True:
    #     rf_connection.wait_heartbeat()
    #     print("Heartbeat from system (system %u component %u)" %
    #           (rf_connection.target_system, rf_connection.target_component))

```
__Note that in the `<message>_send` function, strings should be passed as bytes, like `b"GCS"`.__
Code for the RPI side:
```python
#!/usr/bin/env python3

import rospy
from pymavlink import mavutil
import time
import threading
import os

# setting mavlink version to mavlink20
os.environ["MAVLINK20"] = "1"
rf_connection = mavutil.mavlink_connection("/dev/ttyUSB0", baud=57600, dialect="funnywing")

def writer_worker():
    wc = 0

    while not rospy.is_shutdown():
        rospy.loginfo("RPI writer thread writes!")
        rf_connection.mav.hello_world_send(mavutil.mavlink.SENDER_ID_RPI, b"RPI", wc)
        wc += 1
        time.sleep(0.5)
    return

def reader_worker():
    while not rospy.is_shutdown():
        # receiving any kind of data
        msg = rf_connection.recv_match(blocking=True)
        if not msg:
            return
        if msg.get_type() == "BAD_DATA":
            if mavutil.all_printable(msg.data):
                print(f"Bad Data from GCS: {msg.data}")
        else:
            # Valid message
            print(f"RPI reader thread reads: (SENDER_ID: {msg.sender_id}, SENDER_NAME: {msg.sender_name}, SEQUENCE: {msg.sequence})")
    return


if __name__ == "__main__":
    rospy.init_node("rpi_rf_node", anonymous=True)

    writer_thread = threading.Thread(target=writer_worker)
    reader_thread = threading.Thread(target=reader_worker)

    writer_thread.start()
    reader_thread.start()

    # rf_connection.mav.heartbeat_send(mavutil.mavlink.MAV_TYPE_ONBOARD_CONTROLLER, mavutil.mavlink.MAV_AUTOPILOT_INVALID, 0, 0, 0)

```

Now Let's Do the job with these nice things!
