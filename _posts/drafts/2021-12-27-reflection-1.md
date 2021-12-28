# What should I focus on?

It's been almost 2 years since I started software engineering in the robotics field and since then, my knowledge has expanded rapidly too. There are so many things that pique my interest but unfortunately we don't have all the time in the world to pursue them all.

There will be a point where we are forced to choose what to focus on or end up being a generalist or jack-of-all-trades. 

There are a few things I could choose to focus on
1. Robotics
   1. Controls
      1. Local Planners
   2. Navigation
      1. Path Planning
   3. Perception
      1. Visual SLAM
         1. BOW
2. Machine Learning
   1. Field
      1. Computer Vision
      2. Speech
   2. Machine Learning
      1. Supervised
      2. Unsupervised
      3. Reinforcement
3. Web Back-End
   1. Networking 
      1. Sockets
      2. Concurrency
   2. NodeJS
   3. Databases
      1. MongoDB
      2. SQL
4. Web Front-End
   1. ReactJS





Articles I want to write:
1. Dynamic TF tool
2. Tutorial for ros2dlib
3. Visual SLAM
4. Implementing AMCL from scratch
5. Implement Extended Kalman Filter

6. Golang in AOC
- Interfaces (Composition over Inheritence)
- Concurrency
- Creating custom libraries
- Unit tests
- goroutines

7. Origami flashers

8. Circle raster, Wu's line algorithm


Books to read:

1. Technical
   1. Explain the Cloud Like I’m 10
   2. Unwritten Laws of Engineering
   3. The Passionate Programmer: Creating a Remarkable Career in Software Development
2. Literature
   1. The Brothers Karamazov
   2. The Art of Charlie Chan Hock Chye
   3. Snow Leopard
   4. War and Peace


# Visual SLAM:

Current Implementations:
1. ORB-SLAM

# Install anaconda and ROS together

https://answers.ros.org/question/307757/how-to-get-ros-and-anaconda-to-play-well-together/

https://answers.ros.org/question/256886/conflict-anaconda-vs-ros-catking_pkg-not-found/

https://github.com/ros/ros/issues/149


```
conda install -c conda-forge -c robostack ros-noetic-desktop
```

# Conda cheatsheet

1. All packages installed
conda list

2.
conda init

3. Create virtual environment with dependencies
conda create -n my_env numpy

4. Install packages
conda install <pkg1> <pkg2> ...
conda remove 
conda update
conda search '*<packeg name>*'

# Jupyter notebooks
They are just JSON files with .ipynb extension

1. Convert notebook to another format
   1. jupyter nbconvert --to FORMAT mynotebook.ipynb
   2. Example
      1. jupyter nbconvert --to html mynotebook.ipynb
      2. jupyter nbconvert notebook.ipynb --to slides
         1. To run slideshow: jupyter nbconvert notebook.ipynb --to slides --post serve


## Errors

1. Conda command not found:
   export PATH="/Users/USERNAME/opt/anaconda/bin:$PATH"

