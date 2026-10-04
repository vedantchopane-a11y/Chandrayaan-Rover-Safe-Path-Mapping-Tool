# Chandrayaan-Rover-Safe-Path-Mapping-Tool
Deep Learning Approach for Identification of Safe Navigation Routes on the Moon

The source code for this project is kept private as proprietary intellectual property due to the extensive development and commercial potential of the codebase. However, a detailed overview of the system architecture, features, and outputs is provided below.



# Elevation-Aware Deep Learning Framework for Safe Lunar Rover Path Planning

A deep learning and geospatial framework for detecting lunar craters and generating safe, elevation-aware navigation paths for lunar rovers using Chandrayaan-2 datasets.

## Overview

Autonomous navigation on the lunar surface is challenging due to craters, irregular terrain, elevation variations, and limited surface information. This project proposes an integrated system that combines deep learning-based crater detection with terrain analysis and A* path planning to generate safer rover navigation routes.

The system uses high-resolution Chandrayaan-2 imagery for crater detection and TMC-2 terrain data for geospatial mapping and elevation-aware path planning.


## Objectives

The main objectives of this project are to:

- Detect lunar craters using deep learning-based object detection models.
- Compare different YOLOv5 and YOLOv8 variants and identify a suitable model for crater detection.
- Convert detected crater locations from image/tile coordinates into georeferenced spatial data.
- Integrate crater detections with Chandrayaan-2 TMC-2 Ortho and DTM datasets.
- Develop a custom PyQt-QGIS application for visualizing lunar terrain and detected hazards.
- Implement an elevation-aware A* path planning algorithm for generating safer rover routes.
- Consider crater avoidance, elevation variation, and movement distance during path planning.

## Datasets

The project uses datasets from the **Chandrayaan-2 mission**, primarily OHRC and TMC-2 data.

### OHRC Dataset

The Orbiter High Resolution Camera (OHRC) imagery was used for crater detection and model training.

The labelled dataset consists of **640 × 640 image tiles** with five annotated classes:

- Boulders
- Plain
- Small Craters
- Medium Craters
- Large Craters

Dataset distribution:

| Split | Images |
|---|---:|
| Training | 576 |
| Validation | 40 |
| Testing | 40 |

For the crater detection experiments, boulders and the small, medium, and large crater classes were considered.

### TMC-2 Dataset

TMC-2 data was used for terrain mapping and path planning:

- **TMC-2 Ortho:** approximately 5 m/pixel spatial resolution
- **TMC-2 DTM:** elevation data used for terrain analysis and path planning

## Crater Detection

Several YOLO models were trained and evaluated to determine a suitable model for lunar crater detection:

- YOLOv5s
- YOLOv5m
- YOLOv5l
- YOLOv5x
- YOLOv8s
- YOLOv8m
- YOLOv8l
- YOLOv8x

The models were trained for **100 epochs** and evaluated using:

- Precision
- Recall
- mAP@0.5
- mAP@0.5:0.95

### Selected Model

**YOLOv8m** was selected for integration into the navigation pipeline based on the comparative evaluation.

| Metric | YOLOv8m |
|---|---:|
| Precision | 79.19% |
| Recall | 26.21% |
| mAP@0.5 | 32.34% |
| mAP@0.5:0.95 | 16.80% |

The selected model was subsequently used to detect craters on TMC-2 Ortho image tiles.

## Geospatial Processing

The detected crater bounding boxes initially exist in local image-tile coordinates. A coordinate transformation pipeline was developed to convert these detections into georeferenced spatial data.

The process includes:

1. Identifying the position of each image tile.
2. Converting local bounding-box coordinates into global image coordinates.
3. Applying rotation when required.
4. Applying the world-file affine transformation.
5. Assigning the lunar south-polar Coordinate Reference System (CRS).
6. Creating polygon geometries for detected craters.
7. Exporting the detections as a **GeoPackage (`.gpkg`)**.

The resulting georeferenced crater layer can then be overlaid on the TMC-2 lunar imagery.

## Custom PyQt-QGIS Application

A custom desktop application was developed using **PyQt** and the **QGIS API** to provide an interactive environment for lunar terrain visualization and rover path planning.

The application allows users to:

- Load TMC-2 Ortho imagery.
- Load DTM elevation data.
- Import detected crater GeoPackage files.
- Visualize detected crater locations.
- Add navigation points manually.
- Generate rover paths using the A* algorithm.
- View path distance and accumulated cost.
- Export relevant geospatial outputs.

The application combines the deep learning detection results with geospatial and terrain information in a single workflow.

## Elevation-Aware A* Path Planning

The project uses a modified A* pathfinding approach to generate rover routes while considering both terrain and detected hazards.

The path cost incorporates:

- Euclidean movement distance.
- Elevation variation.
- Uphill movement penalties.
- Downhill movement costs.
- Crater regions as obstacles.

The basic A* formulation is:

$$
f(n) = g(n) + h(n)
$$

where:

- `g(n)` represents the accumulated movement and elevation cost.
- `h(n)` represents the Euclidean-distance heuristic.

A maximum slope constraint of **12°** was incorporated into the terrain traversal logic.

For a horizontal distance of 10 m:

$$
\Delta h = \tan(12^\circ) \times 10
$$

which gives a maximum elevation difference of approximately **2.13 m**.

This allows the path planner to penalize or avoid terrain changes that exceed the defined traversal constraint.

## Results

The complete workflow connects crater detection, geospatial processing, terrain analysis, and path planning.

The system successfully:

- Detected lunar craters using YOLO-based object detection.
- Converted detections into georeferenced crater polygons.
- Visualized crater locations over TMC-2 imagery.
- Integrated DTM elevation information into path planning.
- Generated rover paths between user-defined navigation points.
- Avoided detected crater regions and penalized unsafe elevation changes.

The final paths can be visualized directly within the custom PyQt-QGIS application.

## Technologies Used

### Machine Learning
- Python
- YOLOv5
- YOLOv8
- Deep Learning
- Object Detection

### Geospatial Processing
- QGIS
- QGIS API
- GeoPandas
- Shapely
- NumPy
- Pandas
- GeoPackage
- Coordinate Reference Systems (CRS)
- Digital Terrain Models (DTM)

### Application Development
- PyQt
- Python
- QGIS

### Path Planning
- A* Algorithm
- Terrain Analysis
- Elevation-Aware Cost Function
- Obstacle Avoidance

## Limitations

The current implementation has several limitations:

- TMC-2 imagery has a spatial resolution of approximately 5 m/pixel, limiting fine-grained and sub-meter hazard analysis.
- The system assumes relatively static terrain.
- Changing illumination and shadows are not dynamically modeled.
- False positives and false negatives in crater detection can influence the resulting path.
- Real-time rover imagery has not been incorporated for final route validation.

## Future Work

Future improvements could focus on:

- Exploring transformer-based object detection models.
- Using ensemble techniques to improve crater detection robustness.
- Integrating multispectral or temporal lunar imagery.
- Adding 3D terrain visualization.
- Incorporating real-time rover imagery for route validation.
- Automatically identifying scientifically significant navigation targets.
- Exploring hybrid and more advanced path-planning algorithms.


## Project Pipeline

```text
Chandrayaan-2 Datasets
        │
        ├── OHRC Images
        │       ↓
        │   Dataset Preparation
        │       ↓
        │   YOLOv5 / YOLOv8 Training
        │       ↓
        │   Crater Detection
        │
        └── TMC-2 Ortho + DTM
                ↓
        Terrain & Elevation Data
                │
                ↓
       Coordinate Transformation
                │
                ↓
       GeoPackage (.gpkg)
                │
                ↓
       PyQt + QGIS Application
                │
                ↓
       User-defined Navigation Points
                │
                ↓
       Elevation-Aware A* Algorithm
                │
                ↓
          Safe Rover Path
