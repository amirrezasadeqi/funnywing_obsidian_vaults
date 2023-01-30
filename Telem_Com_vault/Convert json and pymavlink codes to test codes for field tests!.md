### How the test should be?
I think the right off the bat solution would be having a sample data collection known on both sides. This collection is sent by the GCS to the RPI and RPI checks the equallity of received message and what should be sent by the GCS. If they were equal, so increament a counter, otherwise do nothing.
This procedure can be done in oposite direction. To save the result, we can save them as a text file in the receiver side!
On optional thing we can add is adding a RF message for starting the test!

__Note:__ This method of checking the equallity of the received and reference data, leads to some corner cases and complex logic which is hard to handle and I can't stablish some good statistic measurments to verify that and for now I have nore more time. So I will just receive messages and save them as text log files and inspect them visually!
Something that my heart says to me is that the mavlink is industry prooved, So with no doubt you shoud use it!
Let's do the job!

### Test code with the `json` module
You can find the code [here](https://github.com/amirrezasadeqi/funnywing/blob/RF_communication/wing_ros_ws/src/wing_field_tests/scripts/json_rfcom_test.py). Nothing too special about this code, just adding `'\n'` after the serialized messages(they are serialized to strings) and sending them. In the receiver side reading the data with `readline` function and striping `'\n'` from the end of the message and decode them using `json` module. After all logging the number of sended messages and content of the received data.
### Test code with `pymavlink` package
[The code](https://github.com/amirrezasadeqi/funnywing/blob/RF_communication/wing_ros_ws/src/wing_field_tests/scripts/pymav_rfcom_test.py) for this test is pushed to the github of funnywing. The special things about this code are:
- I learned using ternary operator in python:
```python
variable = (value_on_true) if (condition) else (value_on_false)
```
- To use funnywing specific dialect with pymavlink, you need below code:
```python
# setting the mavlink version when using mavutil for connection management
os.environ["MAVLINK20"] = "1"  # 1:MAVLINK20, 0:MAVLINK10
# Using funnywing Dialect which includes ArduPilot messages
port = mavutil.mavlink_connection(
    args.serial_port, baud=args.baudrate, dialect="funnywing")
```
- To prevent the reader thread from locking, specially when terminate the node with the Ctrl-C, you should specify the `timeout` parameter in the `recv_match` function like below:
```python
msg = port.recv_match(blocking=True, timeout=1.0)
```
There are no other important things in the code, and now you can test them.

Let's do the job!
