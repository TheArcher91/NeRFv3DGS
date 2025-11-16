# NeRFv3DGS
Comparing performance parameters of custom dataset under training with NeRF and 3DGS

# Phase 1: Data Preparation (Cropping Images)

1. Open a regular "x64 Native Tools Command Prompt for VS 2022 LTSC 17.8" (or any command prompt)

2. I had already put all my dataset images in a folder inside nerfstudio-projects folder in C drive. But I needed to crop a small portion of black strip off every image as it was greatly affecting the rendering especially the top portion or the sky region. Let's navigate to the folder containing the uncropped files.

```bash
cd %USERPROFILE%\nerfstudio-projects\data\rtcw_crates_original
```
