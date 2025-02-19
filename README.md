
[![Build Status](https://github.com/NanoComp/imageruler/workflows/CI/badge.svg)](https://github.com/NanoComp/imageruler/actions)

Imageruler is a free Python program to compute minimum length scales of 1d or 2d binary images, typically given by topology optimization. The method for 2d cases is based on morphological transformations [1,2], which are realized by the OpenCV library [3].

See the [provided example notebook](notebooks/examples.ipynb) and the code documentation.

The method for estimating the minimum length scale of solid regions in a 2d binary image is outlined as follows.
1. Binarize the 2d array that represents the image, so that each element is a Boolean value. Let $\rho$ denote this Boolean array. For ease of description, elements with values `True` and `False` are referred to as solid and void, respectively.
2. For a kernel given by a disc with the diameter $d$, compute morphological opening $\mathcal{O}_d(\rho)$ and obtain the difference $\mathcal{O}_d(\rho) \oplus \rho$, where $\oplus$ denotes exclusive or.
3. Evaluate whether $\mathcal{O}_d(\rho) \oplus \rho$ contains a solid pixel that overlaps an interior pixel of solid regions in $\rho$. If no, $d$ is less than the minimum length scale of solid regions; if yes, $d$ is not less than the minimum length scale of solid regions. Here the interior of solid regions in $\rho$ is given by morphological erosion using a small kernel like a cross with the size $3\times3$.
4. Use binary search and repeat Steps 2 and 3 to seek the minimum $d$ at which the answer in Step 3 is yes. This value of $d$ is considered as the minimum length scale of solid regions.

To estimate the minimum length scale of void regions in a 2d binary image, the binary image is inverted after binarization, i.e., $\rho \rightarrow \neg \rho$ so that solid and void regions are interchanged. Subsequent procedures are the same as described above. This approach is equivalent to computing $\mathcal{C}_d(\rho) \oplus \rho$ in Step 2 and test the overlap with interior pixels of void regions in $\rho$ in Step 3. Here $\mathcal{C}_d(\rho)$ denotes morphological closing.

The minimum length scale of $\rho$ is the minimum between the minimum length scales of solid and void regions. To obtain this minimum, one can compute the minimum length scales of solid and void regions separately and take the minimum. A more efficient approach is to compute $\mathcal{O}_d(\rho) \oplus \mathcal{C}_d(\rho)$ in Step 2 and test the overlap with the union of interior pixels of solid and void regions in $\rho$ in Step 3.

The accuracy of the minimum lengthscale is limited by the finite resolution of the image: it is obvious that lengthscales below the single-pixel level cannot be measured, but even few-pixel scales cannot always be distinguished from "staircased" discretizations of curved surfaces.  For example, a 90° sharp corner corresponds to a lengthscale (radius of curvature) of *zero* at infinite resolution, but at finite resolution it cannot be distinguished from a discretization of a rounded corner with a radius of a few pixels, so `imageruler` will return a lengthscale proportional to (but greater than) the pixel size in that case.  In general, the measured lengthscale should be viewed as having an "error bar" on the few-pixel level.

For a 1d binary image, the code simply searches for the minimum lengths among all solid or void segments.

References  
[1] Linus Hägg and Eddie Wadbro, On minimum length scale control in density based topology optimization, Struct. Multidisc Optim. 58(3), 1015–1032 (2018).  
[2] Rafael C. Gonzalez and Richard E. Woods, Digital Image Processing (Fourth Edition), Chapter 9 (Pearson, 2017).  
[3] Gary Bradski. The OpenCV Library. Dr. Dobb’s Journal of Software Tools (2000).
