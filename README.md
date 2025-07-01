# PyWattBox 800

A Python client library for controlling SnapAV WattBox power distribution units using the Integration Protocol v2.4.

## Overview

PyWattBox 800 provides a simple, robust Python interface for communicating with SnapAV WattBox devices over Telnet or SSH connections. This library supports comprehensive outlet control, power monitoring, UPS status, and device management capabilities.

## Features

- **Complete Device Control**: Turn outlets on/off, toggle, reset with optional delays
- **Power Monitoring**: Get real-time power consumption data for individual outlets and system-wide status
- **UPS Integration**: Monitor UPS status including battery health, charge level, and runtime
- **Auto Reboot Management**: Configure and monitor automatic reboot functionality
- **Device Information**: Access firmware version, model, hostname, and service tag
- **Network Discovery**: Scan for WattBox devices on your network
- **Robust Communication**: Built-in error handling, authentication, and connection management
- **Context Manager Support**: Clean resource management with `with` statements

## Supported WattBox Models

This library supports all WattBox 800 series models and is compatible with:

- WB-800-IPVM series (12+ outlet models)
- Other WattBox models supporting Integration Protocol v2.4
- Both Telnet (port 23) and SSH (port 22) connections
- Firmware versions 1.3.0.4+ for SSH support

## Installation

Currently, this library is not published to PyPI. Clone the repository and install locally:

```bash
git clone https://github.com/your-username/pywattbox_800.git
cd pywattbox_800
pip install -e .
```

### Dependencies

- Python 3.7+
- No external dependencies (uses only Python standard library)

## Quick Start

### Basic Usage

```python
from pywattbox_800 import WattBoxClient

# Connect to your WattBox device
with WattBoxClient(host="192.168.1.100", username="wattbox", password="wattbox") as client:
    # Get device information
    device_info = client.get_device_info()
    print(f"Connected to {device_info.system_info.model}")
    print(f"Firmware: {device_info.system_info.firmware}")
    print(f"Outlets: {device_info.system_info.outlet_count}")
    
    # Control outlets
    client.turn_on_outlet(1)    # Turn on outlet 1
    client.turn_off_outlet(2)   # Turn off outlet 2
    client.toggle_outlet(3)     # Toggle outlet 3
    client.reset_outlet(4, delay=5)  # Reset outlet 4 with 5-second delay
    
    # Get outlet status
    outlets = client.get_all_outlets_info(include_power_data=True)
    for outlet in outlets:
        print(f"Outlet {outlet.index}: {outlet.name} - {'ON' if outlet.status else 'OFF'}")
        if outlet.power_watts:
            print(f"  Power: {outlet.power_watts}W, {outlet.current_amps}A, {outlet.voltage_volts}V")
```

### Power Monitoring

```python
with WattBoxClient(host="192.168.1.100") as client:
    # Get system-wide power status
    power_status = client.get_power_status()
    if power_status:
        print(f"Total Power: {power_status.power_watts}W")
        print(f"Total Current: {power_status.current_amps}A")
        print(f"Voltage: {power_status.voltage_volts}V")
    
    # Get individual outlet power data
    outlet_power = client.get_outlet_power_status(1)
    if outlet_power:
        print(f"Outlet 1 Power: {outlet_power.power_watts}W")
```

### UPS Monitoring

```python
with WattBoxClient(host="192.168.1.100") as client:
    # Check if UPS is connected
    if client.get_ups_connection_status():
        ups_status = client.get_ups_status()
        print(f"Battery Charge: {ups_status.battery_charge}%")
        print(f"Battery Health: {ups_status.battery_health}")
        print(f"Runtime Remaining: {ups_status.battery_runtime} minutes")
        print(f"Power Lost: {ups_status.power_lost}")
```

### Device Discovery

```python
from pywattbox_800.utils import discover_wattbox_devices

# Scan for WattBox devices on your network
devices = discover_wattbox_devices(subnet="192.168.1")
print(f"Found WattBox devices at: {devices}")
```

## Advanced Usage

### Custom Authentication and Timeouts

```python
client = WattBoxClient(
    host="192.168.1.100",
    port=23,                    # 23 for Telnet, 22 for SSH
    username="admin",
    password="your_password", 
    timeout=15.0,              # Command timeout in seconds
    connection_type="telnet"   # "telnet" or "ssh"
)
```

### Bulk Operations

```python
with WattBoxClient(host="192.168.1.100") as client:
    # Get all outlet information with power data
    outlets = client.get_all_outlets_info(include_power_data=True)
    
    # Turn on all outlets
    for outlet in outlets:
        client.turn_on_outlet(outlet.index)
    
    # Reset all outlets with delay
    client.reset_all_outlets(delay=10)
```

### Auto Reboot Configuration

```python
with WattBoxClient(host="192.168.1.100") as client:
    # Enable auto reboot
    client.set_auto_reboot(True)
    
    # Check auto reboot status
    enabled = client.get_auto_reboot_status()
    print(f"Auto reboot enabled: {enabled}")
```

## API Reference

### WattBoxClient

The main client class for communicating with WattBox devices.

#### Constructor

```python
WattBoxClient(host, port=23, username="wattbox", password="wattbox", timeout=10.0, connection_type="telnet")
```

#### Device Information Methods

- `get_device_info(refresh=False, include_outlet_power=True)` - Get complete device state
- `get_system_info()` - Get system information (model, firmware, etc.)
- `get_model()` - Get device model
- `get_firmware_version()` - Get firmware version

#### Outlet Control Methods

- `get_outlet_count()` - Get number of outlets
- `get_outlet_status()` - Get status of all outlets
- `get_outlet_names()` - Get names of all outlets
- `get_all_outlets_info(include_power_data=False)` - Get complete outlet information
- `set_outlet(outlet, action, delay=None)` - Control outlet (action: "ON", "OFF", "TOGGLE", "RESET")
- `turn_on_outlet(outlet)` - Turn on specific outlet
- `turn_off_outlet(outlet)` - Turn off specific outlet
- `toggle_outlet(outlet)` - Toggle specific outlet
- `reset_outlet(outlet, delay=None)` - Reset specific outlet
- `reset_all_outlets(delay=None)` - Reset all outlets

#### Power Monitoring Methods

- `get_power_status()` - Get system-wide power status
- `get_outlet_power_status(outlet)` - Get power status for specific outlet
- `get_all_outlets_power_data()` - Get power data for all outlets

#### UPS Methods

- `get_ups_connection_status()` - Check if UPS is connected
- `get_ups_status()` - Get UPS status information

#### Auto Reboot Methods

- `get_auto_reboot_status()` - Get auto reboot status
- `set_auto_reboot(enabled)` - Enable/disable auto reboot

#### Utility Methods

- `connect()` - Establish connection
- `disconnect()` - Close connection
- `is_connected()` - Check connection status
- `ping()` - Test device connectivity
- `reboot_device()` - Reboot the WattBox device

### Data Models

#### WattBoxDevice
Complete device state containing system info, outlets, power status, and UPS status.

#### OutletInfo
Information about a specific outlet including name, status, and power data.

#### PowerStatus
System-wide power status with current, power, voltage, and safety status.

#### UPSStatus
UPS status including battery charge, health, runtime, and alarm status.

#### SystemInfo
Device system information including firmware, hostname, service tag, and model.

### Exceptions

- `WattBoxError` - Base exception for all WattBox errors
- `WattBoxConnectionError` - Connection-related errors
- `WattBoxAuthenticationError` - Authentication failures
- `WattBoxCommandError` - Command execution errors
- `WattBoxTimeoutError` - Timeout errors
- `WattBoxResponseError` - Unexpected response errors

## Protocol Support

This library implements the SnapAV WattBox Integration Protocol v2.4, supporting:

- **Authentication**: Username/password login
- **Command Types**: Query (?), Control (!), Error (#), Unsolicited (~)
- **Connection Types**: Telnet (port 23), SSH (port 22, firmware 1.3.0.4+)
- **Response Parsing**: Automatic parsing of all response formats

## Error Handling

The library includes comprehensive error handling:

```python
from pywattbox_800 import WattBoxClient, WattBoxConnectionError, WattBoxAuthenticationError

try:
    with WattBoxClient(host="192.168.1.100") as client:
        client.turn_on_outlet(1)
except WattBoxConnectionError:
    print("Failed to connect to device")
except WattBoxAuthenticationError:
    print("Invalid credentials")
except Exception as e:
    print(f"Unexpected error: {e}")
```

## Logging

Enable debug logging to see detailed communication:

```python
import logging
logging.basicConfig(level=logging.DEBUG)
```

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Acknowledgments

- SnapAV for the WattBox Integration Protocol v2.4 specification
- The WattBox community for testing and feedback

## Support

For issues and questions:
1. Check the [API documentation](wattbox_api_v2.4.md) for protocol details
2. Review the examples in this README
3. Open an issue on GitHub with detailed information about your setup and the problem

## Changelog

### v2.4.0
- Initial implementation of Integration Protocol v2.4
- Support for all device information, outlet control, and monitoring features
- Comprehensive error handling and logging
- Context manager support for clean resource management
- Network discovery utilities
