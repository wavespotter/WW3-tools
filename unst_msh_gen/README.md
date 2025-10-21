# Project Name
Unstructured mesh generation for WW3 using JIGSSAW.
# Description
Mesh generation script capable of creating unstructured meshes for WW3 global modeling. The tool leverages JIGSAWPY (https://github.com/dengwirda/jigsaw-python) for efficient triangulation.
Main changes include:
Implementation of the ocn_ww3.py to create uniform and variable unstructured mesh for global WW3 model.
This tool is under active development, with future work focused on variable unstructured mesh generation.

# Installation

## Quick Start

### 1- Clone the repository
```bash
git clone https://github.com/NOAA-EMC/WW3-tools
cd WW3-tools/unst_msh_gen
```

### 2- Install jigsawpy prerequisites
jigsawpy requires a C++ compiler and CMake. Install them based on your system:

**On macOS:**
```bash
brew install cmake
# C++ compiler comes with Xcode Command Line Tools
xcode-select --install
```

**On Ubuntu/Debian:**
```bash
sudo apt-get install cmake build-essential
```

**On Amazon Linux:**
```bash
sudo yum install cmake gcc-c++ make
# Or on Amazon Linux 2023:
sudo dnf install cmake gcc-c++ make
```

**On Windows:**
- Install CMake from https://cmake.org/download/
- Install Microsoft Visual Studio Build Tools or MinGW

### 3- Create a virtual environment
```bash
python3 -m venv ./.venv
source ./.venv/bin/activate  # On Windows: .\.venv\Scripts\activate
```

### 4- Install jigsawpy from source
```bash
# Clone jigsawpy repository
git clone https://github.com/dengwirda/jigsaw-python.git
cd jigsaw-python

# Build and install
python3 build.py
pip install .

# Return to unst_msh_gen directory
cd ../unst_msh_gen
```

### 5- Install remaining dependencies
```bash
pip install --upgrade pip
pip install -r requirements.txt
```

### 6- Download the DEM data
```bash
aws s3 cp s3://sofar-wx-constants/ww3-model/grid/GEBCO_2025_sub_ice.nc .
```

Make sure the DEM file is in the `WW3-tools/unst_msh_gen` directory.
 
# Usage
## 7- Activate the virtual environment (if not already active)
```bash
source ./.venv/bin/activate  # On Windows: .\.venv\Scripts\activate
```

## 8- Run the script inside of WW3-tools/unst_msh_gen:

- modify config.ini:
- Specify the DEM netcdf file using the 'dem_file' in "DataFiles" section.
- For uniform resolution 'hmax' = 'hmin' = 'hshr' = 'hfun_max' and 'nwave' = 0 in "Spacing" section.
- To include or exclude Black-Sea change 'black_sea' in "CommandLineArg" section to:
- 	3 will have the Black Sea and the connections.
- 	2 will have the Balck sea as a seperate basin.
- 	1 will exclude the Black sea
- you can specify the mesh name (jigsaw format or ww3 format) using 'mesh_file' and 'ww3_mesh_file'in "MeshSetting" section 

- $python3 ocn_ww3.py --config config.ini

NOTE: the output will be gmsh format which will be used by WW3 (specified by 'ww3_mesh_file')

NOTE: The output mesh will have -180:180 longitude, you can convert this by unisg ShiftMesh.py script, to 0:360 longitude.
	input_file_path: your jigsaw mesh in gmsh format with -18:180 long
	output_file_path: shifted mesh in gmsh format with 0:360 long


## 9- Using variable mode:

- modify config.ini
- To create a mesh with finer resolution near the US coastlines you can define different region in json format (east coast, west coast and golf od Mexico, Purto Rico, and Hawaii); use the 'window_file' in the "DataFiles" section to specify the windows.

NOTE: hshr is the shoreline resolution which can be smaller than the hmin which is defined globally.

- window_mask.py can read shapefiles in json format as well and assign user defined resolution (scale) to each polygon; use the 'shape_file' in the "DataFiles" to specify the shapefiles.

NOTE: You can create multiple shapefiles using QGIS or ArcGIS and save them in ./Shapefiles directory and assign different resolution (scale) to each polygone.

NOTE: You can define different background mesh based on lat location in the window_mask.py via config.init
		
NOTE: for the background mesh you can define different resolution (for eaxmple, the globe is divided to three regions based on latitude and corresponding resolutions) in "Scalingsettings" of the config.ini as;

- upper_bound = 50          # Lattitude that starts the upper section (ie lat > upper_bound) 
- middle_bound = -20        # Determines the boundary between the upper  and middle sections.  The middle section is for upper_bound > lat > middle_bound
- lower_bound = -90         # Boundary of lowest section, likely -90,   Lower section is:   middle_bound > lat > lower_bound
    	 
- scale_north = 9           # mesh resolution  in km for upper section ( lat > upper_bound )
- scale_middle = 20         # mesh resolution in km for  middle section for middle_bound < lat < upper_boun
- #Mesh resolution of the lower section is linear from scale_south_upper to scale_south_upper
- scale_south_upper = 30    # km resolution for upper south/lower section 
- scale_south_lower = 9     # km mesh resolution at lower south/lower section 

- $python3 window_mask.py --config config.ini
	
NOTE: The output will be "wmask.nc" file which will have the mesh spacing info.


NOTE:To create the variable mesh based on mesh spacing file "wmask.nc", in the config.ini change the mask_file = wmask.nc

- $python3 ocn_ww3.py --config config.ini


## 10- Plotting the mesh info:

- to plot the elements, mesh size, and bathymetry data, use plot_msh.py and change the 'filename' to ww3 format mesh.
- $python3 plot_msh.py

## Contributing
This is ongoing effort by Ali Salimi-Tarazouj with the great help of Darren Engwirda, JIGSAW developer.

