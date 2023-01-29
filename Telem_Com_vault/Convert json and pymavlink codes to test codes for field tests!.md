### How the test should be?
I think the right off the bat solution would be having a sample data collection known on both sides. This collection is sent by the GCS to the RPI and RPI checks the equallity of received message and what should be sent by the GCS. If they were equal, so increament a counter, otherwise do nothing.
This procedure can be done in oposite direction. To save the result, we can save them as a text file in the receiver side!
On optional thing we can add is adding a RF message for starting the test!

__Note:__ This method of checking the equallity of the received and reference data, leads to some corner cases and complex logic which is hard to handle and I can't stablish some good statistic measurments to verify that and for now I have nore more time. So I will just receive messages and save them as text log files and inspect them visually!
Something that my heart says to me is that the mavlink is industry prooved, So with no doubt you shoud use it!
Let's do the job!

### Test code with the `json` module
