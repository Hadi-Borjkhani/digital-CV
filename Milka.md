Dear user,

I understand that you're trying to connect a Libelium pulse oximeter to your Raspberry Pi 3 via Bluetooth to read raw data and display it on the serial output. You've provided a Python script using the `bleak` library, which is a great start. Let's address the issues in your script and guide you through the steps to achieve your goal.

---

**Issues in Your Current Script:**

1. **Typographical Errors:**
   - In `connect_and_read()`, the line `is_connected = await client_is_connected()` should be `is_connected = await client.is_connected()`.
   - In the same function, `printf(f" Charakteristik: {characteristic.uuid}")` should be `print(f" Charakteristik: {characteristic.uuid}")`.

2. **Undefined Functions:**
   - In your `main()` function, you call `await list_services(device_address)`, but `list_services()` is not defined in your script.

3. **Unused Functions:**
   - The `connect_and_read()` function is defined but never called.

4. **Script Structure:**
   - The script only runs `fetch_bluetooth_devices()` and doesn't proceed to connect to any device or read data.

---

**Steps to Achieve Your Goal:**

1. **Scan for Bluetooth Devices:**
   - Use `BleakScanner` to discover nearby Bluetooth devices.
   - Identify your pulse oximeter from the list (by name or address).

2. **Connect to the Pulse Oximeter:**
   - Establish a connection using `BleakClient` with the device's address.

3. **Discover Services and Characteristics:**
   - Once connected, list all services and characteristics to find the ones relevant to the pulse oximeter data.

4. **Read Data from the Device:**
   - Read from the specific characteristic(s) that provide the raw data.

5. **Display Data:**
   - Output the raw data to the serial console.

---

**Revised Script:**

Below is a revised version of your script incorporating the corrections and the steps mentioned:

```python
import asyncio
from bleak import BleakScanner, BleakClient

# Function to scan for Bluetooth devices
async def fetch_bluetooth_devices():
    devices = await BleakScanner.discover()
    for d in devices:
        features = {"name": d.name, "address": d.address, "rssi": d.rssi}
        print(features)
    return devices

# Function to connect to the pulse oximeter and read data
async def connect_and_read(device_address):
    async with BleakClient(device_address) as client:
        is_connected = await client.is_connected()
        print(f"Connected to {device_address}: {is_connected}")

        if is_connected:
            # Discover services and characteristics
            services = await client.get_services()
            for service in services:
                print(f"Service: {service.uuid}")
                for characteristic in service.characteristics:
                    print(f"  Characteristic: {characteristic.uuid}")
                    if "read" in characteristic.properties:
                        print("    (Readable)")
                        # Read data from the characteristic
                        try:
                            data = await client.read_gatt_char(characteristic.uuid)
                            print(f"    Data: {data}")
                        except Exception as e:
                            print(f"    Could not read data: {e}")

async def main():
    # Step 1: Scan for devices
    devices = await fetch_bluetooth_devices()
    
    # Step 2: Identify the pulse oximeter
    pulse_oximeter_address = None
    for device in devices:
        if device.name == "Pulse Oximeter":  # Replace with the actual name
            pulse_oximeter_address = device.address
            break

    if pulse_oximeter_address is None:
        print("Pulse oximeter not found.")
        return

    # Step 3: Connect to the pulse oximeter and read data
    await connect_and_read(pulse_oximeter_address)

# Run the main function
if __name__ == "__main__":
    asyncio.run(main())
```

---

**Instructions:**

- **Replace `"Pulse Oximeter"`** in the `main()` function with the actual name of your pulse oximeter as it appears in the scan results.

- **Ensure Bluetooth is Enabled:**
  - Make sure that Bluetooth is enabled on your Raspberry Pi and that the pulse oximeter is turned on and discoverable.

- **Permissions:**
  - You might need to run the script with elevated permissions using `sudo` if you encounter permission issues.

---

**Understanding the Script:**

- **Scanning Devices:**
  - The `fetch_bluetooth_devices()` function scans for nearby Bluetooth devices and prints their details.

- **Connecting to Device:**
  - The `connect_and_read()` function attempts to connect to the specified device address.
  - It checks if the connection is successful.

- **Discovering Services and Characteristics:**
  - Once connected, it retrieves all services and characteristics provided by the device.
  - It identifies readable characteristics.

- **Reading Data:**
  - It reads data from all readable characteristics and prints it.

---

**Identifying the Correct Characteristic:**

- **Device Documentation:**
  - Consult the Libelium pulse oximeter documentation to identify the specific UUID of the characteristic that provides the raw data.

- **Modify the Read Function:**
  - If you know the UUID, you can modify the `connect_and_read()` function to read from that specific characteristic:

    ```python
    characteristic_uuid = "your-characteristic-uuid"  # Replace with actual UUID
    data = await client.read_gatt_char(characteristic_uuid)
    print(f"Data: {data}")
    ```

---

**Additional Tips:**

- **Data Interpretation:**
  - The raw data read from the characteristic might be in bytes. You may need to decode or parse it according to the device's data format.

- **Error Handling:**
  - Implement try-except blocks to handle potential exceptions during connection and data reading.

- **Serial Output:**
  - Since you're interested in displaying the data on the serial output, ensure your script prints the data to the console, which you can view via SSH or directly on the Raspberry Pi.

---

**Final Remarks:**

- **Testing:**
  - Test the script incrementally. First, ensure the device appears in the scan results. Then, attempt to connect and read services. Finally, read the data.

- **Troubleshooting:**
  - If you encounter errors, provide the error messages for further assistance.

- **Dependencies:**
  - Ensure all required Python packages are installed:

    ```bash
    pip install bleak
    ```

- **Python Version:**
  - The `asyncio.run()` function is available in Python 3.7 and above. Ensure your Raspberry Pi is running a compatible Python version.

---

By following these steps and utilizing the revised script, you should be able to connect to your Libelium pulse oximeter and read the raw data. If you have any questions or run into issues, feel free to ask for further help.