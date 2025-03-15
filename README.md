# Raspberry Pi CamillaDSP with Multichannel I2S

This repository contains Ansible configuration and setup for running CamillaDSP on a Raspberry Pi with multichannel I2S input using an HDMI extractor, as described in [this AudioScienceReview forum thread](https://www.audiosciencereview.com/forum/index.php?threads/multichannel-audio-on-a-pi-will-get-a-whole-lot-easier-and-cheaper.48233/).

## Project Overview

This project provides an automated way to configure a Raspberry Pi for high-quality audio processing using CamillaDSP. It handles:

- Setting up I2S audio device tree overlays for multichannel audio via HDMI extractor
- Installing and configuring CamillaDSP and its web GUI
- Testing audio input via recording
- Testing audio output via speaker tests
- ~~Configuring volume control~~ (not currently supported)
- ~~Optional read-only filesystem for stability~~ (not currently supported)

## Project Structure

The project is organized into Ansible roles, each handling a specific aspect of the setup:

### Installation and Configuration Roles

- **overlay**: Configures device tree overlays for I2S audio
- **camilladsp**: Installs and configures CamillaDSP and its services
- **config**: Manages CamillaDSP configuration files
- **volume**: ~~Configures volume control~~ (not currently supported)
- **readonly**: ~~Sets up read-only filesystem~~ (not currently supported)

### Testing Roles

- **test_overlay**: Tests audio input via recording
- **test_speakers**: Tests audio output via speaker tests

### Global Configuration

All configuration variables are centralized in the `vars.yml` file at the project root for easier management.

## How to Use

### Prerequisites

- Raspberry Pi (tested on Raspberry Pi 5)
- Raspberry Pi OS (Debian-based)
- SSH access to the Raspberry Pi
- Ansible installed on your local machine

### Basic Usage

1. Clone this repository:
   ```
   git clone https://github.com/michsos/raspi-camilladsp.git
   cd raspi-camilladsp
   ```

2. Update the inventory file with your Raspberry Pi's information:
   ```
   # inventory
   [raspi]
   raspi.local ansible_user=yourusername
   ```

3. Run the main playbook:
   ```
   ansible-playbook site.yml
   ```

### Step-by-Step Usage

For a complete setup, follow these steps in order:

1. **Set up the audio hardware** (device tree overlays):
   ```
   ansible-playbook site.yml --tags overlay
   ```

2. **Test the audio input** via recording:
   ```
   ansible-playbook site.yml --tags test_overlay
   ```
   
   The recordings will be saved to the `./recordings/` directory on your local machine.
   
   You can adjust the recording duration:
   ```
   ansible-playbook site.yml --tags test_overlay --extra-vars "recording_duration=30"
   ```

3. **Test the speakers**:
   ```
   ansible-playbook site.yml --tags test_speakers
   ```

4. **Install and configure CamillaDSP**:
   ```
   ansible-playbook site.yml --tags camilladsp
   ```

5. **Copy configuration files** (optional):
   ```
   ansible-playbook site.yml --tags config --extra-vars "config_source_dir=/path/to/your/configs"
   ```
   
   This will copy configuration files from your local machine to the Raspberry Pi:
   - Files from `/path/to/your/configs/configs/*.yml` will be copied to CamillaDSP's configs directory
   - Files from `/path/to/your/configs/coeffs/*.yml` and `/path/to/your/configs/coeffs/*.wav` will be copied to CamillaDSP's coeffs directory
   
   Note: If either the `configs` or `coeffs` subdirectory doesn't exist in your source directory, it will be skipped.
   
   The config role will only run if you specify a `config_source_dir`.

6. **Configure volume control** (optional):
   ```
   ansible-playbook site.yml --tags volume
   ```

7. **Set up read-only filesystem** (optional):
   ```
   ansible-playbook site.yml --tags readonly
   ```

### Quick Commands

- **Run the complete setup** in one command:
  ```
  ansible-playbook site.yml
  ```

- **Run the complete setup without audio recording**:
  ```
  ansible-playbook site.yml --extra-vars "record_audio_channels=false"
  ```

- **Run the complete setup with config copy**:
  ```
  ansible-playbook site.yml --extra-vars "config_source_dir=/path/to/your/configs"
  ```

- **Run with a specific CamillaDSP user** (different from the Ansible user):
  ```
  ansible-playbook site.yml --extra-vars "camilladsp_user_override=otheruser camilladsp_group_override=othergroup"
  ```

## Configuration Options

You can customize the setup by modifying the variables in the `vars.yml` file at the project root. This file contains settings for:

- User and system settings (user, group, installation directory - defaults to ~/camilladsp)
- Hardware settings (overlay directory, config file path)
- Testing settings (recording directory, durations, volume levels)
- Config settings (source and destination directories)

### User Configuration

By default, CamillaDSP will be installed and run as the same user that Ansible uses to connect to the Raspberry Pi. This simplifies setup in most cases. If you need to use a different user, you can override this with command line variables:

```
ansible-playbook site.yml --extra-vars "camilladsp_user_override=otheruser camilladsp_group_override=othergroup"
```

### CamillaDSP GUI

The CamillaDSP role includes the CamillaDSP GUI, which provides a web interface for configuring and controlling CamillaDSP. The role:

1. Installs CamillaDSP and its GUI
2. Creates systemd service files for both CamillaDSP and CamillaGUI
3. Enables and starts both services automatically
4. Displays service status and logs

CamillaDSP runs with the `-p1234 -w -s` parameters (websocket server on port 1234 with statefile) and the GUI runs on port 5005 (fixed port).

You can access the GUI at:
- http://localhost:5005 (if running on your local machine)
- http://your-raspberry-pi-ip:5005 (if running on a Raspberry Pi)

The services are configured to start automatically on boot and restart if they crash. You can manage them using standard systemd commands:

```
# Check service status
sudo systemctl status camilladsp
sudo systemctl status camillagui

# Restart services
sudo systemctl restart camilladsp
sudo systemctl restart camillagui

# Stop services
sudo systemctl stop camilladsp
sudo systemctl stop camillagui

# View service logs
sudo journalctl -u camilladsp
sudo journalctl -u camillagui
```

## TODOs/Roadmap

- [x] Complete CamillaDSP installation role
- [ ] Add configuration templates for different audio setups
- [ ] ~~Implement volume control integration~~ (not currently supported)
- [ ] ~~Add read-only filesystem support~~ (not currently supported)
- [ ] Create backup/restore functionality
- [x] Add web interface configuration
- [ ] Implement automatic updates

## Troubleshooting

### Audio Device Not Recognized

If the audio device is not recognized after running the overlay role:

1. Check the device tree overlays in `/boot/firmware/config.txt`
2. Verify that both overlays are properly loaded with `dtoverlay -l`
3. Check the kernel logs with `dmesg | grep -i audio`

### Recording Issues

If audio recording fails:

1. Ensure the audio device is properly recognized
2. Verify that audio is being streamed to the I2S inputs
3. Check ALSA configuration with `arecord -l`
4. Try manually recording with `arecord -D hw:GenericStereoAu,0 -c 8 -f S24_LE -r 48000 test.wav`

### CamillaDSP Issues

If you encounter issues with CamillaDSP installation or the GUI:

1. Check if Python and pip are installed correctly: `python3 --version && pip3 --version`
2. Verify the installation directory exists and has correct permissions: `ls -la ~/camilladsp`
   (Note: The default installation directory is ~/camilladsp, configured in vars.yml)
3. Check if the virtual environment was created: `ls -la ~/camilladsp/camillagui_venv`
4. Check the systemd service status: `sudo systemctl status camilladsp && sudo systemctl status camillagui`
5. View the service logs: `sudo journalctl -u camilladsp -n 50 && sudo journalctl -u camillagui -n 50`
6. Try restarting the services: `sudo systemctl restart camilladsp && sudo systemctl restart camillagui`
7. Verify the service files are correctly configured: `cat /etc/systemd/system/camilladsp.service /etc/systemd/system/camillagui.service`
8. Try running CamillaDSP manually: `~/camilladsp/bin/camilladsp -p1234 -w -s ~/camilladsp/statefile.yml`
9. Try running the GUI manually: `cd ~/camilladsp && camillagui_venv/bin/python gui/main.py`
10. Check for any error messages in the Ansible output

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the Creative Commons Attribution-NonCommercial 4.0 International License (CC BY-NC 4.0) - see the LICENSE file for details. This license specifically prohibits commercial use while allowing sharing and adaptation with attribution.