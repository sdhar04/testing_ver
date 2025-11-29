# Verilog Color Processor - Object Tracking Simulator

A hardware-software co-simulation project that implements color-based object tracking for robotic applications. The system uses Verilog RTL for image processing and motor control, simulated via Verilator, with a graphical interface built using ImGui/SDL2/OpenGL and ROS integration for robot control.

## Overview

This project simulates a vision-based object tracking system typically used in mobile robots. The design processes camera frames to detect objects of a specific color, calculates their position (centroid), estimates proximity, and generates motor control signals to track the object.

### Key Features

- **Real-time color filtering** - Supports red, green, blue, cyan, magenta, yellow, and white color detection
- **Centroid calculation** - Determines the horizontal position of detected objects in the frame
- **Proximity estimation** - Estimates object distance based on pixel count
- **Motor control** - Generates differential drive signals for robot navigation
- **ROS integration** - Publishes velocity commands and subscribes to camera feeds
- **Interactive GUI** - Visualizes input/output frames, centroid position, and proximity levels

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        Design Top                               │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────┐  │
│  │ Frame Buffer│───▶│ Color Proc  │───▶│ Frame Buffer (out)  │  │
│  │   (input)   │    │  (filter)   │    │                     │  │
│  └─────────────┘    └──────┬──────┘    └─────────────────────┘  │
│                            │                                     │
│                            ▼                                     │
│                     ┌─────────────┐    ┌─────────────────────┐  │
│                     │  Centroid   │───▶│   Motor Ctrl SPI    │  │
│                     │ Calculator  │    │                     │  │
│                     └─────────────┘    └─────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

### RTL Modules

| Module | Description |
|--------|-------------|
| `design_top.v` | Top-level module integrating all components |
| `frame_buffer.v` | Dual-port RAM for storing image frames (160x120, 12-bit RGB) |
| `color_proc.v` | Color filtering and histogram generation |
| `centroid.v` | Calculates object centroid and proximity from histogram data |
| `motor_ctrl_spi.v` | Generates differential motor speeds based on centroid and proximity |

### Image Processing Pipeline

1. **Input**: QQVGA resolution (160x120 pixels), 12-bit RGB (4 bits per channel)
2. **Color Filter**: Applies selected color threshold to identify target pixels
3. **Histogram**: Divides inner frame (128x104) into 8 horizontal bins
4. **Centroid**: Determines object position using bin analysis
5. **Proximity**: Estimates distance based on detected pixel count
6. **Motor Control**: Generates left/right motor speeds for tracking

## Dependencies

- **Verilator** - Verilog simulator
- **SDL2** - Window management and input handling
- **OpenGL** - Graphics rendering
- **ImGui** - Immediate mode GUI library
- **OpenCV 4** - Image processing
- **ROS Noetic** - Robot Operating System (optional, for robot integration)

### Installing Dependencies (Ubuntu/Debian)

```bash
# Verilator
sudo apt-get install verilator

# SDL2
sudo apt-get install libsdl2-dev

# OpenCV
sudo apt-get install libopencv-dev

# ROS Noetic (optional)
# Follow instructions at http://wiki.ros.org/noetic/Installation
```

## Building

```bash
# Clone the repository
git clone https://github.com/sdhar04/testing_ver.git
cd testing_ver

# Build the project
make

# The executable will be created at obj_dir/Vdesign_top
```

## Usage

```bash
# Run the simulator
./obj_dir/Vdesign_top
```

### GUI Controls

- **Start/Stop** - Toggle continuous simulation
- **Step frame** - Advance one frame
- **Reset** - Reset the simulation state
- **Color filters** - Select red, green, and/or blue filter components
- **Frames per iteration** - Adjust simulation speed

### Color Filter Options

| Filter Code | Color | Description |
|-------------|-------|-------------|
| 000 | None | No filter, passes all pixels |
| 001 | Blue | Detects blue objects |
| 010 | Green | Detects green objects |
| 100 | Red | Detects red objects |
| 011 | Cyan | Detects cyan (green + blue) |
| 110 | Yellow | Detects yellow (red + green) |
| 101 | Magenta | Detects magenta (red + blue) |
| 111 | White | Detects white objects |

### Centroid Output

The 8-bit centroid output indicates the horizontal position of the detected object:

| Value | Position |
|-------|----------|
| `0001_1000` | Centered |
| `0001_0000` | Slightly left |
| `0010_0000` | Left |
| `0100_0000` | More left |
| `1000_0000` | Leftmost |
| `0000_1000` | Slightly right |
| `0000_0100` | Right |
| `0000_0010` | More right |
| `0000_0001` | Rightmost |

### ROS Integration

The simulator integrates with ROS for robot control:

- **Subscribes to**: `turtlebot2/camera/image_raw` - Camera feed
- **Publishes to**: `turtlebot2/cmd_vel` - Velocity commands (Twist messages)

## Project Structure

```
testing_ver/
├── Makefile              # Build configuration
├── rtl/                  # Verilog RTL source files
│   ├── design_top.v      # Top-level design
│   ├── frame_buffer.v    # Dual-port frame memory
│   ├── color_proc.v      # Color processing module
│   ├── centroid.v        # Centroid calculation
│   ├── motor_ctrl_spi.v  # Motor control
│   └── tb_design_top.vhd # VHDL testbench
└── src/                  # C++ source files and assets
    ├── main.cpp          # Main application with GUI
    ├── MotorDriver.cc    # Motor driver utilities
    └── *.png             # UI assets and test images
```

## Technical Specifications

- **Image Resolution**: 160x120 (QQVGA)
- **Color Depth**: 12-bit (4 bits per R/G/B channel)
- **Inner Frame**: 128x104 pixels (borders excluded for processing)
- **Histogram Bins**: 8 horizontal bins
- **Proximity Levels**: 0-7 (0=far, 7=close)
- **Motor Output**: 16-bit signed values (degrees per second)

## Authors

Original RTL design by:
- **Felipe Machado Sanchez**
- Area de Tecnologia Electronica
- Universidad Rey Juan Carlos
- [https://github.com/felipe-m](https://github.com/felipe-m)

## License

Please refer to the original author's repository for license information.
