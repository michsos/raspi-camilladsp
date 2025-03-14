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

4. **Test only the CamillaDSP installation**:
   ```
   ansible-playbook test-camilladsp.yml
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

The CamillaDSP role includes the CamillaDSP GUI, which provides a web interface for configuring and controlling CamillaDSP. When you run the CamillaDSP role, it will:

1. Install CamillaDSP and its GUI
2. Start CamillaDSP with the `-p1234 -w -s` parameters (websocket server on port 1234 with statefile)
3. Start the GUI on port 5005 (fixed port)
4. Pause for you to test the GUI in your browser
5. Attempt to stop both processes after you confirm

You can access the GUI at:
- http://localhost:5005 (if running on your local machine)
- http://raspi.local:5005 (if running on a Raspberry Pi)

**Note:** There is a known issue where the CamillaGUI process may not terminate properly after testing. If you notice the GUI is still accessible after the playbook completes, you may need to manually terminate it using:
```
ssh user@raspi.local "pkill -f 'python.*gui/main.py'"
```

The processes are managed using PIDs rather than screen sessions for more reliable process management.

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
4. Try running CamillaDSP manually: `~/camilladsp/bin/camilladsp -p1234 -w -s ~/camilladsp/statefile.yml`
5. Try running the GUI manually: `cd ~/camilladsp && camillagui_venv/bin/python gui/main.py`
6. Check for any error messages in the Ansible output
7. If the GUI doesn't stop properly after testing, manually terminate it: `pkill -f 'python.*gui/main.py'`
8. If CamillaDSP doesn't stop properly, manually terminate it: `pkill -f 'camilladsp -p1234'`
9. Check running processes: `ps aux | grep -E 'camilladsp|python.*gui/main.py'`

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the MIT License - see the LICENSE file for details.