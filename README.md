# Raspberry-Pi-Camera-CSI-on-Ubuntu
A complete guide to using Raspberry Pi CSI cameras on Ubuntu 22.04. Learn how to detect, capture images, and record videos using V4L2 and FFmpeg. Includes tutorials for standalone usage (no ROS), ROS2 integration, and GUI interfaces (Tkinter/Qt). Also covers CSI camera types, specifications, and troubleshooting.

---

```markdown
## 📹 Record a Video from CSI Camera (Ubuntu 22.04)

### Step 1: Record the Video (on Raspberry Pi)

Run this command to record a **5-second video** at **640x480** resolution:

```bash
ffmpeg -f v4l2 -i /dev/video0 -t 5 -video_size 640x480 video.mp4
```

**Explanation of the command:**

| Parameter | What it does |
|-----------|---------------|
| `-f v4l2` | Use V4L2 camera interface |
| `-i /dev/video0` | Use camera device `/dev/video0` |
| `-t 5` | Record for 5 seconds |
| `-video_size 640x480` | Set resolution to 640x480 |
| `video.mp4` | Save as `video.mp4` file |

**If the video is choppy or slow**, add `-framerate 30` to force 30 frames per second:

```bash
ffmpeg -f v4l2 -framerate 30 -i /dev/video0 -t 5 -video_size 640x480 video.mp4
```

---

### Step 2: Transfer to Laptop (on your laptop)

Open a **new terminal** on your laptop and run:

```bash
scp kamel@10.42.0.109:~/video.mp4 ~/Downloads/
```

**Explanation:**

| Part | What it does |
|------|---------------|
| `scp` | Secure copy (file transfer over SSH) |
| `kamel@10.42.0.109` | Your Pi's username and IP address |
| `:~/video.mp4` | The file on the Pi (home directory) |
| `~/Downloads/` | Destination folder on your laptop |

After this command, the video will be in your laptop's **Downloads** folder.

---

### Step 3: Play the Video (on your laptop)

Run this command to open the video with Ubuntu's default player:

```bash
xdg-open ~/Downloads/video.mp4
```

Or simply navigate to your **Downloads** folder and double-click `video.mp4`.

---

## 📋 Quick Summary

| Step | Where to run | Command |
|------|--------------|---------|
| 1. Record | On Raspberry Pi | `ffmpeg -f v4l2 -i /dev/video0 -t 5 -video_size 640x480 video.mp4` |
| 2. Transfer | On your laptop | `scp kamel@10.42.0.109:~/video.mp4 ~/Downloads/` |
| 3. Play | On your laptop | `xdg-open ~/Downloads/video.mp4` |

---

## ⚠️ Troubleshooting

| Problem | Solution |
|---------|----------|
| `ffmpeg: command not found` | Install with: `sudo apt install ffmpeg` |
| `/dev/video0: No such file` | Camera not detected. Check ribbon cable. Run: `v4l2-ctl --list-devices` |
| Video is choppy/slow | Add `-framerate 30` to the record command |
| `scp: Connection refused` | SSH not enabled. Run on Pi: `sudo systemctl enable ssh` |
```

---

## ✅ Summary

This gives you:

| Section | Content |
|---------|---------|
| Step 1 | Record video on Pi (with explanation) |
| Step 2 | Transfer to laptop using `scp` |
| Step 3 | Play video on laptop |
| Quick Summary | All 3 commands in one table |
| Troubleshooting | Common issues and fixes |
