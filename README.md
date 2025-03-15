# Raspberry Pi CamillaDSP

This repository contains Ansible configuration and setup for running CamillaDSP on a Raspberry Pi with multi-channel audio support.

## Project Overview

This project provides an automated way to configure a Raspberry Pi for high-quality audio processing using CamillaDSP. It handles:

- Setting up I2S audio device tree overlays
- Installing and configuring CamillaDSP
- Recording audio from I2S inputs
- Setting up system services
- Configuring volume control
- Optional read-only filesystem for stability

## Project Structure

The project is organized into Ansible roles, each handling a specific aspect of the setup:

- **overlay**: Configures device tree overlays for I2S audio
- **camilladsp**: Installs and configures CamillaDSP
- **config**: Manages CamillaDSP configuration files
- **services**: Sets up systemd services
- **volume**: Configures volume control
- **readonly**: Sets up read-only filesystem (optional)

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

### Audio Recording

By default, the setup will record audio from all 8 channels to a single multichannel WAV file (48kHz, 24-bit). You have several options:

1. **Run the full setup with audio recording** (default):
   ```
   ansible-playbook site.yml
   ```

2. **Run the full setup without audio recording**:
   ```
   ansible-playbook site.yml --extra-vars "record_audio_channels=false"
   ```

3. **Test only the overlay configuration**:
   ```
   ansible-playbook test-overlay.yml
   ```

4. **Test only the soundcard**:
   ```
   ansible-playbook test-soundcard.yml
   ```

5. **Record audio only** (without redoing configuration):
   ```
   ansible-playbook record-channels-only.yml
   ```

6. **Adjust the recording duration**:
   ```
   ansible-playbook record-channels-only.yml --extra-vars "recording_duration=30"
   ```

The recordings will be saved to the `./recordings/` directory on your local machine.

## Configuration Options

You can customize the setup by modifying the variables in the role defaults files:

- `roles/overlay/defaults/main.yml`: Device tree overlay settings and recording options
- `roles/camilladsp/defaults/main.yml`: CamillaDSP installation settings
- `roles/config/defaults/main.yml`: CamillaDSP configuration settings
- `roles/services/defaults/main.yml`: Service configuration
- `roles/volume/defaults/main.yml`: Volume control settings
- `roles/readonly/defaults/main.yml`: Read-only filesystem settings

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
- [ ] Implement volume control integration
- [ ] Add read-only filesystem support
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

This project is licensed under the MIT License - see the LICENSE file for details.