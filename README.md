# SBC Background Subtraction

This project reduce the background of the HST Solar Blind Channel (SBC) Multi Anode Microchannel Array (MAMA) image.

The main sources of the background are sky background and dark current.
The sky background are scattered lights of the astronomical object, assumed to be uniformed in space. 
The dark current is the thermal noise of the instrument, which has a spatial pattern that the background level is higher at the center region of the image.

![plot](./figure/dark_flt.png)

* dark current is higher at the center region.  The horizontal lines are bad pixels with pixel value = 0 

This code is a modified version of [Sourabh's code](https://github.com/sourabhsc/dark_subtraction_sbc). To fit my purpose, the annulus of the background do not have to centered at the center of the image. (The object is not in the center of the image.) 

In this code, there are two ways to remove the background: 
## 1. Sky subtraction only: ##
The temperature of the detector is recorded in the header.  If the temperature is low enough, we can neglect the dark current and simply subtract a constant value throughout the image. The cutoff temperature is around 22 degree.  When temperature <22,  we produce annuli from r_in to r_out with width of 1, where r_in = rf and r_out =rf+sky_apt_size. We calculate the mean of the pixel value inside the annulus, and take the median of all mean as the sky background. 

note1: This calculation slightly prefer the pixels closer to the object. If the dark current is truely negligible, this is a monor concern. 
note2: Taking median of the means of all annulus help us avoid the unidentified hot pixels.
note3: Since most of the pixels have value = 0, directly taking median of all pixel values between r_in and r_out will result to zero. 

## 2. Sky + dark subtraction: ##
If the science exposure temperature > temp_cutoff (= 22, empirically), we will subtract the background by : 

sci_fd(i, j) = sci_flt(i, j) - A * ts/td * drk_flt(i, j) - K

where sci_fd(i, j) is the product image, sci_flt(i, j) is the original science flat image. 
A is the scale paramter (scalar) for the dark current. 
drk_flt(i, j) is the dark flat image, where you can download the raw image and reduce using acstool.[1] 
ts, td are the exposure time of the science exposure and dark image, respectively.  (td = 1000 seconds)
K is the constant sky background. 

to determine the parameter A and sky, we find the minimum of the residue : 

residue = Sum[ ((sci - A * ts/td * drk - K)^2)]

through out the paramter space of A~[0 : 2] and K~[0 : 0.02].  
The range of parameter spaces of A and K are decided empirically again. It is important to check if the values we pick hit the boundary or not. 


note1: In principle, having a more delicate grid of A and K will give you a better result.  However this is not really efficient, and the result looks similar. 
note2: Since most of the pixels in the flat images have value =0, we mask the galaxy, bin(reshape) the images by every 25*25 pixels, and ignore the side of the images (1024 x 1024  --> 40*40,  with the edge 24 pixels being chopped). The we conisder the residue of the entire frame. 


[1] to download dark images: [HST calibration](https://stsci.edu/hst/instrumentation/acs/calibration)

## Steps of using this code ##
0. install numpy, astropy, pathlib, etc.  

1. put the fits file (rootname_flt.fits) you want to reduce in [PATH_TO_DATA], and assign flt_dir='[PATH_TO_DATA]'

2. put the dark_flt images that corresponds to the data in [PATH_TO_DRK], and assign flt_dir='[PATH_TO_DRK]'

3. Some paramters that's easy to adjust includes temperature cutoff "temp_cutoff = 22", size of the galaxy mask " rf = 150 " (which is r_in), size of the annulus for the sky subtraction "sky_apt_size = 100". 

4. The result will be in [PATH_TO_DATA]/product/, named as rootname_fd.fits. 

5. An analysis figure will be created in [PATH_TO_DATA]/product/plot/. 

![plot](./figure/25.png)
*  The plot created in the code. All three frames are a 10-pixel-Gaussian smoothed images.  THe first row is the flt image, the second row is the reduced image, and the third row is the ratio (reduced/flat).   The black circle marks where the galaxy is masked.    

* The process of making plot is extremely slow.  I'm sorry.

