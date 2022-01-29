---
layout: post
title: "The Ultimate ROS1 Guide to Laser Filters"
---

# Table of Contents

- [Table of Contents](#table-of-contents)
- [Introduction](#introduction)
- [Types of laser filters](#types-of-laser-filters)
  - [LaserScanIntensityFilter](#laserscanintensityfilter)
    - [Use Cases](#use-cases)
    - [Parameters](#parameters)
    - [Pseudocode](#pseudocode)
      - [*Update Function*](#update-function)
  - [LaserScanRangeFilter](#laserscanrangefilter)
    - [Use Cases](#use-cases-1)
    - [Parameters](#parameters-1)
    - [Pseudocode](#pseudocode-1)
      - [*Update Function*](#update-function-1)
  - [LaserScanAngularBoundsFilter](#laserscanangularboundsfilter)
    - [Use Cases](#use-cases-2)
    - [Parameters](#parameters-2)
  - [LaserScanAngularBoundsFilterInPlace](#laserscanangularboundsfilterinplace)
    - [Parameters](#parameters-3)
  - [LaserScanSectorFilter](#laserscansectorfilter)
    - [Parameters](#parameters-4)
  - [InterpolationFilter](#interpolationfilter)
    - [Use Cases](#use-cases-3)
    - [Parameters](#parameters-5)
    - [Pseudocode](#pseudocode-2)
      - [*Update Function*](#update-function-2)
  - [LaserScanFootprintFilter](#laserscanfootprintfilter)
    - [Use Cases](#use-cases-4)
    - [Parameters](#parameters-6)
    - [Pseudocode](#pseudocode-3)
      - [*Update Function*](#update-function-3)
  - [LaserScanBoxFilter](#laserscanboxfilter)
    - [Use Cases](#use-cases-5)
    - [Parameters](#parameters-7)
    - [Pseudocode](#pseudocode-4)
      - [*Update Function*](#update-function-4)
  - [LaserScanPolygonFilter](#laserscanpolygonfilter)
    - [Use Cases](#use-cases-6)
    - [Parameters](#parameters-8)
    - [Pseudocode](#pseudocode-5)
      - [*Update Function*](#update-function-5)
  - [LaserScanMaskFilter](#laserscanmaskfilter)
    - [Parameters](#parameters-9)
    - [Pseudocode](#pseudocode-6)
  - [ScanShadowsFilter](#scanshadowsfilter)
    - [Parameters](#parameters-10)
    - [Pseudocode](#pseudocode-7)
      - [*Update Function*](#update-function-6)
      - [*isShadow Function*](#isshadow-function)
  - [ScanBlobFilter](#scanblobfilter)
    - [Parameters](#parameters-11)
    - [Pseudocode](#pseudocode-8)
      - [*Update Function*](#update-function-7)
  - [LaserScanSpeckleFilter](#laserscanspecklefilter)
    - [Parameters](#parameters-12)
    - [Pseudocode](#pseudocode-9)
      - [*Update Function*](#update-function-8)
  - [pass_through filter](#pass_through-filter)
- [Notes](#notes)
- [References](#references)

# Introduction

Today, I want to help illuminate the filtering algoritms used in the laser_filters package through pseudocode, demonstrating their use cases and through visual diagrams. I also hope to fill in some gaps that have been left by existing documentation on ROS Wiki.

This is written especially for ROS Beginners(and might not understand C++ as well), as well as for those who are simply interested but too busy to read the source code.
By the way, if you [haven't read the documentation](http://wiki.ros.org/laser_filters), it will be immensely useful. Although it hasn't been updated to include new filters from the latest branch.

When I was starting out in ROS, I wondered how ROS developers wrote their own algorithms for filtering lidar noise, or getting the specific data points they desire. And the open source community already had an answer to that, the [ros-perception laser_filters package](https://github.com/ros-perception/laser_filters) is already an essential tool that relieves every other ROS developer out there from having to develop their own lidar processing code. 

However, it's a pity that for such an awesome piece of work, there is insufficient documentation and material out there explaining how these algorithms work. Of course most experienced developers would say, "Read the source code! It's really obvious." But we fail to see that engineers can be too busy to deep dive into the code even if the code is well organized and written (which it just so happens often). 

# Types of laser filters
Laser scans encode 2 primary types of data, the range of each laser beam and it's reflected intensity. By comparing the ranges between different laser beams we can also remove "noisy" laser beams. We can also convert the laser scans to it's corresponding (x, y) position relative to the robot (or obtain their angular position) and filter out points in undesired locations, with the possibility to also invert this operation.
The types of manipulation performed on the laser scan ranges could be setting their value to NaN, Infinity or a user-defined value.

In the following pseudocode, the **Update Function** is the main function that manipulates laser scans within each laser filter.

## LaserScanIntensityFilter
This filter removes all measurements from the sensor_msgs/LaserScan which have an intensity greater than upper_threshold or less than lower_threshold. These points are "removed" by setting the corresponding range value to range_max + 1, which is assumed to be an error case.

Take note that setting **filter_override_intensity** to true will assign all "removed" points the intensity of 0.0, and all non-"removed" points to have an intensity value of 1.0 .

### Use Cases
The intensity filter could be used to extract laser scan points within a range of intensities especially for applications in robot docking, or even SLAM. Some docking stations possess reflective markers which the lidar will perceive as high intensity scan points, which when extracted and processed could provide an indication of the angular and linear offset of the robot relative to the docking station.


### Parameters
```yaml
- name: intensity
  type: laser_filters/LaserScanIntensityFilter
  params:
    lower_threshold: 8000 
    upper_threshold: 100000
    invert: false
    filter_override_range: true #If true, set all "removed" points to NaN
    filter_override_intensity: false #If true, set all "removed" points to have intensity of 0.0, and set all non-"removed" points to have intensity of 1.0
```

### Pseudocode

#### *Update Function*
```python 
FUNCTION Update(scan)

  FOR each i IN RANGE (0, scan.ranges.size) # For each point in laser scan
    #Intensity and range are modified in place here
    intensity = scan.intensities[i]
    range = scan.ranges[i]
    
    #Set filter as true if intensity falls outside the user-defined threshold
    filter = intensity <= config.LOWER_THRESHOLD OR intensity >= config.UPPER_THRESHOLD
    IF (config.INVERT) #Invert "removal" of points
      filter = NOT filter
    ENDIF

    IF filter
      IF config.filter_override_range
        range = NaN #Set current range value as NaN
      ENDIF

      IF config.filter_override_intensity
        intensity = 0.0 
      ENDIF 
    ELSE 
      IF config.filter_override_intensity
        intensity = 1.0 
      ENDIF
    ENDIF   
  ENDFOR

ENDFUNCTION
```

## LaserScanRangeFilter
This filter removes all measurements from the sensor_msgs/LaserScan which are greater than upper_threshold or less than lower_threshold. 

### Use Cases
The range filter could be used to "remove" laser scan points that are below or beyond the usable lidar range (usually defined in the technical datasheet of the lidar). This is important to prevent erroneous readings from being used in the marking/clearing of obstacles from the costmap of the navigation stack.

### Parameters
```yaml
- name: range_filter
  type: laser_filters/LaserScanRangeFilter
  params:
    use_message_range_limits: false   # if not specified defaults to false
    lower_threshold: 0.075            # if not specified defaults to 0.0
    upper_threshold: 24.0             # if not specified defaults to 100000.0
    lower_replacement_value: .inf     # if not specified defaults to NaN
    upper_replacement_value: .inf     # if not specified defaults to NaN
```

### Pseudocode

#### *Update Function*
```python 
FUNCTION Update(scan)

  FOR each i IN RANGE (0, scan.ranges.size) # For each point in laser scan
    intensity = scan.intensities[i]
    range = scan.ranges[i]
    

  ENDFOR

ENDFUNCTION
```

## LaserScanAngularBoundsFilter
Removes points in a [sensor_msgs/LaserScan](http://docs.ros.org/en/api/sensor_msgs/html/msg/LaserScan.html) OUTSIDE of certain angular bounds by changing the minimum and maximum angle. All angular units are in radians.

### Use Cases
The angular bounds filter could be used to remove scan points that might be intersecting with the physical robot base, which are mistaken as obstacles by the navigation stack.

### Parameters
```yaml
- name: angular_bounds
  type: laser_filters/LaserScanAngularBoundsFilter
  params:
    lower_angle: -1.571 
    upper_angle: 1.571 
```

## LaserScanAngularBoundsFilterInPlace
Works in a similar way to LaserScanAngularBoundsFilter. It removes points in a sensor_msgs/LaserScan INSIDE certain angular bounds by changing the minimum and maximum angle. 

### Parameters
```yaml
- name: angular_bounds
  type: laser_filters/LaserScanAngularBoundsFilterInPlace
  params:
    lower_angle: -1.571 
    upper_angle: 1.571 
```

## LaserScanSectorFilter
Removes laser scan points within a sector. This would be equivalent to using both LaserScanAngularBoundsFilter and LaserScanRangeFilter together.

### Parameters
```yaml
- name: scan_filter
  type: laser_filters/LaserScanSectorFilter
  params:
    angle_min: 2.54                     # if not specified defaults to 0.0
    angle_max: -2.54                    # if not specified defaults to 0.0
    range_min: 0.2                      # if not specified defaults to 0.0
    range_max: 2.0                      # if not specified defaults to 100000.0
    clear_inside: true                  # if not specified defaults to true
    invert: false                       # (!clear_inside) if not specified defaults to false
```

## InterpolationFilter
For any invalid scan range outside of the minimum and maximum scan range, the interpolation comes up with a range value which is an interpolation between the surrounding valid values (within the accepted scan range).

### Use Cases
The interpolation filter is sort of a double-edged sword, it could reconstruct scan messages from invalid points. However, it can also introduce unnecessary noise into the scan messages especially if the difference between the surrounding valid range values are relatively large, which is likely for nearby obstacles against other obstacles further away. The navigation stack could mistake this interpolated point as an obstacle when there is none.
I would imagine that it's best used in an environment small enough to be within the maximum range of the lidar.

### Parameters
```yaml
- name: interpolation
  type: laser_filters/InterpolationFilter
```

### Pseudocode

#### *Update Function*
```python 
FUNCTION Update(scan_in, scan_out)

  previous_valid_range = scan_in.range_max - 0.01
  next_valid_range = scan_in.range_max - 0.01

  scan_out = scan_in

  WHILE i < scan_in.ranges.size #Iterate through all the scan points 

    #If scan is outside accepted range
    IF scan_out.ranges[i] <= scan_in.range_min OR scan_out.ranges[i] >= scan_in.range_max 
      #Find the next valid range reading
      j = i + 1
      start_idx = i
      end_idx = i
      WHILE (j < scan_in.ranges.size)

        #If scan is outside accepted range
        IF (scan_out.ranges[j] <= scan_in.range_min OR scan_out.ranges[j] >= scan_in.range_max) 
          #Set the end index of the invalid ranges
          end_idx = j
        ELSE
          #Break out of while loop when a valid range is found
          next_valid_range = scan_out.ranges[j]
          BREAK
        ENDIF
        
        j++
      ENDWHILE

      #Take average of 2 valid range values
      average_range = (previous_valid_range + next_valid_range) / 2.0

      #Assign averaged value to all invalid ranges between the valid values
      FOR k in range(start_index, end_index)
        scan_out.ranges[k] = average_range
      ENDFOR

      #Update previous valid range reading
      previous_valid_range = next_valid_range
      i = j + 1
    
    ELSE
      previous_valid_range = scan_out.ranges[i]
      i++

    ENDIF

  ENDWHILE

ENDFUNCTION
```

## LaserScanFootprintFilter
Removes laser scan points within a prescribed radius of the robot footprint.

### Use Cases
The range filter could be used to "remove" laser scan points within the physical robot footprint, especially if the lidar scan points intersect with the robot chassis.

### Parameters
```yaml
- name: footprint_filter
  type: laser_filters/LaserScanFootprintFilter
  params:
    inscribed_radius: 0.75
```

### Pseudocode

#### *Update Function*
```python 
FUNCTION Update(scan)

  FOR each i IN RANGE (0, scan.ranges.size) # For each point in laser scan
    intensity = scan.intensities[i]
    range = scan.ranges[i]
    

  ENDFOR

ENDFUNCTION
```


## LaserScanBoxFilter
This filter removes points in a sensor_msgs/LaserScan inside of a 3 dimensional cartesian box.
These points are "removed" by setting the corresponding range value to NaN which is assumed to be an error case.

### Use Cases
The range filter could be used to "remove" laser scan points within the physical robot footprint, especially if the lidar scan points intersect with the robot chassis.

### Parameters
```yaml
- name: box_filter
  type: laser_filters/LaserScanBoxFilter
  params:
    box_frame: base_link
    max_x: 0.5
    max_y: 0.5
    max_z: 0.5
    min_x: -0.5
    min_y: -0.5
    min_z: -0.5
    invert: false #sets either the points inside or outside the defined box to be NaN
``` 

### Pseudocode

#### *Update Function*
```python 
FUNCTION Update(scan)

  FOR each i IN RANGE (0, scan.ranges.size) # For each point in laser scan
    intensity = scan.intensities[i]
    range = scan.ranges[i]
    

  ENDFOR

ENDFUNCTION
```

## LaserScanPolygonFilter
"Removes" laser scan points within a user-defined polygon using a method similar to LaserScanBoxFilter.
The example parameters below define a five sided polygon.

### Use Cases
The range filter could be used to "remove" laser scan points within the physical robot footprint, especially if the lidar scan points intersect with the robot chassis.

### Parameters
```yaml
- name: polygon_filter
  type: laser_filters/LaserScanPolygonFilter
  params:
    polygon_frame: base_link
    polygon: [[-1.0, 0.5], [0, 1.0], [1.0, 0.5], [1.0, 0.5], [0.5, -1.0]]
    invert: false #sets either the points inside or outside the defined polygon to be NaN
```

### Pseudocode

#### *Update Function*
```python 
FUNCTION Update(scan)

  FOR each i IN RANGE (0, scan.ranges.size) # For each point in laser scan
    intensity = scan.intensities[i]
    range = scan.ranges[i]
    

  ENDFOR

ENDFUNCTION
```

## LaserScanMaskFilter
Removes scan points by specifying their indexes.

### Parameters
```yaml
- name: mask_filter
  type: laser_filters/LaserScanMaskFilter
  params:
    masks:
      <laser1_frameid>:
      - 100
      - 101
      - 102
      - 103
      - 104
      - 1000
      - 1001
      - 1002
      <laser1_frameid>:
      - [200, 201, 203]
      - ...
```

### Pseudocode
```
FUNCTION Update(scan)
  mask = masks["laser_frameid"]

  FOR idx IN mask
      IF idx > idx_array.length: 
          CONTINUE
      ENDIF
      scan.ranges[idx] = NaN
  ENDFOR
```

It would be nice to be able to state a range, rather than having to explicitly write each index.

## ScanShadowsFilter
Background: “Laser scans sometimes hit objects at a grazing angle, resulting in “split pixels” or “shadows”. These laser hits show up in the scan as single points in the middle of space, part-way between the object they grazed and a background object. For many tasks such as object model fitting, these points are simply noise and should be removed using the scan shadows filter. [2]

### Parameters

Example config parameters taken from [here](https://github.com/uos/calvin_robot/blob/93b506a10c6fb8000517ebd09c9f7041e2ab9cbf/calvin_bringup/config/calvin_lms200_self_filter.yaml)
```yaml
- name: shadows
  type: laser_filters/ScanShadowsFilter
  params:
    min_angle: 10    # Minimum perpendicular angle (Value range: 0 to 90 degrees)
    max_angle: 170  # Maximum perpendicular angle (Value range: 90 to 180 degrees)
    neighbors: 1      # No. of neighbouring points to remove
    window: 1         # Window of measurements to check 
```

### Pseudocode

#### *Update Function*
```python
FUNCTION Update(scan)

  FOR EACH i IN RANGE (0, scan.size) # For each point in laser scan
    FOR y IN RANGE( -config.WINDOW, config.WINDOW) 
      IF (y == 0)
        CONTINUE
      ENDIF

      range_a = scan.ranges[i]
      range_b = scan.ranges[i + y]

      angle = angle between rays scan.ranges[i] and scan.ranges[i + y]
      sin_alpha = sin(angle)
      cos_alpha = cos(angle)
      
      IF isShadow(range_a, range_b, sin_alpha, cos_alpha) 
        FOR index IN RANGE (i - config.NEIGHBOURS, i + config.NEIGHBOURS) #For each neighbouring point
          IF scan.ranges[i] < scan.ranges[index] 
            scan[index] = NaN
          ENDIF
        ENDFOR
        IF remove_shadow_start_point_ 
          scan[i] = NaN
        ENDIF
      ENDIF
      
    ENDFOR
  ENDFOR
  
ENDFUNCTION
```

#### *isShadow Function*
```python
# range_a and range_b are the absolute scan distances of P1 and P2 respectively
FUNCTION isShadow(range_a, range_b, sin_alpha, cos_alpha)
  perpendicular_y = range_b * sin_alpha
  perpendicular_x = range_a - range_b * cos_alpha
  perpendicular_tan = ABS(perpendicular_y) / perpendicular_x

  IF (perpendicular_tan > 0)
    IF (perpendicular_tan < tan(config.MIN_ANGLE)) 
      return TRUE
    ENDIF
  ELSE
    IF (perpendicular_tan > tan(config.MAX_ANGLE)) 
      return TRUE
    ENDIF
  ENDIF
  
  return FALSE

ENDFUNCTION
```

https://answers.ros.org/question/239454/whats-the-rational-behind-laser_filtersscanshadowsfilter/


## ScanBlobFilter
The scan blob filter is useful for extracting “blob objects” which means that racks, human legs can be detected effectively (at close distances).
<!-- Ideally, it should be used after being filtered with shadow filters as the filter works by identifying group of points separated by nan values -->
The minimum points and maximum radius must be tuned so that we only obtain blobs of our desired objects and exclude others. For example, you want to only extract chair legs and not human legs, you would specify a smaller *max_radius* and smaller *min_points*.

<img src="../public/assets/2022-01-09-the_ultimate_guide_to_laser_filters/scan-blob-visualization.png" alt="scan-blob-visualization" width="400"/>

__Figure X: [Scan Blob Visualization](https://github.com/ros-perception/laser_filters/pull/80) __

### Parameters
```yaml
- name: blob_filter
  type: laser_filters/ScanBlobFilter
  params:
    max_radius: 0.25 # maximum radius to be considered as blob object
    min_points: 4 # min scan points to be considered as blob object
```

### Pseudocode

#### *Update Function*
```python 
FUNCTION Update(scan)

  FOR each i IN RANGE (0, scan.ranges.size) # For each point in laser scan
    intensity = scan.intensities[i]
    range = scan.ranges[i]
    

  ENDFOR

ENDFUNCTION
```

## LaserScanSpeckleFilter
This filter removes speckle points in a laser scan by comparing neighboring points. 
The term speckle refers to a random granular pattern which can be observed e.g. when a highly coherent light beam (e.g. from a laser) is diffusely reflected at a surface with a complicated (rough) structure, such as a piece of paper, white paint, a display screen, or a metallic surface.

### Parameters
```yaml
- name: speckle_filter
  type: laser_filters/LaserScanSpeckleFilter
  params:
    # Select which filter type to use.
    # 0: Range based filtering (distance between consecutive points)
    # 1: Euclidean filtering based on radius outlier search
    filter_type: 0

    # Only ranges smaller than this range are taken into account
    max_range: 2.0

    # filter_type[0] (Distance): max distance between consecutive points
    # filter_type[1] (RadiusOutlier): max distance between points
    max_range_difference: 0.1

    # filter_type[0] (Distance): Number of consecutive ranges that will be tested for max_distance
    # filter_type[1] (RadiusOutlier): Minimum number of neighbors
    filter_window: 2
```

### Pseudocode

#### *Update Function*
```python 
FUNCTION Update(scan)

  FOR each i IN RANGE (0, scan.ranges.size) # For each point in laser scan
    intensity = scan.intensities[i]
    range = scan.ranges[i]
    

  ENDFOR

ENDFUNCTION
```
## pass_through filter
There is no documentation available nor does they seem to be an implementation of it in the source code, save for an empty example in "examples/pass_through_example.xml"

# Notes

# References
[1] [ROS Wiki: Laser Filters](http://wiki.ros.org/laser_filters)
[2] [ROS Wiki: Laser filtering using the filter nodes](http://wiki.ros.org/laser_filters/Tutorials/Laser%20filtering%20using%20the%20filter%20nodes#:~:text=Laser%20scans%20sometimes%20hit%20objects,grazed%20and%20a%20background%20object.)