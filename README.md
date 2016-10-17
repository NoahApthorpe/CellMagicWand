# CellMagicWand
Python implementation of ImageJ Cell Magic Wand tool originally created by Theo Walker and provided at [https://www.maxplanckflorida.org/fitzpatricklab/software/cellMagicWand/](https://www.maxplanckflorida.org/fitzpatricklab/software/cellMagicWand/).

This implementation uses the same dynamic programming and edge following algorithm as the original, but operates on Python Numpy arrays.  This makes it easier to incorporate into automated image analysis pipelines. This implementation is more robust to seed point selection and can also optionally operate on 3D images using information from all z slices. 

## Usage
There are 3 primary functions in `cell_magic_wand.py`, all of which have the same calling conventions. 

* `cell_magic_wand_single_point(image, center, min_radius, max_radius, roughness, zoom_factor)` is analogous to the original ImageJ implementation 

* `cell_magic_wand(image, center, min_radius, max_radius, roughness, zoom_factor, center_range)` runs `cell_magic_wand_single_point` with 9 seed points located in a square around and including the given center. A point is included in the final ROI if it is in the majority of ROIs found from each of the 9 seed points. 

* `cell_magic_wand_3d(image, center, min_radius, max_radius, roughness, zoom_factor, center_range, z_step)` runs `cell_magic_wand` on multiple z slices in a 3D image. A point is included in the final ROI if it is in the majority of ROIs found from the z slices. Note that this function can take considerably longer to run than above 2D functions depending on the z depth and the `z_step` argument.

#### Arguments

* `image` A 2D or 3D image in Numpy array format. A 3D `image` argument to `cell_magic_wand_3D` should have its z dimension as the 0th dimension.  For TIFF images, import `tifffile` (or install using `pip`) and run `image = tifffile.load([image_filename])` to get an image variable with the right format.

* `center` A seed point located as close to the center of the cell as possible. Should be a 2-element tuple of integers `(y,x)` (pixels down, pixels right). <em>If you are having problems, make sure you have the right  coordinate system.</em>

* `min_radius` The minimum distance from the seed point that the edge detection algorithm will search. Increasing `min_radius` helps the tool avoid finding organelles and other boundaries inside the cell.

* `max_radius` The maximum distance from the seed point the edge detection algorithm will search.  Decreasing `max_radius` helps keep the tool from circling multiple cells.

* `roughness` Controls the roughness of the edge of the detected ROI.  Analogous to the roughness parameter of the ImageJ Cell Magic Wand tool but not exactly equivalent. The tool samples points at `2 * pi * max_radius * roughness` angles around the center.  Increasing `roughness` increases runtime but can result in better detection.  The default value is 2 and values [1,10] are reasonable. 

* `zoom_factor` Factor by which the image is upsampled prior to edge detection. In most cases, the default value of 1 is appropriate.  

* `center_range` The distance to `center` of the additional 8 seed points used in the robust `cell_magic_wand` and `cell_magic_wand_3d` functions.  The default 2, so for center `(y,x)`, points `(y-2,x)`, `(y,x+2)`, etc. will be used. Set such that all points used will be inside your cells and near the center.  

* `z_step` The z stride used in `cell_magic_wand_3d`.  For an image with N z-slices, `floor(N/z_stride)` will be used. 

#### Return Values

All `cell_magic_wand` functions return a binary mask that is the same size as the input image with 1s inside the detected cell and 0s elsewhere.

`cell_magic_wand_single_point` also returns an array of points that are on the edge of the detected cell

## Tips
Even with multi-seed sampling, the `center` argument still considerably affects cell magic wand tool accuracy. 

If your image has a lot of contrast variability, preprocessing it with a median filter can improve cell magic wand tool results.  The easiest way is to use the `median_filter` function from `scipy.ndimage.filters`.


