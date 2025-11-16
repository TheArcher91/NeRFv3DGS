# NeRFv3DGS
Comparing performance parameters of custom dataset under training with NeRF and 3DGS

# Phase 1: Data Preparation (Cropping Images)

1. Open a regular "x64 Native Tools Command Prompt for VS 2022 LTSC 17.8" (or any command prompt)

2. I had already put all my dataset images in a folder inside nerfstudio-projects folder in C drive. But I needed to crop a small portion of black strip off every image as it was greatly affecting the rendering especially the top portion or the sky region. Let's navigate to the folder containing the uncropped files.

```bash
cd %USERPROFILE%\nerfstudio-projects\data\rtcw_crates_original
```

3. Make a new folder (if it doesn't already exists):

```bash
mkdir ..\rtcw_crates_cropped
```
4. Run the FFmpeg batch-crop command. This will find all .png files, apply your crop (1920:948:0:132 in my case), and save new copies in the cropped folder.

```bash
for %f in (*.png) do ffmpeg -i "%f" -vf "crop=1920:948:0:132" "..\rtcw_crates_cropped\%~nf.png"
```


# Phase 2: Data Processing (Running COLMAP)

Goal: Analyze all your cropped images and create the transforms.json file.

1. Open the "x64 Native Tools Command Prompt" as an ADMINISTRATOR.

2.Activate your nerfstudio environment:

```bash
conda activate nerfstudio-v2
```

3. Navigate to your root project folder (one level above data):

```bash
cd %USERPROFILE%\nerfstudio-projects
```
4. Run the ns-process-data command. This will find colmap.exe in your PATH and process your cropped images.

```bash
ns-process-data images --data data/rtcw_crates_cropped --output-dir data/rtcw_crates_croppedns-process-data images --data data/rtcw_crates_cropped --output-dir data/rtcw_crates_cropped
```


# Phase 3: Training & Monitoring

Goal: Train the nerfacto model, enable the test/eval split, and monitor the process.

1. Start the Training: In the same terminal (still as Admin, in the nerfstudio-projects folder), run the full ns-train command. This is the syntax that includes the test split and enables the evaluation metrics:

```bash
ns-train nerfacto --data data/rtcw_crates_cropped --vis viewer+tensorboard nerfstudio-data --eval-mode fraction --train-split-fraction 0.9
```

2.Watch the 3D Viewer: Your terminal will print a URL. Open it in your browser to watch the 3D model train in real-time.

```bash
http://localhost:7007/
```

3. Check GPU Percentage: Open a NEW, separate terminal. Run the NVIDIA monitor:

```bash
nvidia-smi -l 1
```
Look for the GPU-Util percentage (it should be 90-100%). You can leave this terminal running.

4. Watch Your PSNR Graphs: Open a THIRD, separate terminal (as Admin). Activate your environment to nerfstudio-v2 (my case). Navigate to your project folder. Run the TensorBoard server:

```bash
tensorboard --logdir=outputs
```
Your terminal will print a new URL. Open it in your browser to see your live PSNR, SSIM, and LPIPS graphs.

```bash
http://localhost:6006/
```
You can leave this terminal running.



