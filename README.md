# Lynx Conveyor Controller

## Overview

The Lynx Conveyor Controller is an automated linear rail conveyor system designed for precise tray handling and positioning in industrial environments. This Arduino-based system controls a pneumatic shuttle that can transport trays between multiple stations with high precision positioning and comprehensive safety monitoring.

## System Purpose

This system manages:
- **Precise linear motion control** with 3 tray loading positions plus home position
- **Pneumatic shuttle system** that can lock/unlock to grab and release trays during transport
- **Individual pneumatic tray locks** at each position (lock1, lock2, lock3) to secure trays
- **Tray detection sensors** throughout the system to track tray locations
- **Pressure monitoring** to ensure pneumatic systems have adequate pressure for operation
- **Manual positioning control** via MPG handwheel encoder interface
- **Comprehensive logging** with circular buffer for error diagnosis and system history
- **Serial and Ethernet command interfaces** with extensive help documentation
- **Automated test sequences** for system validation and troubleshooting

## Hardware Requirements

### 1. Controller & I/O
- **ClearCore Industrial I/O and Motion Controller** (main controller)
- **CCIO-8 Digital I/O Expansion** (8-point expansion for additional sensors/outputs)
- **CABLE-RIBBON6** (6 inch ribbon cable for CCIO connection)

### 2. Motion System
- **NEMA 23 ClearPath-SDSK Model CPM-SDSK-2321S-RLS** (servo motor with integrated drive)
- **Linear rail system** with precision positioning (1050mm total travel)

#### Motor Configuration Requirements
The motor must be configured using **Teknic ClearPath-MSP software** with these exact settings:

| Parameter | Value | Notes |
|-----------|-------|-------|
| Input Resolution | 800 pulses per revolution | Critical for position calculations |
| Input Format | Step and Direction | Standard interface |
| Torque Limit | 50% | Operational safety limit |
| HLFB Output | ASG-POSITION WITH MEASURED TORQUE | Position feedback mode |
| Homing | Enabled | Required for system initialization |
| Homing Mode | Normal | Standard homing operation |
| Homing Style | User seeks home; ClearPath ASG signals when homing is complete | |
| Homing Occurs | Upon every Enable | Ensures position accuracy |
| Homing Direction | CCW (Counter-clockwise) | Physical system requirement |
| Homing Torque Limit | 40% | Safety limit during homing |
| Homing Speed (RPM) | 40.00 | Slow approach for accuracy |
| Homing Accel/Decel (RPM/s) | 5,000 | Motion profile |
| Precision Homing | Enabled | For maximum repeatability |
| Physical Home Clearance | 200 cnts | Distance from hard stop |
| Home Offset Move Distance | 0 cnts | No offset required |

#### Motor Setup Procedure
1. **Configure settings** using Teknic ClearPath-MSP software with values above
2. **Load configuration** to motor (download settings to motor memory)
3. **Perform auto-tuning** with realistic load conditions:
   - Lock a tray to the shuttle to simulate actual operating load
   - Set auto-tune torque limit to 50%
   - Set auto-tune RPM to 350 RPM (matches operational velocity with tray)
   - Run auto-tune sequence to optimize motor performance
4. **Verify homing operation** and position accuracy after installation
5. **Calibrate precision homing** if home reference moves or mechanics change

### 3. Feedback & Control
- **CL-ENCDR-DFIN Encoder Input Adapter** (for MPG handwheel manual control)
- **MPG handwheel encoder** (manual positioning interface)
- **Tray detection sensors** positioned throughout the conveyor system

### 4. Pneumatic System
- **Pneumatic shuttle mechanism** with lock/unlock capability for tray grabbing
- **Individual pneumatic tray locks** at each of the 3 loading positions
- **Pressure sensor** for pneumatic system monitoring (minimum 21.75 PSI)
- **Solenoid valves** for shuttle and tray lock control
- **Compressed air supply** system

### 5. Power System
- **IPC-5 DC Power Supply** (350/500W, 75VDC output for motor power)
- **POWER4-STRIP DC Bus Distribution Strip** (power distribution)
- **24VDC supply** for logic and pneumatics

### 6. Safety Systems
- **Emergency stop (E-stop)** circuit with normally closed contacts
- **Pressure monitoring** system (minimum 21.75 PSI threshold)
- **Tray detection sensors** for position tracking and safety validation
- **Comprehensive error detection** with historical logging

## System Architecture

The codebase is organized into modular components:

### Core Files
- **`lynx_conveyor.ino`** - Main Arduino sketch with system initialization and main loop
- **`Commands.h/cpp`** - Command parsing and execution system
- **`CommandController.h/cpp`** - Command interface management

### Control Modules
- **`MotorController.h/cpp`** - Servo motor control, positioning, and homing
- **`ValveController.h/cpp`** - Pneumatic valve and sensor management
- **`EncoderController.h/cpp`** - MPG handwheel manual control interface
- **`EthernetController.h/cpp`** - Network communication interface

### Support Systems
- **`OutputManager.h/cpp`** - Unified output handling for serial/ethernet
- **`Logging.h/cpp`** - System state logging and monitoring
- **`LogHistory.h/cpp`** - Historical logging with circular buffer
- **`Tests.h/cpp`** - Automated test sequences
- **`Utils.h/cpp`** - Utility functions and data structures

## Position System

The conveyor operates with precisely calibrated positions:

| Position | Distance from Home | Purpose |
|----------|-------------------|---------|
| Home | 0.0 mm | Reference position |
| Position 1 | 36.57 mm | Tray loading position and tray 1 station |
| Position 2 | 477.79 mm | Tray 2 station |
| Position 3 | 919.75 mm | Tray 3 station |
| Position 4 | 1050.0 mm | Maximum travel position |

**Key Calibration Value**: `MM_PER_REV = 53.98` mm per motor revolution
- This critical value must be recalibrated if mechanical components change
- Calibration procedure is documented in `MotorController.h`

## Communication Interfaces

### Serial Interface (USB)
- **Baud Rate**: 115200
- **Purpose**: Direct command interface and diagnostics
- **Connection**: USB cable to development computer

### Ethernet Interface
- **Port**: 8888 (TCP)
- **Max Clients**: 4 simultaneous connections
- **Purpose**: Remote command interface and monitoring
- **Timeout**: Automatic client timeout for connection management

### Command System
All commands include extensive help documentation:
- Type `help` for available commands
- Type `help,<command>` for specific command usage
- Commands use comma-separated format: `command,parameter1,parameter2`

## Operational Modes

### A. Automated Mode (Default)
- System executes pre-programmed tray handling sequences
- Automatic tray detection, transport, and positioning
- Pneumatic shuttle automatically grabs/releases trays during transport
- Position locks engage/disengage automatically based on sequence
- Full safety monitoring and fault detection active

### B. Manual Mode
- Direct command control for setup, testing, and maintenance
- Manual positioning via MPG handwheel encoder
- Individual control of all pneumatic systems
- Step-by-step operation for troubleshooting

## Quick Start Guide

### Initial Setup
1. **Hardware Setup**
   - Configure ClearPath motor using Teknic ClearPath-MSP software
   - Connect all hardware components per system documentation
   - Ensure pneumatic system pressure ≥ 21.75 PSI

2. **System Initialization**
   ```
   motor,init          # Initialize motor system
   motor,home          # Home the system to establish reference
   system,status       # Verify all sensors and pneumatics
   ```

### Basic Operations

#### Manual Positioning
```bash
move,1              # Move to position 1 (36.57mm)
move,2              # Move to position 2 (477.79mm)
move,3              # Move to position 3 (919.75mm)
move,<mm>           # Move to specific position in mm
```

#### Pneumatic Control
```bash
lock,1              # Lock tray at position 1
lock,2              # Lock tray at position 2
lock,3              # Lock tray at position 3
lock,shuttle        # Lock shuttle to grab tray
unlock,shuttle      # Unlock shuttle to release tray
```

#### Manual Control Mode
```bash
encoder,enable      # Enable handwheel control
encoder,disable     # Return to automated control
```

#### System Monitoring
```bash
system,status       # Current system status
system,pressure     # Check pneumatic pressure
motor,status        # Motor state and position
system,history      # View operation history
```

### Automated Operations
```bash
tray,load          # Execute full loading sequence
tray,unload        # Execute full unloading sequence
```

### Diagnostics and Testing
```bash
test,homing        # Test homing repeatability
test,positioning   # Test position cycling
test,tray          # Test tray handling operations
system,reset       # Clear fault conditions
network,status     # Ethernet connection info
```

## Safety Features

### Emergency Stop (E-Stop)
- Normally closed contacts on DI6
- Checked every 10ms in main loop
- Immediately disables motor and pneumatics
- System requires manual reset after E-stop clearing

### Pressure Monitoring
- Continuous monitoring of pneumatic pressure
- Minimum threshold: 21.75 PSI
- Automatic warnings when pressure drops
- Operations blocked if pressure insufficient

### Tray Detection
- Sensors at each position detect tray presence
- Used for safety validation during operations
- Prevents operations when trays in unexpected positions

### Motion Safety
- Position limits enforced (0-1050mm)
- Velocity scaling based on move distance
- Automatic deceleration near targets
- Timeout monitoring for all operations

## Troubleshooting

### Common Issues

#### Motor Won't Initialize
1. Check motor power (75VDC supply)
2. Verify motor configuration matches requirements
3. Check ClearCore connections
4. Use `motor,status` to check fault conditions

#### Homing Fails
1. Verify homing direction (CCW)
2. Check for mechanical obstructions
3. Ensure motor configuration is loaded
4. Verify home sensor operation

#### Pneumatic Issues
1. Check system pressure ≥ 21.75 PSI
2. Verify CCIO-8 board connections
3. Test individual valves manually
4. Check cylinder sensor feedback

#### Communication Problems
1. Verify baud rate (115200 for serial)
2. Check Ethernet port 8888 availability
3. Test with `help` command
4. Check for command syntax errors

### Diagnostic Commands
```bash
help                # List all available commands
system,status       # Complete system status
system,history      # View recent operations
motor,status        # Motor diagnostic information
network,status      # Network configuration
log,enable          # Enable periodic logging
log,disable         # Disable periodic logging
```

## Development Notes

### Key Constants
- `PULSES_PER_REV`: 800 (motor configuration)
- `MM_PER_REV`: 53.98 (mechanical calibration)
- `MAX_TRAVEL_MM`: 1050.0 (physical limit)
- `POSITION_TOLERANCE_MM`: 1.0 (positioning accuracy)

### Velocity Profiles
- **With Tray**: 325 RPM (loaded shuttle)
- **Without Tray**: 800 RPM (empty shuttle)
- **Short Moves (<30mm)**: 175 RPM
- **Very Short Moves (<10mm)**: 50 RPM

### Pin Assignments
- **Motor**: ConnectorM0
- **E-Stop**: DI6 (normally closed)
- **Encoder**: ClearCore encoder input
- **Pneumatics**: CCIO-8 board (IO-0 through IO-7)
- **Sensors**: Various ClearCore digital inputs

## Configuration Files

- **`CPM-SDSK_2311S-RLN-2-0_motor setting_hard_homing.mtr`**: Motor configuration file for ClearPath-MSP software

## Maintenance

### Regular Checks
1. **Pneumatic pressure** - maintain ≥ 21.75 PSI
2. **Position accuracy** - verify with dial indicator
3. **Sensor operation** - test all tray detection sensors
4. **E-stop function** - verify emergency stop operation

### Calibration
- **Position calibration** required after mechanical changes
- **Motor auto-tuning** recommended after significant load changes
- **Pressure sensor calibration** if pneumatic readings drift

### Software Updates
- Configuration stored in motor memory
- System settings in source code constants
- Update both motor and Arduino code for major changes

## Author Information

- **Created**: April 14, 2025
- **Author**: rlucien
- **Project**: Lynx Conveyor Controller
- **Repository**: rud-lucien (GitHub)
- **Branch**: bulk_dispense_rust

---

For additional support or questions about this system, refer to the extensive help system built into the controller by typing `help` at the command prompt.
