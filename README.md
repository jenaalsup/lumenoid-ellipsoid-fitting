## Overview
This project provides a simple pipeline for segmenting lumen cavities from 3D `.tif` stacks and fitting ellipses to their inner and outer boundaries. The goal is to extract geometric features of each lumen (e.g., major/minor axes, aspect ratio, orientation) for downstream analysis of lumenoid morphology. Segmentation is done manually-assisted using Napari with the [nnInteractive](https://github.com/MIC-DKFZ/nnInteractive) plugin, and the resulting masks are processed by a Python script that performs ellipse fitting. This repository includes instructions for inner cavity segmentation, outer boundary segmentation, and running the final ellipse analysis.

## Inner cavity segmentation (with Napari/nnInteractive)

1. Open Anaconda Prompt  
   (On Windows, click the Start button and search for "Anaconda Prompt")

2. Create a new environment  
   ```
   conda create -n nninteractive python=3.10 -y
   ```
   Note: If this generates an error related to retry failed, just rerun the command.
   
4. Activate the environment  
   ```
   conda activate nninteractive
   ```

5. Install Napari and the nnInteractive plugin  
   ```
   pip install "napari[all]" napari-nninteractive
   ```

6. Launch Napari with the plugin  
   ```
   napari -w napari-nninteractive
   ```
   This opens the Napari GUI with the nnInteractive plugin already open.

7. Drag the raw .tif file into the GUI (any number of lumens ok).

8. Press Initialize in the right-side panel. This will open an nnInteractive label layer that looks like the screenshot below:  
   <img width="2171" height="1308" alt="Screenshot 2025-11-09 153413" src="https://github.com/user-attachments/assets/0349e7d0-f95e-4fe2-b7f9-465d590b0325" />

9. Under "Prompt type", select "Positive", and under "Interaction tools", select "Point".

10. Click inside the lumen cavities to place positive points (~1 per lumen or as needed).  
   When placing points, note that the cavity will be colored red; it will take ~4 seconds for this to show up.  
   Only if needed, switch the prompt type to "Negative" and place points outside of the cavity. This is to correct for overshoot (eg the cavity is bigger than intended).  
   Example slice of lumens after placing 1 positive point in the left lumen and 3 positive points in the right lumen:  
    <img width="2170" height="1300" alt="Screenshot 2025-11-09 154239" src="https://github.com/user-attachments/assets/6cb73586-c863-48ef-894d-ae22b20a1ba2" />

11. Go to File > Save Selected Layer(s) and choose the Labels layer, which will export a tif file.


## Outer lumen wall segmentation (with Napari/nnInteractive)
The setup and workflow are the same as the Inner cavity segmentation section, with just a few differences:
- Use ~3–5 positive points per lumen (more than for inner) to fill the entire outer boundary (including the inner cavity too).
- Save each lumen as its own file:
   - After segmenting the first lumen, press Next object so the next lumen is written to a new layer.
   - Then save each layer individually via File → Save Selected Layer(s).
- Note: Avoid trying to segment inner + outer at the same time using mixed positive/negative prompts — this usually breaks the segmentation.
- Note: Napari may crash after 3+ lumens. If it happens, just save and reopen the tif before continuing.
<img width="2154" height="1380" alt="image" src="https://github.com/user-attachments/assets/bee96b17-602b-4245-9c67-22b227256a71" />

## Ellipsoid fitting
1. Move all files outputted from the previous step to be in this file structure, where each file has its own folder and for n lumens, there are n + 1 files. Ensure that the file containing the segmented inner cavities includes the string "inner_labels" in the name. <img width="784" height="311" alt="Screenshot 2025-11-12 at 2 28 25 PM" src="https://github.com/user-attachments/assets/c4fac917-5910-42bd-aa61-dc54f354a446" />

2. Set up Python virtual environment by running these commands in the Terminal (only need to do this once)
   ```
   python3 -m venv venv

   source venv/bin/activate

   pip install tifffile pandas numpy scipy scikit-image matplotlib h5py pyvista trimesh pymeshfix napari "PyQt5"
   ```

3. Export the segmentation directory (optional if code is changed to include correct directory pointing towards segmented tif files)
   ```
   export SEGMENTATION_DIR="..."
   ```
   (e.g. /Users/jenaalsup/Desktop/segmentation-testing)
   
4. Run the ellipse analysis
   ```
   python3 fit-ellipsoids.py
   ```

Note: the output is a csv file with one row each lumen and columns representing the parameters for both the inner and outer ellipsoid
   
## Analysis
1. Move day 2, day 3, day 4 CSV outputs of fit-ellipsoids.py into a folder (some example data is stored at fake_data in this repository)
2. Run the Jupyter Notebook data-analysis.ipynb, results will be stored in plots/fake_data_plots.pdf


---
## [deprecated] Outer lumen wall segmentation (with thresholding) - alternative to Napari
1. Export the path to the raw image
   ```
   export IMAGE_PATH="..." 
   ```
   (e.g. /Users/jenaalsup/Desktop/CKHRJQ~2.TIF)

2. Run the segmentation
   ```
   python3 segment-outer.py (expect this to take ~2-3 minutes)
   ```
   
Note: Deprecated b/c requires finetuning most images
