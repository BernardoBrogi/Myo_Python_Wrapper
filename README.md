This work is based on the [myo-python](https://github.com/NiklasRosenstein/myo-python) repository.

# Myo Python Wrapper

Python bindings for the Thalmic Labs Myo SDK, plus a small set of example scripts for reading device state, streaming EMG, and plotting live data.

## Requirements

This repository is currently being used on Windows 11 with Python 3.10.

- Windows 11
- Python 3.10
- The Myo Connect app running in the background
- The Myo SDK extracted locally and passed to `myo.init()`

The official SDK is no longer distributed by Thalmic. To download the SDK used for this project, use [this release](https://github.com/NiklasRosenstein/myo-python/releases/tag/v1.0.4).

## Installation

Install the package into a virtual environment and make sure the Myo dongle is connected before running any script.

```bash
python -m venv .venv
.venv\Scripts\activate
pip install -e .
```

If you want to run the live plotting example, also install:

```bash
pip install numpy matplotlib
```

## Basic Usage

```python
import myo


class Listener(myo.DeviceListener):
  def on_connected(self, event):
    print("Hello, {}!".format(event.device_name))
    event.device.vibrate(myo.VibrationType.short)

  def on_unpaired(self, event):
    return False

  def on_orientation(self, event):
    orientation = event.orientation
    acceleration = event.acceleration
    gyroscope = event.gyroscope
    # Use the sensor data here.


if __name__ == '__main__':
  myo.init(sdk_path='./myo-sdk-win-0.9.0/')
  hub = myo.Hub()
  listener = Listener()
  while hub.run(listener.on_event, 500):
    pass
```

If you prefer a stateful helper instead of implementing every callback yourself, use `myo.ApiDeviceListener` to read the most recent device state.

```python
import myo
import time


def main():
  myo.init(sdk_path='./myo-sdk-win-0.9.0/')
  hub = myo.Hub()
  listener = myo.ApiDeviceListener()

  with hub.run_in_background(listener.on_event):
    print("Waiting for a Myo to connect ...")
    device = listener.wait_for_single_device(2)
    if not device:
      print("No Myo connected after 2 seconds.")
      return

    print("Hello, Myo! Requesting RSSI ...")
    device.request_rssi()
    while hub.running and device.connected and not device.rssi:
      print("Waiting for RSSI...")
      time.sleep(0.001)

    print("RSSI:", device.rssi)
    print("Goodbye, Myo!")


if __name__ == '__main__':
  main()
```

## Examples

All examples live in the [examples](examples) folder and assume that Myo Connect is running, the dongle is paired, and `myo.init()` can find the SDK on disk.

### `01_hello_myo.py`

Minimal listener example. It prints a greeting when a device connects, requests the battery level, vibrates once, and exits when you perform a double tap.

Run it with:

```bash
python examples/01_hello_myo.py
```

### `02_display_data.py`

Terminal dashboard example. It shows orientation, pose, RSSI, lock state, and EMG data when streaming is enabled. Double tap turns EMG streaming on, and finger spread turns it off.

Run it with:

```bash
python examples/02_display_data.py
```

### `03_live_emg.py`

Live EMG plot example. It streams EMG data into a rolling matplotlib chart with eight channels.

Run it with:

```bash
python examples/03_live_emg.py
```

Before running this one, install `numpy` and `matplotlib`.

### `04_emg_rate.py`

EMG rate monitor example. It prints the approximate EMG callback rate in the terminal.

Run it with:

```bash
python examples/04_emg_rate.py
```

### `05_api_listener.py`

Stateful listener example. It waits for a single device, requests RSSI, and reads the latest values through `ApiDeviceListener`.

Run it with:

```bash
python examples/05_api_listener.py
```

## Notes

- The Myo SDK path in the examples is just a placeholder. Update it to match where you extracted the SDK on your machine.
- If you are adapting the examples for a different Python version, check any standard-library calls that may have changed over time.