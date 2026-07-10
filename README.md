Awesome! Setting up the environment on Ubuntu is where the real magic begins. Since this involves modifying the Linux kernel to unlock the AX211, here is your step-by-step roadmap for exactly what to do once you boot into your new Ubuntu 22.04 installation:

### Step 1: System Prep & Copying the Project
1. Boot into Ubuntu and connect to the internet (preferably via a wired Ethernet cable or USB tethering, because we are about to mess with your Wi-Fi driver).
2. Open a terminal and install the basic build tools:
   ```bash
   sudo apt update
   sudo apt install build-essential git python3-pip python3-venv
   ```
3. Copy your entire `WiFi-Spatial-Mapper` folder into your Ubuntu Documents or Home folder.

### Step 2: Install PicoScenes (The Magic Driver)
PicoScenes is the platform that will replace your default Intel Wi-Fi driver with a custom one that exposes the CSI data.
1. Open their official installation guide: [PicoScenes Installation](https://picoscenes.com/docs/installation/)
2. Run their automated installer script in your terminal (check their site for the most up-to-date command, but it generally looks like this):
   ```bash
   wget -O - https://raw.githubusercontent.com/PicoScenes/PicoScenes/master/scripts/install.sh | bash
   ```
3. During the installation, it will detect your Intel AX211 and ask if you want to install the `iwlwifi` plugin. **Say Yes.**
4. Once it finishes, **Reboot** your computer to load the new kernel module.

### Step 3: Start the Hardware Stream
Once rebooted, your Wi-Fi card is now a radar! You need to tell PicoScenes to turn on the card, listen to ambient Wi-Fi traffic, and forward the raw CSI data to our Python script.
1. Open a terminal and run PicoScenes with UDP forwarding. The command will look something like this (refer to PicoScenes documentation for exact flags):
   ```bash
   sudo PicoScenes -d iwlwifi --mode monitor --udp-address 127.0.0.1:5500
   ```
2. Your terminal will start printing that it is capturing frames!

### Step 4: Run Our Python Software!
Leave the PicoScenes terminal running in the background. Open a **new** terminal, navigate to your copied project folder, and run the exact same steps we used on Arch:
```bash
# Setup the environment (only need to do this once)
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

# Run the calibration on the empty room (10 seconds)
.venv/bin/python tests/calibrate_room.py

# Launch the live GUI!
.venv/bin/python tests/ambient_mapper.py
```

Because our script listens to `127.0.0.1:5500`, it will seamlessly pick up the *real* hardware frames streaming from PicoScenes instead of the dummy daemon. 

Let me know if you hit any snags during the PicoScenes installation—kernel patching can sometimes be a bit finicky!
