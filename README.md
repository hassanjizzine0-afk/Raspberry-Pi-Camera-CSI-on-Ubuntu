# Raspberry-Pi-Camera-CSI-on-Ubuntu
A complete guide to using Raspberry Pi CSI cameras on Ubuntu 22.04. Learn how to detect, capture images, and record videos using V4L2 and FFmpeg. Includes tutorials for standalone usage (no ROS), ROS2 integration, and GUI interfaces (Tkinter/Qt). Also covers CSI camera types, specifications, and troubleshooting.



---


### 🟢 Option 1: Raspberry Pi OS (Pre‑installed camera support)



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

