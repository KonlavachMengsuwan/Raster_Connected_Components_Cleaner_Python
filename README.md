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
from scipy.ndimage import label
import numpy as np

# Define file paths
file_paths = {
    "Built Area": "/content/Built Area_LST.tif",
    "Crops": "/content/Crops_LST.tif",
    "Flooded Vegetation": "/content/Flooded Vegetation_LST.tif",
    "Rangeland": "/content/Rangeland_LST.tif",
    "Trees": "/content/Trees_LST.tif",
    "Water": "/content/Water_LST.tif"
}

# Function to remove small connected pixel groups
def remove_small_pixel_groups(raster_data, min_size=2):
    # Identify all non-zero pixels (assuming zero is the background)
    mask = raster_data > 0
    # Label connected components
    labeled_array, num_features = label(mask, structure=np.ones((3, 3)))
    
    # Get the size of each component
    component_sizes = np.bincount(labeled_array.ravel())
    
    # Create a mask for small components
    small_components = component_sizes < min_size
    small_components_mask = small_components[labeled_array]
    
    # Set small components to zero
    raster_data[small_components_mask] = 0
    
    return raster_data

# Plotting and saving function
def process_and_save_raster(file_path, title, output_path):
    with rasterio.open(file_path) as src:
        raster_data = src.read(1)
        profile = src.profile
        
    # Remove small pixel groups
    cleaned_data = remove_small_pixel_groups(raster_data)
    
    # Save the cleaned raster
    with rasterio.open(output_path, 'w', **profile) as dst:
        dst.write(cleaned_data, 1)
    
    # Plot the cleaned raster
    plt.figure(figsize=(10, 8))
    plt.imshow(cleaned_data, cmap='viridis')
    plt.colorbar(label='Land Surface Temperature')
    plt.title(title)
    plt.xlabel('Longitude')
    plt.ylabel('Latitude')
    plt.show()

# Process each raster file and save the results
for title, path in file_paths.items():
    output_path = path.replace('.tif', '_cleaned.tif')
    process_and_save_raster(path, f"LST - {title}", output_path)

```

