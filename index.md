# Startracker: Star Trails - Center Finder

```{eval-rst}
.. abstract::

   This document presents the analysis of the star trails center finder algorithm.
   This algorithm was used to determine the center of the TMA on the Startrackers fields.
   The analysis was performed on the data taken on the 9th, 17th, and 22nd of March 2023.

```

The algorithm demonstrated here is a simple method to find the center of the star trails. 
The method is based on finding reasonable large streaks. 
The center is then determined by fitting a circle to the points on the streaks.

The test case associated with this analysis is [SITCOM-1411](https://rubinobs.atlassian.net/browse/SITCOM-1411)

The full analysis notebook can be found here:
[strailCenter.nbviewer](https://nbviewer.org/github/estevesjh/ComScratchStuff/blob/summit2023/starTracker/streakFinder/starTrailCenter2.ipynb)

## Method Description

The algorithm is based on the following steps:
1. Find the streak in the starTracker fits images.
2. Select a sub-set of streaks with a large angular length (>10 deg).
3. Fit a circle to each streak.
4. Determine the star trail center by taking the average of step 3 results.

## Streak Finder
To find the streaks, we use the algorithm [ASTRiDE](https://github.com/dwkim78/ASTRiDE#4-how-to-use-astride).

```
# Import the ASTRiDE library.
from astride import Streak

# Read a fits image and create a Streak instance.
streak = Streak(fname, contour_threshold=1.5, area_cut=25, radius_dev_cut=0.5, remove_bkg='map')

# Detect streaks.
streak.detect()
```

We first find the image's prominent streak (1.5 times the background) without a length selection. 
The `radius_dev_cut` criteria ensure we do not select blobs.
The `area_cut` criteria ensure we do not select tiny streaks. 

## Streak Selection
The streak selection is based on the angular length of the streak.
Arcs with more considerable angular lengths are better at pinpointing the circle center. 

The angular length is defined as:

$$
\theta = \ell \times r \,, 
$$
where $\ell$ is the streak length and $r$ is the radius of the circle.

:::{note}
 **This method should be used with visual inspection.** An initial guess of the center of the trails is needed to compute the streak angular length. Results should be visually expected, a bad arc selection can delivery a bad circle center fit.
:::

:::{figure} /_static/arc_angle_distribution.png
:name: figs-arc-angle-distribution.

The angle distribution of the streaks found by the streak finder algorithm.
:::

## Circle Fitting

Once you have a set of points along the streak, you can fit a circle to the points.
The python module used here is [circle_fit](https://pypi.org/project/circle-fit/). 
This module have different sets of algorithms to fit a circle.
In particular, the Algebraic circle fit (`taubinSVD`) by G. Taubin proved the most robust.

Note that you can implement your circle-fitting algorithm.
However, this numerical problem is not trivial.
A good reference is [Chernov  & Lesort 2008](https://arxiv.org/abs/cs/0301001).

## Center Determination
We take the average solution of the circle fitting algorithm as the center of the star trails.
The selection based on angle length provides a very accurate center determination.
The center determination may differ if other selections are used, e.g., inner or outer trails.

:::{figure} /_static/star_trail_centroid_102.png
:name: fig-star-trail-centroid

Startracker strail image 102. The selected streaks are the colorful selection. The center of the star trails is marked with a black cross.
:::


## More Information
For more information on the algorithm and the analysis, please refer to the analysis repo:
[streakFinderFolder](https://github.com/estevesjh/ComScratchStuff/tree/summit2023/starTracker/streakFinder)

Slides with the results for three different starTracker images:
[slides](https://drive.google.com/file/d/1bo2l10HO1lA3KoAJWgcUbpMf44smyGt7/view?usp=sharing)

The notebook used to develop the method is:
[devNotebook](https://nbviewer.org/github/estevesjh/ComScratchStuff/blob/summit2023/starTracker/streakFinder/starTrailCenter.ipynb)