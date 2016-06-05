# Diffusion from a moving source

_(this was a side project I did during my PhD)_

![Neutrophil chasing bacteria](https://github.com/igor25/moving_source_diffusion/blob/master/videos/neutrophil_chasing_bacteria.gif)
![Large molecule diffusion](https://github.com/igor25/moving_source_diffusion/blob/master/videos/video_d1e-11_xystep3_g1.gif)
![Small molecule diffusion](https://github.com/igor25/moving_source_diffusion/blob/master/videos/video_d2e-10_xystep3_g0.1.gif)

Many living cells eat and digest other living cells. One example is shown in the
video below taken from a 16 mm movie made in 1950s by the late prof. David Rogers
at Vanderbilt University. Here a white blood cell (largest thing moving around) is crawling among red blood cells (circular objects) and chasing a _Staphylococcus aureus_ "Staph" bacteria (little black dumbbell looking thing). Bacteria secretes chemicals which white blood cells can sense and use as a tracer to find and kill bacteria.

I was curious what is the best possible accuracy with which cells (such as white blood cells) can track chemical gradients generated by other cells under these dynamical conditions. This seems challenging since the detection of chemical signals is very noisy and that chemicals also diffuse and spread in the environment. To put it into perspective, if you manually introduce a small patch of chemical the size of bacteria in the video, in one second it would spread over the area of a white blood cell (~ 20 &mu;m by 10 &mu;m).

## Modeling: COMSOL Multiphysics / MATLAB

I simulated the bacteria as a point source that is continuously producing a chemical which is free to diffuse in a rectangular environment with periodic boundary conditions.

**MATLAB instructions**

* In the work I've done for this project so far, I only used MATLAB to generate a set of coordinates that we use to move the bacteria around. Looking back, better way would be to start "COMSOL with MATLAB", define a ``fem`` object and then do everything through MATLAB.

* Running the script ``generate_random_walk_matlab.m`` in MATLAB creates two tab delimited files containing two columns: first one is time and second one is the coordinate at that time given by the random walk model.

**COMSOL Multiphysics (3.5) instructions**

* Create two geometries ('geom's). The first one is the main
domain where the diffusion is simulated (a square, insulating boundary, diffusion and no
  convection and reaction terms under ``Physics`` / ``Subdomain Settings``) and a second one consisting of a single point.
* Define two functions (``Options`` / ``Functions``): `xcoord` and `ycoord` which are going to be used to move the second geom (point) in the coordinate system of the first geom. For both functions select ``Interpolation`` and ``Use data from: File``, then for each select a text file generated by the MATLAB script.
* Now we can connect the two geoms. From ``Options`` select ``Extrusion coupling variables``. In the ``Source`` tab select the only subdomain in geom 1 (rectangle), with name `c`, expression `c` and general transformation ``x: x``, ``y: y``. In the ``Destination`` tab set Geometry: \*Geom2, Variable: c, Level \*Boundary, check "Use selected boundaries as destination" and use Destination transformation: ``x: xcoord(t)*10^-6``, ``y: ycoord(t)*10^-6``.
* Press F11 to open ``Solver parameters``. Under Analysis types, select "Diffusion" and "Transient" for both and select a ``Time dependent`` solver. Under General tab select Times: 0:0.5:1, Relative tolerance: 0.1, Absolute tolerance: 0.01 and use a linear system solver Direct (UMFPACK).
* Generating output. Press Solve to solve the model. Export the data using ``File``, ``Export``, ``Postprocessing data``. On ``Subdomain`` tab select the number of points in x and y grid to calculate the concentration in. On ``General`` tab, type file name and select ``Coordinates, data`` as a format of exported data. This file can be loaded into the R script.

**R instructions. Plotting results.**
