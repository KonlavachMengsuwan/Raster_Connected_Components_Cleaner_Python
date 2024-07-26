# Raster_Connected_Components_Cleaner_Python [Incomplete]
A Python script to remove small connected components in raster files.

**Google Colab: ** https://colab.research.google.com/drive/1yuioqz0zWc3-HoyptlijYgRQflc_k5x2?usp=sharing

## Features

- Reads raster files.
- Removes small connected pixel groups from the raster data.
- Saves the cleaned raster files.
- Plots the cleaned raster data.

## Requirements

- Python 3.x
- rasterio
- matplotlib
- scipy
- numpy

## Code:
```
import matplotlib.pyplot as plt
import rasterio
from rasterio.plot import show
from scipy.ndimage import label
import numpy as np
import os
import pandas as pd

def read_raster(file_path):
    with rasterio.open(file_path) as src:
        raster_data = src.read(1)
        profile = src.profile
    return raster_data, profile

def binarize_raster(raster_data):
    binary_raster = (raster_data > 0).astype(np.int32)
    return binary_raster

def label_connected_components(binary_raster):
    labeled_array, num_features = label(binary_raster, structure=np.ones((3, 3)))
    return labeled_array, num_features

def calculate_component_cells(labeled_array):
    component_sizes = np.bincount(labeled_array.ravel())
    return component_sizes

def remove_small_components(labeled_array, component_sizes, min_cells):
    small_components = np.where(component_sizes < min_cells)[0]
    mask = np.isin(labeled_array, small_components)
    cleaned_binary_raster = labeled_array.copy()
    cleaned_binary_raster[mask] = 0
    cleaned_binary_raster = (cleaned_binary_raster > 0).astype(np.int32)
    return cleaned_binary_raster

def apply_mask(original_raster, cleaned_binary_raster):
    cleaned_raster = np.where(cleaned_binary_raster, original_raster, np.nan)
    return cleaned_raster

def save_raster(output_path, cleaned_raster, profile):
    profile.update(dtype=rasterio.float32, nodata=np.nan)
    with rasterio.open(output_path, 'w', **profile) as dst:
        dst.write(cleaned_raster.astype(rasterio.float32), 1)

def save_component_sizes(file_path, component_sizes):
    df = pd.DataFrame({
        'Label': np.arange(len(component_sizes)),
        'Number_of_Cells': component_sizes
    })
    csv_path = file_path.replace('.tif', '_component_sizes.csv')
    df.to_csv(csv_path, index=False)
    print(f"Component sizes saved to {csv_path}")

def process_raster(file_path, output_path, min_cells):
    raster_data, profile = read_raster(file_path)
    binary_raster = binarize_raster(raster_data)
    labeled_array, num_features = label_connected_components(binary_raster)
    component_sizes = calculate_component_cells(labeled_array)
    save_component_sizes(file_path, component_sizes)  # Save component sizes to CSV
    cleaned_binary_raster = remove_small_components(labeled_array, component_sizes, min_cells)
    cleaned_raster = apply_mask(raster_data, cleaned_binary_raster)
    save_raster(output_path, cleaned_raster, profile)
    return cleaned_raster

def plot_raster(raster_data, title):
    plt.figure(figsize=(10, 8))
    plt.imshow(raster_data, cmap='viridis')
    plt.colorbar(label='Land Surface Temperature')
    plt.title(title)
    plt.xlabel('Longitude')
    plt.ylabel('Latitude')
    plt.show()

# Main script
if __name__ == "__main__":
    file_paths = {
        "Built Area": "Built Area_LST.tif",
        "Crops": "Crops_LST.tif",
        "Flooded Vegetation": "Flooded Vegetation_LST.tif",
        "Rangeland": "Rangeland_LST.tif",
        "Trees": "Trees_LST.tif",
        "Water": "Water_LST.tif"
    }

    output_directory = 'cleaned_rasters'
    os.makedirs(output_directory, exist_ok=True)

    min_cells = 2  # Minimum number of cells to keep

    for title, path in file_paths.items():
        output_path = os.path.join(output_directory, path.replace('.tif', '_cleaned.tif'))
        cleaned_raster = process_raster(path, output_path, min_cells)
        plot_raster(cleaned_raster, f"Cleaned LST - {title}")

```

