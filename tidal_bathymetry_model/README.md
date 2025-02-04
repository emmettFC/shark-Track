# Tidal bathymetry model & review of applied methods 

![alt text](https://github.com/emmettFC/selected-projects/blob/master/tidal_bathymetry_model/assets_README/bathy_12.png)

 ## Part I: Review of applied methods for WV2 images of South Water Cay Marine Preserve, Belize 

### Introduction 
Estimating bathymetric models based on mutlispectral data is a well studied problem. Generally speaking models are either physical / theoretical or empirical. Physical models use known properties of light attenuation in water, substrate reflectance, and diffuse attenuation related to suspended concentrations of biogenic constituents—such as chlorophyll, suspended sediment or dissolved organic matter—to estimate depth from spectral values. Empirical models use known depth values to ‘train’ predictive functions of the relationship between spectral values and depth (5). Models of the latter category are typically hybrid physical-empirical models, and employ some theoretical model of bathymetry to make baseline predictions which are then corrected based on the observed error between depth estimates and ground truth. 
This repository describes two attempts at bathymetric estimation from remote sensing data: 1) an application of the Stumpf (2003) model to WV2 sattelite imagery of the South Cay Marine Preserve in Belize; 2) a ongoing project that intends to use a sequence of images at known points through the tidal cycle to parameterize physical models.  

### The South Cay Marine Preserve, Belize
I collaborated with a colleague of mine at UCSB on a project seeking to build a high resolution bathymetric model of the South Cay Marine Preserve in Belize. The purpose of this project was to use passive depth sounding data as labels to train the Stumpf and Lyzenga equations for bathymetric estimation. The spectral data for the region of interest are a set of images from the WV2 digital globe satelite. The image below (left) shows the coverage of the WV2 imagery in red, and the minimal bounding box for the passive depth sounding labels in blue. The righthand figure is a true color map from the WV2 image of the entire South Cay preserve, with the red showing the enitre region and the green boundary indicating a no-take zone: 

![alt text](https://github.com/emmettFC/selected-projects/blob/master/tidal_bathymetry_model/assets_README/regions-clean.png)

The labeled region is quite small compared to the overall scene, though the real challenge comes from the sparse coverage of labeled points within the smaller region. These labels are passively generated from a hand held depth sounding device onboard a research ship studying the patch reef ecosystem of the South Cay. The coverage within the region is pictured below on the left: 

![alt text](https://github.com/emmettFC/selected-projects/blob/master/tidal_bathymetry_model/assets_README/labels-intersect.png)

To generate bathymetric estimates based on these data, the depth sounding labels have to be intersected with the satellite raster data, such that the spectral values for each band at a given pixel are associated with the depth measurement at that point (pictured above in the right). Once the spectral data are corrected for atmospheric effects and sun glint, according to the specifications of the satelittle product, then the model coeficients can be calibrated using regression analysis. 

### Stumpf and Lyzenga bathymetric estimation models
Initially we hoped to compare the performance of two well known methods for using multispectral data to estimate bathymetric models: Stumpf (2003) and Lyzenga (1978). The general for of these equations is given below for Lyzenga (equation 1) and Stumpf (equation 2): 

![alt text](https://github.com/emmettFC/selected-projects/blob/master/tidal_bathymetry_model/assets_README/stumpf-lyzenga-1.png)

Both of these techniques take advantage of the physical nature of light’s attenuation in the water column, namely that shorter wavelength bands attenuate faster than longer wavelength bands. This relationship is therefore correlated with depth. The first equation, the Lyzenga method, was difficult to use given that it makes predictions based not only on the relationship bewtween band ratios and depth, but also the reflectance properties of different types of substrate (ex sand, seagrass, rock etc). Therefore, this method requires the specification of at least 5 empirical parameters / sets of labels. In equation 1 above, Li is the TOA radiance value for band i, Lj is the TOA radiance value for band j, Ki/Kj are the light attenuation coeficients for band i and j respectively. 

We applied the Stumpf ratio method for bathymetric estimation according to Ehes & Rooney (2015). Before running the regressions to generate calibration coeficients, we had to convert the raw digital numbers to top of atomosphere spectral radiance values. In order to do this, you need the bandwidth and absolute clibaration factors from the imaging product speficiations for each band. After the digital numbers are converted to TOA values, you can apply the Stumpf method given in equation 2 above to generate coeficients and asses model performance. Linear models were trained and run for two subsets of the raw data: 

    1)	For all data <= 30 meters 
    2)	For all data <= 15 meters

For the 30-meter subset, we found an absolute mean difference between the predicted and actual values of 4.15 meters, with R2 of .5411 and RMSE of 3.655 meters. For the 15-meter subset, we found an absolute mean difference of .98 meters, with R2 of .5859 and RMSE of 1.91 meters. The graph below shows the OLS best-fit lines for each implementation: 

![alt text](https://github.com/emmettFC/selected-projects/blob/master/tidal_bathymetry_model/assets_README/regression-plot-final.png)

The 15 meter subset performs pretty well considering the simplicity of the model applied. It is apparent that some nonlinear function of the data would perform better given the output shown above. Utlimately though, the model trained on the labeled subset of data can be extrapolated to the entire region for depth < 15 meters and a bathymetric model can be generated. The model produced by this analysis is pictured below: 

![alt text](https://github.com/emmettFC/selected-projects/blob/master/tidal_bathymetry_model/assets_README/bathymetric-map.png)

## Part II: Proposal for UAV method of bathymatric modeling with sequential tidal correction

### Introduction
Models based on empirically calibrated coefficients outperform theoretical models in the literature (5). This is because of the fact that the underlying coastal and fluvial scenes have very different optical properties. There are four main sources of heterogeneity that prevent physical models from making generically accurate predictions: 

    1) surface turbulence and white water
    2) optical properties of the water column / variability in diffusive attenuation 
    3) surface reflectance and sun-glint
    4) substrate reflectance / albedo 

In this section I propose a method of developing empirical coefficients to correct physical depth estimates by flying consecutive flights with a UAV at regular 1 hour intervals through the course of a semidiurnal tidal cycle. Given appropriately short flight durations, I hypothesize that the known changes in depth corresponding to the high-low tidal cycle can be compared with estimated depths in order to empirically calibrate a physical model without in situ depth labels. This hypothesis is supported by the fact that known functional relationships exist between the associated physical variables and spectral radiance: spectral radiance is an exponential function of depth and diffuse attenuation, and a linear function of substrate reflectance (17)(18). This section has three main components:

    1) description of the experimental design and some of the problems encountered when conduting feild experiments
    2) review of the uses and methods for unsupervised image preprocessing in bathymetric estimation
    3) proposal for multi-tidal correction & steps towards model derivation

### Experimental design and preliminary feild work
In order to collect the data for the model of tidal depth correction described above, I flew a drone over the region of interest at regular 1-hour intervals through the course of a 6 hour tidal cycle. Flights were made with a DJI phantom pro drone with an RGB-NIR camera, and controlled using flight plans set in the Ground Station Pro App. The flights were made at an elevation of 270 feet, with an overhead angle of 0 degrees. Images were taken continuously through the flight duration, as opposed to a hover and shoot flight, given the battery restrictions on the DJI Phantom Pro (flight plan and drone pictured below): 

![alt text](https://github.com/emmettFC/selected-projects/blob/master/tidal_bathymetry_model/assets_README/flight-plan-and-drone.png)

My first attempt at doing this experiment demonstrated that ground control points (GCPs) were needed in order to provide tie points for the generation of an image mosaic. This was clear from the fact that photoscan was only able to incorporate a third of the raw images into the image mosaic, and nearly all images with no un-submerged points were excluded. In order to produce a set of image data with sufficient over-water coverage for depth analysis, I made GCPs for the region that mimicked the calibration targets typically used in Drone Photogrammetry experiments. Given that the GCPs were intended to provide physical context for the SIFT (scale invariant feature transform) algorithm, they needed to be 1) completely stationary and 2) sufficiently far from shore. The GCPs could not be attached to buoys because slack in the mooring line due to falling tide would cause them to move substantially in the direction of the current. Therefore it was necessary to build large wooden steaks that could be driven into the sand / peat and attach the GCPs to the top (pictured bellow): 

![alt text](https://github.com/emmettFC/selected-projects/blob/master/tidal_bathymetry_model/assets_README/gcp-image.png)

The ground control points worked very well, and provided enough tie points for the photoscan program to compile an image mosaic of the region. The design of the GCPs was adaquate and they did not move through periods where the current was at its strongest. The impotance of the GCPs in generating the image mosiac is apparent in the image mosaic shown below, given that they determine the outermost edge of the region that was generated by photoscan: 

![alt text](https://github.com/emmettFC/selected-projects/blob/master/tidal_bathymetry_model/assets_README/bathy_12.png)

The data collected were ultimately not viable for the indended bathymetric analysis. This for two main reasons: 1) The flight duration was 20 minutes, which at peak tidal current velocity has a large impact on the time-depth reltationship that is meant to be captured, and 2) The NIR images were not captured due to a connection error between the NIR camera and the drone, and the NIR is needed to de-glint the images which have very significant sun-glint. 

For the next iteration of the data collection process, we are going to improve the mounting rig for the NIR camera and change the flight plan to a thin rectange perpedicular to the coastline. The latter change will reduce the flight time which will allow for a more precise time-depth snapshot of the region, and give a transect with more variable depth to allow for a more robust test of the model. Finally, the ground control points will need to be modified to accomodate the new transect. There will be a larger proportion of completely over water images, and so the calibration targets will need to be more prominent features in the images taken at 270 feet. To to this the marking will be changed from a lattice pattern to a checker pattern, the latter of which is more visible in the overhead images as determined through this experiment. Further, the targets themselves will need to be larger and the steaks will need to be longer in order to remain stationary and unsubmerged in deeper water. These proposed changes are pictured below: 

![alt text](https://github.com/emmettFC/selected-projects/blob/master/tidal_bathymetry_model/assets_README/bathy_13.png)

### Unsupersived preprocessing of UAV images for land mask generation: 
Though the images were not viable for application of the bathymetric model, they could still be used to test and develop preprocessing methods required for analysis. For example, the Lyzenga method requires that: 

    1) pixels with optically deep water are labeled to calibrate volume scattering effects 
    2) land and patch reef structures are masked from the image
    3) regions with white water due to surface turbulence are removed for interpolation
    4) pixels with identifiable substrate are labeled to calibrate for variable bottom albedo (18)

For bathymetric models based on UAV inputs, the labeling requirements are much more significant given the quantity of images used for analysis(2). I think that most or all of these feature identification pre-processing steps can be accomplished by either supervised or ideally unsupervised image segmentation and classification methods. 

The first and most accesible of these proprocessing steps is the generation of a land-mask to exclude the unsubmerged coastline. I attempted to apply several unsupervised image segmentation processes to accomplish this, both of which follow the same general three-step process involving: 

    1) threshold denoising  
    2) edge detection / segmentation
    3) image masking

One method that performed well was the sobel operator for edge detection. The algorithm is based on two 3x3 kernels which are used to approximate the x and y directional derivatives of the image, resulting in an approximation of the image gradient function (19). To make the operation more robust to noise, I first ran the image through Gaussian filtration. The output of this process is shown below, using the skimage implementation of both Gaussian filtration and the sobel operator in python: 

![alt text](https://github.com/emmettFC/selected-projects/blob/master/tidal_bathymetry_model/assets_README/image-process-1.png)

The sobel operator does a good job of representing the variable reflectance with the image gradient. The process works well for approximating a contour than can be used to mask the land, though does not capture the fine-scale gradient along the coast that will be most useful to model calibration. The final graph above can be used to generate contour / polygon approximations in the image, which we can then use either for masking or even substrate classification / estimating model parameters. I will address this below for the canny edge detection algorithm, which outperforms sobel in this case. 

Canny edge detection is another algorithm that finds edges in images through the use of intensity gradient approximation. However, unlike the sobel operator, the canny algorithm incorporates the Gaussian filtration step, and so it is not required as a preliminary (in fact, the gradient approximation step in the canny algorithm uses the same convolutional kernel approximation for the image gradients, which is likely why they outperformed relative to other processes) (11). After intensity gradients are approximated, fine scale edges are identified using non-maximum suppression and then reduced to an optimal set through hysterical thresholding (11). For this analysis I used the canny edge detection algorithm as implemented in openCv for python. The algorithm produced the following result: 

![alt text](https://github.com/emmettFC/selected-projects/blob/master/tidal_bathymetry_model/assets_README/canny-edge-output.png)

The output is similar to the output of the sobel operator after Gaussian filtration, which is essentially the same algorithm without subsequent identification of ‘thin-edges’. This turns out to be a critical difference for the granularity of coastal boarders. Consider the comparison below: 

![alt text](https://github.com/emmettFC/selected-projects/blob/master/tidal_bathymetry_model/assets_README/image-processing-3.png)

The canny edge detection algorithm does a better job at distinguishing the granular features of the coastline, which will be helpful going forward to do substrate segmentation. More basically though, the output from the canny process can be used to generate and image mask for the image mosaic and isolate the submerged and unsubmerged contours of the coastline: 

![alt text](https://github.com/emmettFC/selected-projects/blob/master/tidal_bathymetry_model/assets_README/canny-mask-final.png)

### Methods for unsupervised image segmentation / masking away from the coastline & substrate segmentation 

The above discussion is based on an analysis of the orthomosaic of the whole image set as produced by photoscan. Though when I re-run this analysis according to my new experimental design, I am going to collect data for a narrow region extending out into the water instead of along the coast. At low tide, this will include sections that are purely water, and sections that are mostly water with isolated un-submerged patch reef structures. When I applied the algorithms described above to images of from this kind of transect—which I collected in my first flight—the results were inadequate to create masks and isolate un-submerged patches. For example, consider the result from application of the sobel operator pictured below:

![alt text](https://github.com/emmettFC/selected-projects/blob/master/tidal_bathymetry_model/assets_README/sobel-raw-OW.png)

The poor performance of image gradient methods for over-water images may be because the effects of sun-glint are very significant (19). This is something I will have to experiment with going forward, and will require the inclusion of an NIR band in the imaging set-up on the UAV(17). While edge detection methods produced poor results for these images, I had very good success using a light-weight implementation of a super-pixel segmentation algorithm called SLIC (simple linear iterative clustering)(12). This algorithm is a gradient ascent algorithm based on the development of local k-means super-pixels(12). ScikitImage offers a very efficient version of this algorithm for python. The application of this algorithm to the above image produces the segmentation pictured below (it can be difficult to see so a zoomed in section is included beside):

![alt text](https://github.com/emmettFC/selected-projects/blob/master/tidal_bathymetry_model/assets_README/super-pixels-apply-OW.png)

This implementation of the SLIC algorithm is optically very good at building segments in the over-water imagery. The zoomed in section on the right shows how the model is able to distinguish between subtlety different but meaningful segments between submerged regions. The red arrow in the image above on the right indicates a segment that contains no un-submerged points, but is correctly separated from the adjacent segment on the left. The substrate of the indicated segment is shallow rocks, while the adjacent segment has a sand bottom. I mention this because it underscores the high performance of this algorithm given no context or supervision. It can therefore be used to help ameliorate some of the effort required to prepare image data for bathymetric analysis. This is because if the number of clusters (K) is increased—it is passed as a parameter—the model will be able to isolate un-submerged points in over-water imagery with high accuracy(15)(14). The model also shows the ability to distinguish between variable substrate reflectance properties. The former allows for the generation of masks for these over-water segments, while the latter could help to parameterize bathymetric models that specify values for substrate reflectance classes. 

An interesting feature of the canny edge detection process is that it builds a binarized representation of the coastline that can also be used to propose substrate classes. I ran a spatial density clustering on the canny image matrix and then used the convex hull approximation fucntion in openCV to produce segments in the image. The process did not work, even though a mask based on the density clusters shows a very good optical subsetting of the data. Ultimetly the goal though is to generate some set of minimal polygons for the density clusters and use them as abstract substrate labels. The mask from DBSCAN is shown below along with a manual labeling of how I would like the segmentation to draw boundaries. More work needs to be done to accomplish this going forward. 

![alt text](https://github.com/emmettFC/selected-projects/blob/master/tidal_bathymetry_model/assets_README/cluster-centers.png)


### Discussion of the physical model & potential for a sequential tidal correction 
The majority of physical models are based on approximations of the solution to the radiative transfer equations in water (3). In general, these physical models are used to propose a generic formal expression that can be empirically parameterized though regression analysis given known depth points. The tidal correction I am proposing is meant to help calibrate physical depth estimates by comparing the relationship between known and predicted changes in depth through periods of tidal change. Depth estimates will be made at regular one-hour intervals through the tidal cycle, starting at dead-low tide (t=0) and ending at dead-high tide (t=6), for a total of seven flights. 

For each flight, depth is then estimated using the physical model proposed by Lyzenga (2006) (17). For different intervals in the tidal cycle (t=i, t=i+n), the difference between estimated depths [Xe(i+n) – Xe(i)] will be compared to known change in depth based on tidal fluctuation (Xa = avg(tidal current velocity(t=i, t=i+n)) * dt). These known and estimated depths can be used directly to derive coefficients for the physical model specified by Lyzenga (2006). 

Lyzenga (2006) propose a simplification of an approximate solution to the radiative transfer equation in water. They begin with the approximate solution: 

![alt text](https://github.com/emmettFC/selected-projects/blob/master/tidal_bathymetry_model/assets_README/lyzenga-equations-ii.png)

## Appendix: Patch reef grazing halos 

A further application of drone photogrammetry that I intend to explore is the use of convolutional models to detect grazing halos in patch reef structures as a way to evaluate trophic activity in a given region. The idea is that the size of the grazing halos is related to the presence of predators, and as the predator population declines the grazing fish are able to move further and further away from the protection of the reef structure. Given the ability to use drones to take high frequency observations, we could look at the change in the grazing halo distance over time as a way to get some information on the corresponding change in predation. An example of the grazing halos detected in the Belize imagery is shown below: 

![alt text](https://github.com/emmettFC/selected-projects/blob/master/tidal_bathymetry_model/assets/grazing-halos.png)

