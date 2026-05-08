# LeRobot SO101 Setup Guide for Raspberry Pi

> Complete Raspberry Pi setup guide for LeRobot + SO100/SO101 leader/follower arms with Feetech motors.

This document is designed for:

- Raspberry Pi OS Bookworm/Trixie
- Raspberry Pi 4 and Raspberry Pi 5
- SO100 / SO101 leader-follower robots
- LeRobot teleoperation and data collection workflows
- Feetech servo motor setup and calibration

Official references:

- https://wiki.seeedstudio.com/lerobot_so100m_new/
- https://huggingface.co/docs/lerobot/en/so101
- https://github.com/huggingface/lerobot

This guide documents a stable setup process for running LeRobot with SO101/Feetech hardware on a Raspberry Pi.

---

# 1. Recommended Environment

## Recommended OS

- Raspberry Pi OS Bookworm (64-bit)
- Raspberry Pi 4 or Raspberry Pi 5
- Python 3.11 recommended

Avoid using Python 3.13 for robotics projects unless you specifically need it. Many robotics and ML packages still have compatibility issues with Python 3.13.

---

# 2. System Update

Note for Raspberry Pi OS Trixie:

- `libatlas-base-dev` is no longer available.
- `libopenblas-dev` should be used instead.
- Python 3.13 is the default system Python.


Update the Raspberry Pi first:

```bash
sudo apt update
sudo apt upgrade -y
```

Install required system packages:

```bash
sudo apt install -y \
    git \
    python3-full \
    python3-venv \
    python3-pip \
    build-essential \
    cmake \
    libopenblas-dev \
    libjpeg-dev \
    libpng-dev \
    ffmpeg
```

---

# 3. Install Miniconda (Optional but Recommended)

Download Miniconda for ARM64:

```bash
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-aarch64.sh
```

Install:

```bash
bash Miniconda3-latest-Linux-aarch64.sh
```

Restart terminal or reload shell:

```bash
source ~/.bashrc
```

---

# 4. Create a Clean Python Environment

Install ffmpeg first (required for video pipelines and datasets).

Important:

`ffmpeg` is a system package installed with `apt`.
You only install it once globally.
It does NOT need to be reinstalled inside virtual environments or conda environments.

Install ffmpeg:

```bash
sudo apt install -y ffmpeg
```


Newer versions of Conda require Terms of Service acceptance before creating environments.

Run:

```bash
conda tos accept --override-channels --channel https://repo.anaconda.com/pkgs/main
```

```bash
conda tos accept --override-channels --channel https://repo.anaconda.com/pkgs/r
```

After accepting the Terms of Service, you must rerun the environment creation command.

The environment does not exist until `conda create` completes successfully.

Then create a Python 3.11 environment:

```bash
conda create -n lerobot python=3.11 -y
```

Activate it:

```bash
conda activate lerobot
```

Upgrade pip:

```bash
pip install --upgrade pip
```

Install compatible setuptools version:

```bash
pip install setuptools==80.9.0
```

---

# 5. Clone LeRobot

Clone repository:

```bash
git clone https://github.com/huggingface/lerobot.git
```

Enter directory:

```bash
cd lerobot
```

---

# 6. Install LeRobot with Feetech Support

Install editable package:

```bash
pip install -e ".[feetech]"
```

---

# 7. Fix Common NumPy Issues

If NumPy or PyTorch errors occur, install compatible NumPy manually:

```bash
pip install numpy==2.2.6
```

Verify:

```bash
python3 -c "import numpy; print(numpy.__version__)"
```

Expected output:

```text
2.2.6
```

---

# 8. Verify PyTorch

Test PyTorch:

```bash
python3 -c "import torch; print(torch.__version__)"
```

Example output:

```text
2.7.1+cpu
```

---

# 9. Verify LeRobot

Test LeRobot import:

```bash
python3 -c "import lerobot; print('LeRobot OK')"
```

Test Feetech support:

```bash
lerobot-find-port
```

---

# 10. Find and Verify USB Serial Ports

LeRobot provides a helper command to identify ports:

```bash
lerobot-find-port
```

You can also manually list serial devices:


List USB serial devices:

```bash
ls /dev/ttyACM*
```

Typical output:

```text
/dev/ttyACM0
```

If permissions errors occur:

```bash
sudo chmod 666 /dev/ttyACM0
```

For multiple devices:

```bash
sudo chmod 666 /dev/ttyACM*
```

Useful debugging command:

```bash
dmesg | tail
```

Typical output:

```text
/dev/ttyACM0
```

If device is missing:

- reconnect USB cable
- verify power
- check dmesg logs

Useful command:

```bash
dmesg | tail
```

---

# 11. Motor ID Setup (Critical Step)

Each motor on the bus requires a unique ID.

Factory-default motors often all ship with ID `1`, so configuration is mandatory before calibration.

Important Rules:

- Connect ONLY ONE motor at a time during setup.
- Do not daisy-chain motors during ID assignment.
- Use stable external motor power.
- USB alone does NOT power the motors.
- Verify correct motor voltage before powering on.
- Incorrect voltage can permanently damage motors.

Recommended ID Mapping:

## Leader Arm

| Joint | ID |
|---|---|
| shoulder_pan | 1 |
| shoulder_lift | 2 |
| elbow_flex | 3 |
| wrist_flex | 4 |
| wrist_roll | 5 |
| gripper | 6 |

## Follower Arm

| Joint | ID |
|---|---|
| shoulder_pan | 1 |
| shoulder_lift | 2 |
| elbow_flex | 3 |
| wrist_flex | 4 |
| wrist_roll | 5 |
| gripper | 6 |

Before setup, disconnect and reconnect USB devices if ports are not detected correctly.

Find ports:

```bash
lerobot-find-port
```

Fix permissions if needed:

```bash
sudo chmod 666 /dev/ttyACM0
sudo chmod 666 /dev/ttyACM1
```

Configure follower motors:

```bash
lerobot-setup-motors \
  --robot.type=so101_follower \
  --robot.port=/dev/ttyACM0
```

Configure leader motors:

```bash
lerobot-setup-motors \
  --teleop.type=so101_leader \
  --teleop.port=/dev/ttyACM1
```

The setup script will:

- assign motor IDs
- configure baudrate
- validate communication
- verify detected motors

If setup fails:

- reconnect USB
- power-cycle motors
- verify permissions
- rerun `lerobot-find-port`
- verify only one motor is connected during assignment

---

# 12. Setup Motors Before Calibration

Before calibration, ensure motors are detected and configured.

Recommended checks:

```bash
lerobot-find-port
```

Verify all motors are powered correctly.

Important:

- Use stable external motor power.
- Avoid USB hubs during setup.
- Connect directly to Raspberry Pi USB ports.
- Verify all cables are firmly connected.

For SO100/SO101 leader arms:

- Ensure the gripper and gears move freely.
- Follow assembly instructions carefully.
- Incorrect gear installation can cause calibration failure.

---

# 13. Calibration and Teleoperation

After motor setup is complete, calibrate both leader and follower arms.

## Calibrate Leader Arm

The leader arm is the teleoperation input device.

```bash
lerobot-calibrate \
  --teleop.type=so101_leader \
  --teleop.port=/dev/ttyACM0 \
  --teleop.id=leader_arm
```

## Calibrate Follower Arm

The follower arm executes motion commands.

```bash
lerobot-calibrate \
  --robot.type=so101_follower \
  --robot.port=/dev/ttyACM1 \
  --robot.id=follower_arm
```

Important calibration notes:

- Keep the arm steady during calibration.
- Avoid forcing joints.
- Ensure joints rotate smoothly.
- Follow on-screen instructions exactly.
- Remove mechanical obstructions before calibration.
- Verify motor IDs before calibration.
- Verify USB permissions before calibration.
- Do NOT place comments after line continuation backslashes (`\`).

Incorrect:

```bash
--teleop.port=/dev/ttyACM0 \# comment
```

Correct:

```bash
--teleop.port=/dev/ttyACM0 \
```

## Validation

Test teleoperation after calibration:

```bash
lerobot-teleoperate \
  --robot.type=so101_follower \
  --robot.port=/dev/ttyACM0 \
  --robot.id=follower_arm \
  --teleop.type=so101_leader \
  --teleop.port=/dev/ttyACM1 \
  --teleop.id=leader_arm
```

Expected behavior:

- follower arm mirrors leader arm motion
- motion should appear smooth and responsive
- gripper should respond correctly

If calibration or teleoperation fails:

- reconnect USB cable
- power-cycle motors
- rerun calibration
- verify serial permissions
- verify motor IDs
- rerun `lerobot-find-port`
- verify correct USB ports
- verify power supply stability

Important:

- Keep emergency stop access nearby.
- Start with slow movements.
- Ensure workspace is clear.
- Avoid joint collisions.

---

# 16. Data Collection

LeRobot can record demonstrations for training policies.

Recommended before recording:

- stable lighting
- fixed camera positions
- stable robot base
- consistent background
- sufficient SSD storage

## Record Dataset

Example:

```bash
lerobot-record \
  --robot.type=so101_follower \
  --robot.port=/dev/ttyACM0 \
  --robot.id=follower_arm \
  --teleop.type=so101_leader \
  --teleop.port=/dev/ttyACM1 \
  --teleop.id=leader_arm \
  --dataset.repo_id=my_dataset
```

The recording workflow typically:

- records robot states
- records actions
- stores camera frames
- saves synchronized timestamps

Dataset storage location:

```text
~/.cache/huggingface/lerobot/
```

## Recommended Dataset Practices

- Record smooth demonstrations.
- Avoid sudden jerky motion.
- Keep camera framing consistent.
- Record multiple task variations.
- Use clear object placement.
- Verify recordings after each session.

## Camera Notes

If using USB cameras:

- avoid low-quality USB hubs
- prefer powered hubs if necessary
- verify bandwidth limits
- Raspberry Pi 5 recommended for multi-camera recording

Verify OpenCV if camera pipelines fail:

```bash
pip install opencv-python
```

Test:

```bash
python -c "import cv2; print(cv2.__version__)"
```

---

# 17. Dataset and Recording Recommendations

Recommended before recording:

- stable lighting
- fixed camera positions
- stable robot base
- consistent background
- sufficient SSD storage

Best practices:

- Record smooth demonstrations.
- Avoid sudden jerky motion.
- Keep camera framing consistent.
- Record multiple task variations.
- Use clear object placement.
- Verify recordings after each session.

Storage recommendation:

- Prefer SSD storage over microSD cards.
- Raspberry Pi 5 recommended for camera pipelines and inference workloads.

---

# 18. SSH and Networking

Enable SSH:

```bash
sudo raspi-config
```

Then:

- Interface Options
- SSH
- Enable

Find Raspberry Pi IP:

```bash
hostname -I
```

SSH from another machine:

```bash
ssh rpi@<raspberry-pi-ip>
```

---

# 19. Common Problems

## Problem: externally-managed-environment

Error:

```text
error: externally-managed-environment
```

Cause:

Raspberry Pi OS blocks system-wide pip installs.

Fix:

Use a virtual environment or conda environment.

---

## Problem: import torch fails

Example:

```text
AttributeError: module 'numpy' has no attribute 'ndarray'
```

Fix:

Reinstall NumPy:

```bash
pip uninstall -y numpy
pip cache purge
pip install numpy==2.2.6
```

---

## Problem: lerobot requires setuptools<81

Fix:

```bash
pip install setuptools==80.9.0
```

---

# 20. USB Hub and Camera Warnings

Avoid cheap USB hubs.

Recommended:

- direct USB connection
- powered USB hubs if required
- official Raspberry Pi power supply

Camera issues can occur when:

- USB bandwidth is overloaded
- insufficient power is available
- multiple cameras share one hub

Raspberry Pi 5 performs significantly better for:

- multi-camera pipelines
- inference workloads
- dataset recording

---

# 21. Recommended Workflow

Every new terminal:

```bash
conda activate lerobot
cd ~/lerobot
```

Then run robot commands.

---

# 22. Useful Test Commands

Verify Python:

```bash
python3 --version
```

Verify NumPy:

```bash
python3 -c "import numpy; print(numpy.__version__)"
```

Verify PyTorch:

```bash
python3 -c "import torch; print(torch.__version__)"
```

Verify LeRobot:

```bash
python3 -c "import lerobot"
```

---

# 23. Recommended Hardware

## Raspberry Pi

- Raspberry Pi 4 (4GB or 8GB)
- Raspberry Pi 5 preferred

## Storage

- Fast SSD recommended
- High quality microSD minimum 64GB

## Power

- Official Raspberry Pi power supply
- External power for motors

---

# 24. Notes

- Keep robotics projects isolated in their own environment.
- Avoid mixing apt packages, conda packages, and global pip installs.
- Python 3.11 currently provides the best compatibility for LeRobot.
- Raspberry Pi 5 performs significantly better for inference and camera pipelines.

