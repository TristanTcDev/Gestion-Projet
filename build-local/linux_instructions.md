Linux (Debian-based distros)
====

Get IIWA
---
	
	> git clone --branch intelirris https://github.com/Waziup/smart-irrigation-waziapp.git intel-irris-waziapp-local
	> cd intel-irris-waziapp-local
	
For installing Flask and dependencies, using virtual environment
---
	> cd intel-irris-waziapp-local
	> python3 -m venv iiwa
	> . iiwa/bin/activate
	> pip3 install -r requirements.txt 
	> deactivate iiwa

Install WaziEdge and its dependencies. Only MongoDB up to v5.0 is supported.
---
	> sudo apt install golang-go
	
Download MongoDB Community Server from https://www.mongodb.com/try/download/community, select the correct platform/architecture and select the server package, then install the package with dpkg:

	> sudo dpkg -i mongodb-org-server_5.0.x_arch.deb (with x correponding to the subversion number and arch=amd64 or arm64)

Then, get WaziEdge source files:
	
	> git clone https://github.com/Waziup/wazigate-edge.git
	> cd wazigate-edge
	
Compile the source and build the wazigate-edge executable:
	
	> go build .
			
Run the IIWA local instance
---	

We will start MongoDB and wazigate-edge. In one terminal window:

	> sudo mongod --config /etc/mongod.conf &
	> cd wazigate-edge
	> sudo ./wazigate-edge

In another terminal, start the IIWA application:
	
	> cd intel-irris-waziapp-local
	> . iiwa/bin/activate
	> python3 app.py
	
Then open http://127.0.0.1:5000/ on your host computer's web browser.	

Adding default test devices to WaziEdge and configuring IIWA's configuration files
---

Before running the scripts below, ensure that WaziEdge and MongoDB servers are running. We can automatically add the starter-kit default test devices. In a terminal window:

	> cd intel-irris-waziapp-local
	> cd build-local/scripts
	> ./intel-irris-auto-config.sh
	
This will create the INTEL-IRRIS starter-kit default configuration which consist in a capacitive device (SOIL-AREA-1) and a tensiometer device (SOIL-AREA-2). These 2 devices are automatically added into the IIWA's configuration files. See [description of INTEL-IRRIS starter-kit default configuration](https://github.com/CongducPham/PRIMA-Intel-IrriS) for more detail.

**Note that, currently, IIWA only takes into account sensor data from `temperatureSensor_0` logical sensor which is the `Raw value from SEN0308` for a capacitive and the `centibars from WM200` for a tensiometer.**

You can then refresh http://127.0.0.1:5050/ to see that the 2 devices have been added to IIWA. Then you can quickly push in real-time some sensor data to the default starter-kit devices as follows, in another terminal window:

	> cd intel-irris-waziapp-local
	> cd build-local/scripts
	> ./push_starterkit_test_values.sh 170 15
	
170 is for the capacitive device (SOIL-AREA-1) and 15 is for the tensiometer device (SOIL-AREA-2). 

**Note that for the moment, the IIWA's dashboard only display information for the active device which is by default the first capacitive device.**

Then, it is now possible to change and modify IIWA's source code in real-time during development phase on your host computer.

After testing:

	> deactivate iiwa

Adding more devices
---

You can add more devices for your tests. To add a new capacitive device named `SOIL-AREA-3` with device's address `26011DAB`:

	> cd intel-irris-waziapp-local
	> cd build-local/scripts
	> ./create_new_capacitive.sh 3 AB
	
To add a new tensiometer device named `SOIL-AREA-4` with device's address `26011DB2`:	

	> ./create_new_tensiometer.sh 4 B2
	
**Note that the device's address is actually not really important for IIWA. However, we still keep it as these scripts use real scripts to add real devices	to the INTEL-IRRIS system.**

**What is important is to avoid assigning same device name, e.g. several `SOIL-AREA-1` for instance.**

After the creation of new devices, you can also push new sensor data as follows:

	> ./push_device_test_value.sh 3 170
	
which will push value 170 to device `SOIL-AREA-3`. In our example it is the newly created capacitive device.	

	> ./push_device_test_value.sh 4 15
	
will will push value 15 to device `SOIL-AREA-4`. In our example it is the newly created tensiometer device.	

You can alternatively use the more generic script `push_sensor_test_value.sh`:

	> ./push_sensor_test_value.sh 63bfeddd6e45da24473eca6e temperatureSensor_0 170
	> ./push_sensor_test_value.sh 63bfeddd6e45da24473eca72 temperatureSensor_0 15
	
assuming your capacitive device id is `63bfeddd6e45da24473eca6e` and your tensiometer device id is `63bfeddd6e45da24473eca72`.

