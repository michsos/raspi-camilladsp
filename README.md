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

4. **Record audio only** (without redoing configuration):
   ```
   ansible-playbook record-channels-only.yml
   ```

5. **Adjust the recording duration**:
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

## TODOs/Roadmap

- [ ] Complete CamillaDSP installation role
- [ ] Add configuration templates for different audio setups
- [ ] Implement volume control integration
- [ ] Add read-only filesystem support
- [ ] Create backup/restore functionality
- [ ] Add web interface configuration
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

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the MIT License - see the LICENSE file for details.