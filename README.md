# Purpose

The goal of this repository is to get the [BGMv2 demonstration repo](andreyryabtsev/BGMv2-webcam-plugin-linux) working on Nvidia Ampere based cards(RTX 30-series and A100). This was tested on a RTX 3060Ti with Pop_OS! 20.10 with Nvidia 460.67 drivers (CUDA version 11.2 and cuDNN 8).

# Prerequisites
This plugin requires Linux, because it relies on the [v4l2loopback kernel module](https://github.com/umlaeute/v4l2loopback) to create and stream to virtual video devices. We welcome and encourage community adaptations to other platforms.

1. Install v4l2loopback. On Debian/Ubuntu, the command is likely `sudo apt-get install v4l2loopback-dkms` or `sudo apt-get install v4l2loopback-utils`.
2. Make sure that you have the latest Nvidia Linux Drivers installed. (would not hurt to also install CUDA and cuDNN)
3. Clone this repository(`git clone https://github.com/charitarthchugh/BGMv2-webcam-plugin-linux-Ampere`)
4. Install Anaconda/ Miniconda if you have not already. This will **not work** with pip.
5. Install required packages in `enviroment.yml` using `conda env create -f enviroment.yml`.

# Directions

Before running the plugin, the virtual web camera device needs to be created.

`sudo modprobe v4l2loopback devices=1`

The above command should create a single virtual webcam at `/dev/video1` (the number may change), which is the default stream output for the plugin script. This webcam can now be selected by software such as Zoom, browsers, etc.
After downloading the TorchScript weights of your choice [here](https://drive.google.com/drive/u/1/folders/1cbetlrKREitIgjnIikG1HdM4x72FtgBh), launch the pluging with e.g.:
```python demo_webcam.py --model-checkpoint torchscript_resnet50_fp32.pth```

- I was only able to get the `torchscript_resnet50_fp32` model to work. `torchscript_resnet_50_fp16` crashed on me. 

Once the plugin is launched, a simple OpenCV-based UI will show two buttons:

- Toggle between background selection view and (after snapping a background) actual matting
- Cycle through target background options:
  1. Plain white
  2. Gaussian blur of the background frame
  3. Supplied target video background (samples included in this repo)
  4. Supplied target image background
At any point, press Q to exit.

# Troubleshooting
If you get an error like the following:
```bash
python3 demo_webcam.py --model-checkpoint torchscript_mobilenetv2_fp16.pth
Traceback (most recent call last):
  File "/root/demo_webcam.py", line 586, in <module>
    fake_camera = pyfakewebcam.FakeWebcam(args.camera_device, cam.width, cam.height)
  File "/usr/local/lib/python3.9/dist-packages/pyfakewebcam/pyfakewebcam.py", line 54, in __init__
    fcntl.ioctl(self._video_device, _v4l2.VIDIOC_S_FMT, self._settings)
OSError: [Errno 22] Invalid argument
FATAL: exception not rethrown
Aborted
```

1. Do `ls /dev/ | grep video`(see all potential webcam outputs.)
   - Most likely, `/dev/video0` is your actual webcam.
2. Switch between webcam outputs using the `--camera-device` flag.
   1. Ex: `python3 demo_webcam.py --camera-device /dev/video2 --model-checkpoint ...`
3. You do not need to reinstall `pyfakewebcam`!.

# Guidelines
For best results:
1. Avoid blocking bright light, especially during background capture.
2. Avoid changing light conditions. For prolonged use, it may be helpful to re-take the background as light conditions drift.

# Community Contributions
1. [@kevinlester](https://github.com/kevinlester/BGMv2-webcam-plugin-linux) added many realtime settings to the UI controls
2. [@KostiaChorny](https://github.com/KostiaChorny/BGMv2-webcam-plugin-linux) added Windows OBS virtual camera support (not merged but please visit their fork)

Thanks to [CK FreeVideoTemplates](https://www.youtube.com/watch?v=DHRUNWdf3ms) on YouTube for the seasonal video target background.