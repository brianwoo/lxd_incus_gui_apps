# How to Run X11 Applications in LXD / Incus Container

## Create a GUI Profile
```bash
wget https://blog.simos.info/wp-content/uploads/2018/06/lxdguiprofile.txt
incus profile create gui
# modify raw.idmap ==> both 1000 1001 in lxdguiprofile.txt
cat lxdguiprofile.txt | incus profile edit gui
incus profile list
```

## Create a Container
```bash
incus launch --profile default --profile gui images:ubuntu/noble gui-apps
# For reassigning to an existing container
# incus profile assign gui-apps default,gui
```

## Setup Non-root User
```bash
incus exec gui-apps -- adduser bwoo
incus exec gui-apps -- usermod -aG sudo bwoo
incus exec gui-apps -- sed -i '1 i\TERM=xterm-256color' /home/bwoo/.bashrc
incus exec gui-apps -- sh -c "echo 'export DISPLAY=:0' >> /home/bwoo/.bashrc"
incus exec gui-apps -- sh -c "echo 'Set disable_coredump false' > /etc/sudo.conf"
```

## Install Apps for Testing
```bash
incus exec gui-apps -- apt update
incus exec gui-apps -- apt install -y x11-apps mesa-utils pulseaudio
```

## Fix Audio (Pulseaudio)
```bash
incus exec gui-apps -- sh -c "echo 'export PULSE_SERVER=unix:/tmp/.pulse-native' | tee --append /root/.profile"
incus exec gui-apps -- sh -c "echo 'export PULSE_SERVER=unix:/tmp/.pulse-native' | tee --append /home/bwoo/.profile"
incus exec gui-apps -- sh -c "echo 'default-server = unix:/tmp/.pulse-native' | tee --append /etc/pulse/client.conf"
incus restart gui-apps
```

## Login
```bash
incus exec gui-apps -- sudo -u bwoo bash
```

## Verify HW Acceleration Is Working (Nvidia)
```bash
bwoo@gui-apps:~$ glxinfo -B
name of display: :0
display: :0  screen: 0
direct rendering: Yes
Memory info (GL_NVX_gpu_memory_info):
    Dedicated video memory: 4096 MB
    Total available memory: 4096 MB
    Currently available dedicated video memory: 3494 MB
OpenGL vendor string: NVIDIA Corporation
OpenGL renderer string: NVIDIA GeForce GTX 1050Ti/PCIe/SSE2
OpenGL core profile version string: 4.6.0 NVIDIA 535.171.04
OpenGL core profile shading language version string: 4.60 NVIDIA
OpenGL core profile context flags: (none)
OpenGL core profile profile mask: core profile

OpenGL version string: 4.6.0 NVIDIA 535.171.04
OpenGL shading language version string: 4.60 NVIDIA
OpenGL context flags: (none)
OpenGL profile mask: (none)

OpenGL ES profile version string: OpenGL ES 3.2 NVIDIA 535.171.04
OpenGL ES profile shading language version string: OpenGL ES GLSL ES 3.20
```

## Verify PulseAudio Is Working
```bash
bwoo@gui-apps:~$ pactl info
Server String: unix:/tmp/.pulse-native
Library Protocol Version: 35
Server Protocol Version: 35
Is Local: yes
Client Index: 566
Tile Size: 65472
User Name: woowoo
Host Name: host
Server Name: pulseaudio
Server Version: 15.99.1
Default Sample Specification: s16le 2ch 44100Hz
Default Channel Map: front-left,front-right
Default Sink: alsa_output.pci-0000_00_1f.3.analog-stereo
Default Source: alsa_input.usb-046d_Webcam.analog-stereo
Cookie: a725:c62a
```

# Credits to Kali Linux and Simos
- https://blog.simos.info/running-x11-software-in-lxd-containers/
- https://www.kali.org/docs/containers/kalilinux-lxc-images/