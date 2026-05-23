# Aircraft tracking feeders

Run ADS-B aircraft tracking feeders (FlightAware, FlightRadar24) with containerized services.

This project provides Docker/Podman container definitions for building and running aircraft tracking applications, including dump1090 (ADS-B decoder), piaware (FlightAware client), and fr24feed (FlightRadar24 client). All services are built from official source repositories with multi-platform support (x86_64, ARM64).

## Quick start

### Prerequisites

- Docker and Docker Compose, or Podman with Podman Compose
- A USB RTL-SDR dongle or equivalent ADS-B receiver
- Valid credentials for FlightAware and/or FlightRadar24 (optional)

### Using Docker/Podman Compose

1. Clone the repository:
   ```bash
   git clone https://github.com/ranisalt/feeders.git
   cd feeders
   ```

2. Configure your feeders:
   - `piaware.conf` — FlightAware feeder configuration, which must contain:

   ```
   receiver-type beast
   ```

   - `fr24feed.ini` — FlightRadar24 feeder configuration. If you want to use together with piaware and the dump1090 container, then it must contain:

   ```ini
   receiver="beast-tcp"
   ```

3. Start the services:
   ```bash
   docker-compose up -d
   ```

4. View logs:
   ```bash
   docker-compose logs -f
   ```

### Using podman quadlets

The repository includes systemd unit files for Podman in the `systemd/` directory. These are useful for:
- Running containers without Docker Compose
- Auto-starting services on boot
- Managing services via systemctl

To use systemd containers:

1. Copy unit files to the appropriate systemd user directory:
   ```bash
   mkdir -p ~/.config/containers/systemd/
   cp systemd/*.{container,network} ~/.config/containers/systemd/
   ```

2. Place config files in the expected locations:
   ```bash
   mkdir -p ~/.config/fr24feed ~/.config/piaware
   cp /path/to/fr24feed.ini ~/.config/fr24feed/
   cp /path/to/piaware.conf ~/.config/piaware/
   ```

3. Enable and start the services:
   ```bash
   systemctl --user daemon-reload
   systemctl --user start dump1090.container
   systemctl --user start piaware.container
   systemctl --user start flightradar24.container
   ```

## Services

### dump1090

Decodes ADS-B Mode S signals from RTL-SDR hardware and provides data to other feeders.

**Image**: `ghcr.io/ranisalt/dump1090:latest`  
**Source**: [FlightAware dump1090](https://github.com/flightaware/dump1090)

- Requires USB device access (`/dev/bus/usb`)
- Outputs to network port for consumption by other services
- No configuration file needed — runs standalone

### piaware

FlightAware's feeder application. Sends aircraft data to FlightAware and provides a local web UI.

**Image**: `ghcr.io/ranisalt/piaware:latest`  
**Source**: [FlightAware piaware_builder](https://github.com/flightaware/piaware_builder)

**Configuration**:
- Mount `piaware.conf` to `/etc/piaware.conf`
- Required: Set a `feeder-id` to identify your station
- Optional: Configure `mlat` client for multilateration

**Web UI**: Available on port 8080 (disabled by default)

### fr24feed

FlightRadar24's feeder application. Sends aircraft data to FlightRadar24 and provides a local web UI.

**Image**: `ghcr.io/ranisalt/fr24feed:latest`  
**Source**: FlightRadar24 official repositories

**Configuration**:
- Mount `fr24feed.ini` to `/etc/fr24feed.ini`
- Required: Set a valid `receiver-key` from FlightRadar24
- Optional: Configure `mlat-client` for multilateration

**Web UI**: Available on port 8754 (disabled by default)

## Configuration

### piaware.conf

Example configuration:

```
# Set a unique feeder ID
feeder-id xxxxxxxxxxxxxxxx

# Enable MLAT (multilateration)
allow-mlat yes
use-gpsd no
```

See [piaware documentation](https://github.com/flightaware/piaware) for all options.

### fr24feed.ini

Example configuration:

```ini
fr24key="xxxxxxxxxxxxxxxx"
mlat="yes"
mlat-without-gps="yes"
```

See FlightRadar24 documentation for all options.

## Architecture

```
dump1090 (ADS-B decoder)
    ↓
    ├→ piaware (FlightAware feeder)
    └→ fr24feed (FlightRadar24 feeder)
```

All services communicate locally. dump1090 must start first and run continuously for other feeders to receive data.

## Building images

Images are automatically built and published to GitHub Container Registry via GitHub Actions. To build locally:

```bash
# Build all images
podman build -t dump1090 dump1090/
podman build -t piaware piaware/
podman build -t fr24feed fr24feed/

# Or with Docker
docker build -t dump1090 dump1090/
docker build -t piaware piaware/
docker build -t fr24feed fr24feed/
```

## Troubleshooting

### Services won't start
- Ensure your RTL-SDR device is connected and accessible
- Check USB permissions: `ls -la /dev/bus/usb/`
- View logs: `docker-compose logs <service-name>`

### Feeder not connecting
- Verify your feeder credentials (piaware feeder-id, fr24feed receiver-key)
- Check network connectivity: `docker-compose exec <service> ping 8.8.8.8`
- Review service logs for authentication errors

## Resources

- [FlightAware piaware](https://github.com/flightaware/piaware)
- [FlightAware dump1090](https://github.com/flightaware/dump1090)

## License

See LICENSE file for details.
