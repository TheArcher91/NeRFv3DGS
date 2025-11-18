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
4. Run the ns-process-data command. This will find colmap.exe in your PATH and process the cropped images.

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



# Phase 4: Getting Final Results

Goal: Wait for training to finish, then export your video, metrics table, and graphs.

1. Let Training Finish: Wait for the ns-train terminal (your first terminal) to complete its 30,000 steps.

2.Render Your Video: You need to go the live rendering web page of nerfstudio and select a few keyframes (basically location and pose of cameras) and click on generate command and copy the command. After training stops, run the ns-render command you saved from the viewer (Sample Link from my case):

```bash
ns-render camera-path --load-config outputs\rtcw_crates_cropped\nerfacto\2025-11-16_054618\config.yml --camera-path-filename C:\Users\ARITRA\nerfstudio-projects\data\rtcw_crates_cropped\camera_paths\2025-11-16-05-46-23.json --output-path renders/rtcw_crates_cropped/2025-11-16-05-46-23.mp4
```

3. Get Final Metrics Table: After rendering, run the ns-eval command. This points to the exact config.yml from your training run and saves the results to a file named rtcw_metrics.json.

```bash
ns-eval --load-config "outputs\rtcw_crates_cropped\nerfacto\2025-11-16_054618\config.yml" --output-path "rtcw_metrics.json"
```
I can then open this rtcw_metrics.json file in VS Code to get the final numbers.

4. Save Your Graphs: Go to your TensorBoard page and save the graphs containing LPIPS, PSNR, SSIM and depth map etc. You can now safely shut down the process by pressing Ctrl+C in the nvidia-smi terminal and the tensorboard terminal to close them.




# 3DGS Workflow:

Goal: Train the same dataset using splatfacto and compare the results with that of trained using nerfacto

You must be in your Administrator terminal, with nerfstudio active, and in the projects directory. This step may particularly be irrelevant to general users. I needed to fix one critical error with ninja build process. Set the environment variables that fix the gsplat build: 

```bash
:: This tells the compiler to define the "ignore" flag for the CUB/Thrust conflict
set NVCC_APPEND_FLAGS=-DTHRUST_IGNORE_CUB_VERSION_CHECK

:: This tells the build script where your patched CUB 1.17.0 folder is
set CUB_HOME="C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v11.8\include"
```

Run the final ns-train command. This will first trigger the gsplat build and then start the training.

```bash
ns-train splatfacto --data data/rtcw_crates_cropped --vis viewer+tensorboard nerfstudio-data --eval-mode fraction --train-split-fraction 0.9
```

The training should have started at this point and in the meantime you can check in on the gpu percentage and other performance parameter graphs like PSNR by running the TensorBoard server.


Let the process to finish. In my scenario the steps taken by splatfacto was 29900 and while in case of nerfacto it was 29500. You can get the .json file prepared containing relevant informations as (in my case):

```bash
ns-eval --load-config "outputs\rtcw_crates_cropped\splatfacto\2025-11-16_161105\config.yml" --output-path "rtcw_splatfacto_metrics.json"
```

If you have kept the command generated after adding the keyframes you can run it now. Usually you should see drastic fall in the video rendering time. In case of nerfacto it was ~3.5 hr while with splatfacto it was 1.5 minutes! There was certain issues with artifacts floating and fogs with both of them. More on this can be found on the relevant folders within the main branch and in the paper.

```bash
ns-render camera-path --load-config outputs\rtcw_crates_cropped\splatfacto\2025-11-16_161105\config.yml --camera-path-filename C:\Users\ARITRA\nerfstudio-projects\data\rtcw_crates_cropped\camera_paths\2025-11-16-16-11-09.json --output-path renders/rtcw_crates_cropped/2025-11-16-16-11-09.mp4
```





# Depth Map Generated by Nerfacto:


![Depth Map 0](https://github.com/TheArcher91/NeRFv3DGS/blob/main/rtcw/NeRF/depth.png?raw=true)

# SSIM comparison:


![NeRF v 3DGS](https://github.com/TheArcher91/NeRFv3DGS/blob/main/rtcw/3DGS/Eval%20Images%20Metrics-ssim.png?raw=true)





# Drive Link :

The following link contains all the relevant files along with the paper describing the comparison between the two methods in detail.

- 📖 [Google drive](https://drive.google.com/drive/folders/1eTfcvEhTG1rrAkEWmcNGgAQLKOJeqbTo?usp=sharing)
