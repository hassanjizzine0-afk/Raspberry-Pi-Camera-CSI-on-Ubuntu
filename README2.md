

---

```markdown
# 🎥 Raspberry Pi CSI Camera Interface on Ubuntu 22.04

[![Ubuntu 22.04](https://img.shields.io/badge/Ubuntu-22.04-orange?logo=ubuntu)](https://ubuntu.com/)
[![Raspberry Pi](https://img.shields.io/badge/Raspberry%20Pi-4B-red?logo=raspberrypi)](https://www.raspberrypi.com/)
[![License](https://img.shields.io/badge/License-MIT-green)](LICENSE)

---

## 📌 What is This Repository?

A complete, step‑by‑step guide to interface a **CSI camera module** with a **Raspberry Pi** running **Ubuntu 22.04** – **without ROS**.

You will learn:

- ✅ What CSI cameras are and how they work
- ✅ How to detect and test your camera on Ubuntu 22.04
- ✅ How to capture images and record videos using `v4l2-utils` and `ffmpeg`
- ✅ How to transfer videos from Pi to laptop via SSH/SCP
- ✅ How to build GUI camera viewers with **Tkinter** and **Qt**
- ✅ The difference between Raspberry Pi OS and Ubuntu approaches

---

## 🧰 My Current Setup

| Component | Specification |
|-----------|---------------|
| **Board** | Raspberry Pi 4B (4GB RAM) |
| **Operating System** | Ubuntu 22.04 LTS (Jammy) on SD card |
| **Camera** | CSI camera module (v2 / IMX219) |
| **Connection** | Ethernet cable to laptop (SSH access) |

---

## 📡 Understanding CSI Camera Interface

### What is CSI?

**CSI (Camera Serial Interface)** is a high‑speed hardware interface that connects a camera directly to the Raspberry Pi’s GPU. It is much faster and more efficient than USB webcams.

```
Camera Sensor → CSI Ribbon Cable → Raspberry Pi CSI Port → GPU/V4L2 Driver → Your Application
```

### CSI vs USB Webcam

| Feature | CSI Camera | USB Webcam |
|---------|-----------|------------|
| Speed | Up to 2 Gbps | Up to 480 Mbps |
| CPU Usage | Low (hardware accelerated) | High (software encoding) |
| Latency | < 20 ms | 50‑100 ms |
| Image Quality | Raw sensor data | Compressed |
| Cable Length | Short (15‑50 cm) | Long (1‑5 m) |

### Supported CSI Camera Modules

| Model | Sensor | Resolution | Features | Best For |
|-------|--------|-----------|----------|----------|
| **v1** | OV5647 | 5 MP (2592x1944) | Basic, low cost | Beginner projects |
| **v2** | IMX219 | 8 MP (3280x2464) | Better low light | General purpose |
| **v3** | IMX708 | 12 MP (4608x2592) | HDR, autofocus | Advanced vision |
| **HQ** | IMX477 | 12.3 MP (4056x3040) | Interchangeable lenses | Professional |
| **GS** | IMX296 | 1.6 MP (1456x1088) | Global shutter | Fast motion |

---

## ⚠️ Two Operating Systems: Two Approaches

### 🟢 Option 1: Raspberry Pi OS (Pre‑installed camera support)

If you use **Raspberry Pi OS**, camera libraries are ready out‑of‑the‑box.

| Library | Example | Best For |
|---------|---------|----------|
| `picamera2` (Python) | `picam2.capture_file("test.jpg")` | Modern Python apps |
| `libcamera` (CLI) | `libcamera-still -o test.jpg` | Command line |
| `rpicam` (CLI) | `rpicam-still -o test.jpg` | Newer CLI tools |

**Example (Raspberry Pi OS):**

```python
from picamera2 import Picamera2
import time

picam2 = Picamera2()
picam2.start()
time.sleep(2)
picam2.capture_file("test.jpg")
picam2.stop()
print("✅ Image saved")
```

---

### 🔵 Option 2: Ubuntu 22.04 – This Guide (V4L2 + FFmpeg)

**Ubuntu does NOT include `picamera` or `libcamera` pre‑installed.**  
Instead, the CSI camera appears as a standard **Video4Linux2 (V4L2)** device – `/dev/video0`.

| Why Ubuntu works differently |
|-----------------------------|
| Uses generic V4L2 kernel drivers (not Pi‑specific) |
| `rpicam-apps` / `libcamera` are not in Ubuntu repositories |
| You work with standard Linux camera tools (`v4l2-ctl`, `ffmpeg`) |

#### Tools that work on Ubuntu 22.04

| Tool | Status | Command |
|------|--------|---------|
| `v4l2-ctl` | ✅ Works | `v4l2-ctl --list-devices` |
| `ffmpeg` | ✅ Works | `ffmpeg -f v4l2 -i /dev/video0 test.jpg` |
| OpenCV | ✅ Works | `cv2.VideoCapture(0)` |

---

## 🛠️ Installation (Ubuntu 22.04)

### 1. Update system
```bash
sudo apt update && sudo apt upgrade -y
```

### 2. Install V4L2 utilities and FFmpeg
```bash
sudo apt install v4l-utils ffmpeg -y
```

### 3. Detect camera
```bash
v4l2-ctl --list-devices
```
Expected output:
```
mmal service (platform:fe801000.csi):
    /dev/video0
    /dev/video1
```

### 4. Check supported formats
```bash
v4l2-ctl --list-formats-ext
```
Look for `MJPG` (compressed) or `YUYV` (raw).

### 5. Enable camera in config (if not detected)
```bash
sudo nano /boot/firmware/config.txt
```
Add:
```bash
camera_auto_detect=1
gpu_mem=256
```
Reboot:
```bash
sudo reboot
```

---

## 📹 Record a Video (Step‑by‑Step)

### Step 1: Record on Raspberry Pi

Record a **5‑second** video at **640x480** resolution:

```bash
ffmpeg -f v4l2 -i /dev/video0 -t 5 -video_size 640x480 video.mp4
```

**Explanation:**

| Parameter | What it does |
|-----------|---------------|
| `-f v4l2` | Use V4L2 camera interface |
| `-i /dev/video0` | Camera device |
| `-t 5` | Record 5 seconds |
| `-video_size 640x480` | Resolution |
| `video.mp4` | Output file |

**If the video is choppy** – add `-framerate 30`:

```bash
ffmpeg -f v4l2 -framerate 30 -i /dev/video0 -t 5 -video_size 640x480 video.mp4
```

### Step 2: Transfer to Laptop

On your **laptop** (new terminal):

```bash
scp kamel@10.42.0.109:~/video.mp4 ~/Downloads/
```

| Part | Meaning |
|------|---------|
| `scp` | Secure copy over SSH |
| `kamel@10.42.0.109` | Pi username and IP |
| `:~/video.mp4` | File on Pi |
| `~/Downloads/` | Destination folder on laptop |

### Step 3: Play the Video (on laptop)

```bash
xdg-open ~/Downloads/video.mp4
```
Or double‑click the file in your file manager.

---

## 📸 Capture a Single Image

```bash
ffmpeg -f v4l2 -i /dev/video0 -frames:v 1 image.jpg
```

Transfer to laptop:
```bash
scp kamel@10.42.0.109:~/image.jpg ~/Downloads/
```

---

## 📋 Quick Summary of Commands

| Task | Raspberry Pi | Laptop |
|------|--------------|--------|
| Record 5s video | `ffmpeg -f v4l2 -i /dev/video0 -t 5 -video_size 640x480 video.mp4` | – |
| Transfer video | – | `scp kamel@10.42.0.109:~/video.mp4 ~/Downloads/` |
| Play video | – | `xdg-open ~/Downloads/video.mp4` |
| Capture image | `ffmpeg -f v4l2 -i /dev/video0 -frames:v 1 image.jpg` | – |
| Transfer image | – | `scp kamel@10.42.0.109:~/image.jpg ~/Downloads/` |

---

## 🎨 GUI Camera Viewers

### Tkinter Live Viewer

```python
import cv2
import tkinter as tk
from PIL import Image, ImageTk

class CameraApp:
    def __init__(self, window):
        self.window = window
        self.window.title("CSI Camera - Tkinter")
        self.cap = cv2.VideoCapture(0)
        self.label = tk.Label(window)
        self.label.pack()
        self.update()
        window.protocol("WM_DELETE_WINDOW", self.on_close)

    def update(self):
        ret, frame = self.cap.read()
        if ret:
            frame = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
            img = Image.fromarray(frame)
            imgtk = ImageTk.PhotoImage(img)
            self.label.imgtk = imgtk
            self.label.config(image=imgtk)
        self.window.after(10, self.update)

    def on_close(self):
        self.cap.release()
        self.window.destroy()

root = tk.Tk()
app = CameraApp(root)
root.mainloop()
```

### Qt5 Live Viewer

```python
import sys
import cv2
from PyQt5 import QtCore, QtGui, QtWidgets

class CameraApp(QtWidgets.QMainWindow):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("CSI Camera - Qt")
        self.cap = cv2.VideoCapture(0)
        self.label = QtWidgets.QLabel()
        self.setCentralWidget(self.label)
        self.timer = QtCore.QTimer()
        self.timer.timeout.connect(self.update_frame)
        self.timer.start(30)

    def update_frame(self):
        ret, frame = self.cap.read()
        if ret:
            frame = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
            h, w, ch = frame.shape
            bytes_per_line = ch * w
            qt_img = QtGui.QImage(frame.data, w, h, bytes_per_line, QtGui.QImage.Format_RGB888)
            self.label.setPixmap(QtGui.QPixmap.fromImage(qt_img))

    def closeEvent(self, event):
        self.cap.release()

app = QtWidgets.QApplication(sys.argv)
window = CameraApp()
window.show()
sys.exit(app.exec_())
```

---

## ⚠️ Troubleshooting

| Problem | Solution |
|---------|----------|
| `ffmpeg: command not found` | `sudo apt install ffmpeg` |
| `/dev/video0` not found | Check ribbon cable; enable camera in `config.txt` and reboot |
| `v4l2-ctl: command not found` | `sudo apt install v4l-utils` |
| Video choppy / slow | Add `-framerate 30` or reduce resolution to `320x240` |
| `scp: Connection refused` | Enable SSH: `sudo systemctl enable ssh` |
| Permission denied on `/dev/video0` | `sudo usermod -aG video $USER` and reboot |

---

## 📁 Repository Structure

```
camera-interface-raspberry-pi/
├── README.md
├── scripts/
│   ├── capture_image.py
│   ├── record_video.py
│   ├── gui_tkinter.py
│   └── gui_qt.py
└── docs/
    ├── installation.md
    └── troubleshooting.md
```

---

## 📚 References

- [Raspberry Pi Camera Docs](https://www.raspberrypi.com/documentation/computers/camera.html)
- [V4L2 Documentation](https://www.kernel.org/doc/html/latest/userspace-api/media/v4l/v4l2.html)
- [FFmpeg Documentation](https://ffmpeg.org/documentation.html)

---

## 📄 License

MIT License

## 👤 Author

**Your Name** – [GitHub Profile](https://github.com/yourusername)

---

## ⭐ Show Your Support

If this guide helped you, please give it a ⭐!
```

---

This README is **ROS‑free**, focuses entirely on Ubuntu 22.04 + V4L2, includes the exact video recording steps you wanted, and provides clear differentiation between Raspberry Pi OS and Ubuntu.
