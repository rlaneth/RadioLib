# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

RadioLib is a universal wireless communication library for embedded devices that supports dozens of radio modules (SX127x, SX126x, CC1101, RF69, etc.) and protocols (LoRa, FSK, RTTY, Morse, APRS, LoRaWAN, etc.). It's primarily an Arduino library but can run in non-Arduino environments.

## Architecture

### Core Structure
- **src/Module.h/cpp**: Base class for all radio modules, handles SPI communication and pin management
- **src/Hal.h/cpp**: Hardware Abstraction Layer, with platform-specific implementations in src/hal/
- **src/RadioLib.h**: Main header that includes all modules and protocols
- **src/TypeDef.h**: Common type definitions and error codes

### Module Organization
- **src/modules/**: Radio module drivers (SX127x, SX126x, CC1101, etc.)
  - Each chip family has its own directory with base class and specific variants
  - Common pattern: base class (e.g., SX127x) with specific chips inheriting (SX1278, SX1276)
- **src/protocols/**: High-level protocol implementations (APRS, LoRaWAN, RTTY, etc.)
- **src/utils/**: Utility functions for CRC, cryptography, FEC

### HAL (Hardware Abstraction Layer)
- **src/hal/Arduino/**: Arduino-specific implementation
- **src/hal/RPi/**: Raspberry Pi implementation  
- **src/hal/RPiPico/**: Raspberry Pi Pico implementation
- Platform detection via preprocessor macros like `RADIOLIB_BUILD_ARDUINO`

## Development Commands

### Building
- **Arduino**: Uses Arduino CLI or IDE, no specific build commands
- **Non-Arduino (CMake)**: 
  ```bash
  mkdir build && cd build
  cmake ..
  make -j4
  ```
- **Unit tests**: `cd extras/test/unit && ./test.sh`

### Code Quality
- **Static analysis**: `cd extras/cppcheck && ./cppcheck.sh`
- **Arduino compilation test**: `extras/test/ci/build_arduino.sh <board> <sketch> <flags>`

### Testing
- Unit tests use Boost.Test framework: `extras/test/unit/test.sh`
- CI builds all examples for supported platforms
- Hardware-in-the-loop tests: `extras/test/SX126x/`

## Code Style Requirements

Follow the style guidelines in CONTRIBUTING.md:
- **Bracket style**: 1TBS (JavaScript style) with opening brace on same line
- **Indentation**: 2 spaces (no tabs)
- **Comments**: Single-line comments start with lowercase letter, one space after `//`
- **Memory**: Support both dynamic allocation and static-only mode via `RADIOLIB_STATIC_ONLY`
- **God Mode**: Protect private members unless `RADIOLIB_GODMODE` is defined
- **No Arduino Strings**: Avoid Arduino String class internally, only in public API

## Key Conventions

### Error Handling
- All functions return int16_t status codes (defined in TypeDef.h)
- Success = RADIOLIB_ERR_NONE (0), errors are negative values
- Use status code decoder: https://radiolib-org.github.io/status_decoder/decode.html

### Platform Compatibility
- Always check platform-specific code with `#if defined(PLATFORM_MACRO)`
- Dynamic memory allocation must have static alternatives
- Use HAL methods instead of direct hardware access

### Module Implementation Pattern
1. Inherit from appropriate base class (PhysicalLayer for radios)
2. Implement required virtual methods
3. Add platform-specific initialization
4. Add Doxygen documentation for all public methods
5. Update keywords.txt for Arduino IDE syntax highlighting

## Examples Structure
- **examples/**: Organized by module/protocol type
- Each example is self-contained with .ino file
- **examples/NonArduino/**: Platform-specific examples for ESP-IDF, Pico, etc.
- **examples/LoRaWAN/**: Complete LoRaWAN implementation examples