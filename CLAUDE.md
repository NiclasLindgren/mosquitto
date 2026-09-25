# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Eclipse Mosquitto is an open source MQTT broker implementing versions 5.0, 3.1.1, and 3.1 of the MQTT protocol. It includes:
- **mosquitto** - MQTT broker
- **libmosquitto** - C client library (with C++ wrapper)
- **mosquitto_pub/sub/rr** - Command-line client tools
- **Plugins** - Authentication, ACL, and message processing plugins

## Building

### Windows (CMake)
```cmd
mkdir build && cd build
cmake ..
cmake --build .
```

### Linux/Unix (Make preferred)
```bash
make                    # Full build with docs
make binary             # Skip building man pages
make WITH_TLS=no        # Build without TLS
make WITH_WEBSOCKETS=yes  # Enable WebSocket support
```

### Key Build Options (config.mk or -DWITH_*)
- `WITH_TLS` - OpenSSL/TLS support (default: yes)
- `WITH_THREADING` - Client library threading (default: yes)
- `WITH_WEBSOCKETS` - WebSocket support (default: no)
- `WITH_CJSON` - JSON support for clients and dynamic security (default: yes)
- `WITH_SRV` - DNS SRV lookup, requires c-ares (default: no)
- `WITH_BRIDGE` - Broker-to-broker bridging (default: yes)
- `WITH_PERSISTENCE` - Message persistence (default: yes)

## Testing

```bash
make test               # Serial test execution (slow)
make ptest              # Parallel tests (up to 20 concurrent)
make -C test utest      # Unit tests only (requires CUnit)
```

Tests require Python 3. Unit tests require CUnit.

## Architecture

### Source Organization
- **src/** - Broker implementation (~18K lines)
  - `mosquitto.c` - Main entry point
  - `loop.c` - Main event loop
  - `handle_*.c` - MQTT packet handlers (connect, publish, subscribe, etc.)
  - `database.c` - Message database and session management
  - `security*.c` - Authentication and ACL
  - `bridge*.c` - Broker-to-broker bridging
  - `mux_epoll.c`/`mux_poll.c` - Platform-specific event multiplexing
  - `persist_*.c` - Database persistence (v2.3.4 and v5 formats)

- **lib/** - Client library (~13K lines)
  - `mosquitto.c` - Library initialization
  - `connect.c`, `loop.c` - Connection and event loop
  - `packet_mosq.c` - MQTT packet encoding/decoding
  - `property_mosq.c` - MQTT v5 properties
  - `net_mosq.c`, `tls_mosq.c` - Network and TLS
  - **cpp/** - C++ wrapper (mosquittopp)

- **client/** - Command-line tools
  - `pub_client.c`, `sub_client.c`, `rr_client.c` - mosquitto_pub/sub/rr
  - `client_shared.c` - Shared client utilities

- **plugins/** - Example and core plugins
  - `dynamic-security/` - Full authentication/ACL system
  - `message-timestamp/`, `payload-modification/` - Example plugins

- **include/** - Public headers
  - `mosquitto.h` - Client library API
  - `mosquitto_broker.h`, `mosquitto_plugin.h` - Plugin interfaces
  - `mqtt_protocol.h` - MQTT protocol constants

### Key Patterns
- Files ending in `_mosq.c` contain code shared between broker and library
- Internal headers use `*_internal.h` or `*_broker_internal.h`
- Conditional compilation via `WITH_*` defines for feature control
- Platform-specific code guarded by `#ifdef WIN32`, `#ifdef __unix__`, etc.

## Code Style
- Indentation: tabs (size 4) for C/C++, spaces (size 4) for Python
- No enforced line length limits (existing code does not break long lines)
- Function naming: lowercase with underscores (`mosquitto_*`)

## Windows Notes
- Use CMake exclusively for Windows builds
- libmosquitto compiled without threading by default (no `mosquitto_loop_start()`)
- Connection limits: ~8192 on Windows 10/Server 2019, ~2048 on older versions
- WebSocket support included in binary releases
