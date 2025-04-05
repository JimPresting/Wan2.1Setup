# Wan2.1 Setup on Google Colab

## Requirements
1. First, you need to sign up for a Google Colab Pro Account ($10 per month as of writing) - this provides 100 Runtime hours per month and access to more than enough high-power GPUs.
2. Instead of downloading all models locally every time, I recommend storing all folders and dependencies in your Google Drive (associated with your Colab Pro account). This allows you to store outputs and save models without redownloading them each time. You definitely need a paid account as these models combined are too large for the free account. Ensure you have 40-50 GB of free storage in your Drive as a safety net.

You can use this guide to set up your own ipynb Notebook.

Once you've installed ComfyUI, if you want to restart the instance, you just have to enter:

```python
from google.colab import drive
drive.mount('/content/drive')
%cd /content/drive/MyDrive/ComfyUI
# Check if we are in the ComfyUI folder
!pwd
```

And then start ComfyUI (skip to the code that runs ComfyUI with Cloudflare).
Select the A100 graphics card (or any other you can directly see their VRAM when clicking on it): 
![image](https://github.com/user-attachments/assets/564d87c6-3937-4f4d-b242-8e9f3224b200)
![image](https://github.com/user-attachments/assets/59fcfbea-53f4-4ccd-a316-9aad17fcf977)


This is in German now but you can see the hourly rates a certain card costs you with the settings you have - a T4 with 15GB of GPU RAM uses about 1.44 hours. With advanced RAM its 1.67 - I think to minimize costs you can go with a T4 and use normal RAM. You have to click into the usage field itself not the drop down selector to the right to see that. 


![Screenshot 2025-04-05 123120](https://github.com/user-attachments/assets/8ada20a6-2b6a-423d-a434-777d49c94291)

Also make sure that under Manage Sessions you always just have one active notebook. Otherwise it constantly takes from your monthly usage credits... (if you open a new Colab Notebook and did NOT selected disconnect and delete runtime)
![image](https://github.com/user-attachments/assets/bc82183a-6c7c-40b8-a338-aceb30a0f7c2)


## 1. Installing ComfyUI and connecting it to your Drive (ONLY FOR THE FIRST TIME!!):

```python
# #@title Environment Setup

from pathlib import Path

OPTIONS = {}

USE_GOOGLE_DRIVE = True  #@param {type:"boolean"}
UPDATE_COMFY_UI = True  #@param {type:"boolean"}
USE_COMFYUI_MANAGER = True  #@param {type:"boolean"}
INSTALL_CUSTOM_NODES_DEPENDENCIES = True  #@param {type:"boolean"}
OPTIONS['USE_GOOGLE_DRIVE'] = USE_GOOGLE_DRIVE
OPTIONS['UPDATE_COMFY_UI'] = UPDATE_COMFY_UI
OPTIONS['USE_COMFYUI_MANAGER'] = USE_COMFYUI_MANAGER
OPTIONS['INSTALL_CUSTOM_NODES_DEPENDENCIES'] = INSTALL_CUSTOM_NODES_DEPENDENCIES

current_dir = !pwd
WORKSPACE = "/content/ComfyUI"

if OPTIONS['USE_GOOGLE_DRIVE']:
    !echo "Mounting Google Drive..."
    %cd /

    from google.colab import drive
    drive.mount('/content/drive')

    WORKSPACE = "/content/drive/MyDrive/ComfyUI"
    %cd /content/drive/MyDrive

![ ! -d $WORKSPACE ] && echo -= Initial setup ComfyUI =- && git clone https://github.com/comfyanonymous/ComfyUI
%cd $WORKSPACE

if OPTIONS['UPDATE_COMFY_UI']:
  !echo -= Updating ComfyUI =-

  # Correction of the issue of permissions being deleted on Google Drive.
  ![ -f ".ci/nightly/update_windows/update_comfyui_and_python_dependencies.bat" ] && chmod 755 .ci/nightly/update_windows/update_comfyui_and_python_dependencies.bat
  ![ -f ".ci/nightly/windows_base_files/run_nvidia_gpu.bat" ] && chmod 755 .ci/nightly/windows_base_files/run_nvidia_gpu.bat
  ![ -f ".ci/update_windows/update_comfyui_and_python_dependencies.bat" ] && chmod 755 .ci/update_windows/update_comfyui_and_python_dependencies.bat
  ![ -f ".ci/update_windows_cu118/update_comfyui_and_python_dependencies.bat" ] && chmod 755 .ci/update_windows_cu118/update_comfyui_and_python_dependencies.bat
  ![ -f ".ci/update_windows/update.py" ] && chmod 755 .ci/update_windows/update.py
  ![ -f ".ci/update_windows/update_comfyui.bat" ] && chmod 755 .ci/update_windows/update_comfyui.bat
  ![ -f ".ci/update_windows/README_VERY_IMPORTANT.txt" ] && chmod 755 .ci/update_windows/README_VERY_IMPORTANT.txt
  ![ -f ".ci/update_windows/run_cpu.bat" ] && chmod 755 .ci/update_windows/run_cpu.bat
  ![ -f ".ci/update_windows/run_nvidia_gpu.bat" ] && chmod 755 .ci/update_windows/run_nvidia_gpu.bat

  !git pull

!echo -= Install dependencies =-
!pip3 install accelerate
!pip3 install einops transformers>=4.28.1 safetensors>=0.4.2 aiohttp pyyaml Pillow scipy tqdm psutil tokenizers>=0.13.3
!pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121
!pip3 install torchsde
!pip3 install kornia>=0.7.1 spandrel soundfile sentencepiece

if OPTIONS['USE_COMFYUI_MANAGER']:
  %cd custom_nodes

  # Correction of the issue of permissions being deleted on Google Drive.
  ![ -f "ComfyUI-Manager/check.sh" ] && chmod 755 ComfyUI-Manager/check.sh
  ![ -f "ComfyUI-Manager/scan.sh" ] && chmod 755 ComfyUI-Manager/scan.sh
  ![ -f "ComfyUI-Manager/node_db/dev/scan.sh" ] && chmod 755 ComfyUI-Manager/node_db/dev/scan.sh
  ![ -f "ComfyUI-Manager/node_db/tutorial/scan.sh" ] && chmod 755 ComfyUI-Manager/node_db/tutorial/scan.sh
  ![ -f "ComfyUI-Manager/scripts/install-comfyui-venv-linux.sh" ] && chmod 755 ComfyUI-Manager/scripts/install-comfyui-venv-linux.sh
  ![ -f "ComfyUI-Manager/scripts/install-comfyui-venv-win.bat" ] && chmod 755 ComfyUI-Manager/scripts/install-comfyui-venv-win.bat

  ![ ! -d ComfyUI-Manager ] && echo -= Initial setup ComfyUI-Manager =- && git clone https://github.com/ltdrdata/ComfyUI-Manager
  %cd ComfyUI-Manager
  !git pull

%cd $WORKSPACE

if OPTIONS['INSTALL_CUSTOM_NODES_DEPENDENCIES']:
  !echo -= Install custom nodes dependencies =-
  !pip install GitPython
  !python custom_nodes/ComfyUI-Manager/cm-cli.py restore-dependencies
```

## 2. Install torchsde

```bash
!pip install torchsde
```

## 3. Go to the models folder

## 4. Install all the models we need within ComfyUI

You can install what you want, but stick to the ComfyUI guide as it perfectly explains what you need to install for the sample JSON workflows to work and where to store the files:
https://comfyanonymous.github.io/ComfyUI_examples/wan/

### Text-to-Video Models
```bash
# 14B Model (highest quality, requires more VRAM)
wget -c https://huggingface.co/Comfy-Org/Wan_2.1_ComfyUI_repackaged/resolve/main/split_files/diffusion_models/wan2.1_t2v_14B_fp16.safetensors -P /content/drive/MyDrive/ComfyUI/models/diffusion_models/

# 1.3B Model (faster, less VRAM) I guess that's not why we are using Google Colab right?
wget -c https://huggingface.co/Comfy-Org/Wan_2.1_ComfyUI_repackaged/resolve/main/split_files/diffusion_models/wan2.1_t2v_1.3B_fp16.safetensors -P /content/drive/MyDrive/ComfyUI/models/diffusion_models/
```

### Image-to-Video Models
```bash
# 14B 480P Model
wget -c https://huggingface.co/Comfy-Org/Wan_2.1_ComfyUI_repackaged/resolve/main/split_files/diffusion_models/wan2.1_i2v_480p_14B_fp16.safetensors -P /content/drive/MyDrive/ComfyUI/models/diffusion_models/

# 14B 720P Model
wget -c https://huggingface.co/Comfy-Org/Wan_2.1_ComfyUI_repackaged/resolve/main/split_files/diffusion_models/wan2.1_i2v_720p_14B_fp16.safetensors -P /content/drive/MyDrive/ComfyUI/models/diffusion_models/
```

### Required Support Files
```bash
# Text Encoder
wget -c https://huggingface.co/Comfy-Org/Wan_2.1_ComfyUI_repackaged/resolve/main/split_files/text_encoders/umt5_xxl_fp16.safetensors -P /content/drive/MyDrive/ComfyUI/models/text_encoders/

# VAE
wget -c https://huggingface.co/Comfy-Org/Wan_2.1_ComfyUI_repackaged/resolve/main/split_files/vae/wan_2.1_vae.safetensors -P /content/drive/MyDrive/ComfyUI/models/vae/

# CLIP Vision (for image-to-video)
wget -c https://huggingface.co/Comfy-Org/Wan_2.1_ComfyUI_repackaged/resolve/main/split_files/clip_vision/clip_vision_h.safetensors -P /content/drive/MyDrive/ComfyUI/models/clip_vision/
```

### Workflows
```bash
# Download Workflows
mkdir -p /content/drive/MyDrive/ComfyUI/workflows/
wget -c https://comfyanonymous.github.io/ComfyUI_examples/wan/text_to_video_wan.json -P /content/drive/MyDrive/ComfyUI/workflows/
wget -c https://comfyanonymous.github.io/ComfyUI_examples/wan/image_to_video_wan_example.json -P /content/drive/MyDrive/ComfyUI/workflows/
```

## 5. Start ComfyUI

```python
!wget -P ~ https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
!dpkg -i ~/cloudflared-linux-amd64.deb

import subprocess
import threading
import time
import socket
import urllib.request

def iframe_thread(port):
  while True:
      time.sleep(0.5)
      sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
      result = sock.connect_ex(('127.0.0.1', port))
      if result == 0:
        break
      sock.close()
  print("\nComfyUI finished loading, trying to launch cloudflared (if it gets stuck here cloudflared is having issues)\n")

  p = subprocess.Popen(["cloudflared", "tunnel", "--url", "http://127.0.0.1:{}".format(port)], stdout=subprocess.PIPE, stderr=subprocess.PIPE)
  for line in p.stderr:
    l = line.decode()
    if "trycloudflare.com " in l:
      print("This is the URL to access ComfyUI:", l[l.find("http"):], end='')
    #print(l, end='')

threading.Thread(target=iframe_thread, daemon=True, args=(8188,)).start()
#ADJUST THIS PATH BASED ON YOUR INSTALLATION
!python /content/ComfyUI/main.py --dont-print-server
```

ComfyUI should start on a Cloudflare subdomain - click one of them (it takes some time)
![image](https://github.com/user-attachments/assets/1726e4d1-0e5b-4adc-a813-ccb56c2454fe)

If you do nothing for a long time, Google Colab will cut the connection since you are "paying" in Runtime hours. If you are inactive, they will terminate the connection, meaning you will also have to restart ComfyUI, which takes time. To prevent this, install a Google Chrome extension like: https://chromewebstore.google.com/detail/google-colab-keep-alive/bokldcdphgknojlbfhpbbgkggjfhhaek
It will click on the screen from time to time. There's no guarantee of it working properly, but it's a workaround.

## Installing Video Helper Suite

Install the Video Helper Suite to be able to save the results as MP4:
![image](https://github.com/user-attachments/assets/72ab4d98-2e94-4058-b8ad-68861d1f5d74)

1. Click on Custom Nodes Manager
2. Search for Video Helper Suite and click on Install
3. You will be prompted with a restart and a reload of the screen. Click on that.

![image](https://github.com/user-attachments/assets/b10caa8f-2208-4c72-be9f-271cb05e6bf3)
![image](https://github.com/user-attachments/assets/1ec65e4b-c6d3-43d8-8080-5672baba9573)

## Working with Workflows

Drag and Drop the downloaded sample JSON Workflows from https://comfyanonymous.github.io/ComfyUI_examples/wan/ onto your screen:
- Text-to-Video: https://comfyanonymous.github.io/ComfyUI_examples/wan/text_to_video_wan.json
- Image-to-Video: https://comfyanonymous.github.io/ComfyUI_examples/wan/image_to_video_wan_example.json

![image](https://github.com/user-attachments/assets/0fc43573-2f66-4aa5-a3f6-511820aa7add)

Now double click on an empty space next to the WebP module and enter "Video Combine" and connect it to the output:
![image](https://github.com/user-attachments/assets/44bf59f1-2367-401b-9fc6-b5f95b563091)
![image](https://github.com/user-attachments/assets/421a56e0-19c2-4d54-975a-98ac5a065044)

Now change the output format to MP4 or leave it as GIF, whichever you prefer:
![image](https://github.com/user-attachments/assets/958d4665-7980-4520-bd14-679f6ddda068)

## Running Your First Generation

Make sure you selected all the correct models:
![image](https://github.com/user-attachments/assets/ceee869c-b7f5-4474-9af4-90b23c2cfce0)

I made the same error of not checking, as the template we dropped in selects its models even if they are not installed in our ComfyUI folder. So click in all fields and make sure you select the right ones from the Wan Github ComfyUI Guide.

Note that if something isn't installed yet or not in the correct folder, you might have to fully relaunch the Code after storing the models in the respective folders. You can always manually upload the models to your drive - it will just take way longer than doing it via Colab.

Click on Run:
![image](https://github.com/user-attachments/assets/e32860cb-8483-4976-afbe-1e6a6bde8bc7)

You'll see output like this:
```
BATCH:
Using pytorch attention in VAE
Using pytorch attention in VAE
VAE load device: cuda:0, offload device: cpu, dtype: torch.bfloat16
CLIP/text encoder model load device: cuda:0, offload device: cpu, current: cpu, dtype: torch.float16
Requested to load WanTEModel
loaded completely 38863.675 10835.4765625 True
BATCHEND
```

It takes some time and shows the status on top:
![image](https://github.com/user-attachments/assets/9a05f90d-8649-4394-9358-f78a418d6547)

The output shows in ComfyUI and should also be stored as MP4:
![image](https://github.com/user-attachments/assets/64d272dd-8eae-4774-8cba-d3424767bd6f)
![image](https://github.com/user-attachments/assets/dc06e9a7-07b3-465a-8d77-3a4b41cba428)

## Adjusting Settings

You can adjust the settings to increase the resolution as well as the length of the video:
![image](https://github.com/user-attachments/assets/015c03d2-564f-4730-913a-10f04d0e9336)

Since we are connected to an A100, it can easily handle these settings, yet it takes some time. Be aware of that.

On the SaveAnimatedWEBP module, it tells you how many FPS the generation is (currently 16fps) - which means the length parameter of 33 is about a 2-second long video. You can calculate for yourself what you need to enter and how long it's going to be. To generate 720p HD videos, change the width to 1280 and height to 720.

Note: It takes around 7 minutes with the A100 for a 2-second 720p video.

## Saving Your Workflows

If you make any changes to your workflows, they are saved to the workflows folder within ComfyUI:
![image](https://github.com/user-attachments/assets/c598aa04-521c-41d3-9357-34eccbc916bb)

This means the easiest way would be to download the JSON flow from that folder and drag it to the ComfyUI screen if you restart your runtime environment. This way you are being safe.



## Wan2.1 I2V Setup Note

1. Drag the I2V workflow (from https://comfyanonymous.github.io/ComfyUI_examples/wan/image_to_video_wan_example.json) that we downloaded before to the ComfyUI screen and adjust the models again.
   
   Note: This flow adds a clip_vision model but we downloaded that before.
   
   Before starting you can add the video output mp4 as we did before.
   
   ![image](https://github.com/user-attachments/assets/b3df833f-6621-430c-a315-f962742c78ff)

2. Upload an image and click on Run
   
   ![image](https://github.com/user-attachments/assets/1b0407eb-1800-44b3-9520-10c01be7a017)
   ![image](https://github.com/user-attachments/assets/c36aa508-94df-45b4-be23-3004e8de754b)
   ![image](https://github.com/user-attachments/assets/00f1b1cd-c5c9-4c6b-9bf3-d0728c9af41a)

A 512x512 2 second clip took 344.84 seconds with the A100 GPU. (input size was 1500x1500, with smaller images it takes around 90 seconds if you select 512x512)

Not the best result but on first shot with a random prompt and definitely fun to play around with ;-)... 
https://github.com/user-attachments/assets/5ad49559-f1b5-443b-aaca-18d26695a54a

Note: To improve the output you should input the image as the same size as the output size you define.


A simple N8N workflow would then help you delete the file types that you don't need.
![image](https://github.com/user-attachments/assets/693cd0d1-4eb4-49a4-afb4-2e9e99e126e9)


