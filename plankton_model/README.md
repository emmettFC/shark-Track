# Simple physical plankton model
## Describing coastal plankton density with a one dimensional diffusion-sedimentation model with spatially variable diffusion coefficient and sinusoidal excitation

### Introduction
This aim of this project is to develop a model that describes the variability of plankton density in-phase with the tidal current cycle of ~6 hours and 12 minutes. The model is intended to describe a short-duration localized periodicity of near-surface (top 1m) plankton density in terms of variable tidal current velocity. The behavior of the model will be compared to empirical data provided by 

   * 1) a SMartBuoy deployed in the Warp station estuary in the North Sea by the Center for Environment Fisheries and Aquaculture Science        (CEFAS) and 
   * 2) tidal gauge data from the nearby Sheerness station collected by the British Oceanographic Data Center (BODC)(8)(17)(18) 

The Warp station in the North Sea is located in a shallow tidal inlet, with stable depth of 15 meters and tidal range of 4.3 meters (8)(17). The water column is well mixed due to its shallow depth and turbulent mixing as a result of tidal current. The proposed model will make several simplifying assumptions, which follow from the short-duration temporal scale of the research question and the characteristics of the empirical context that is being explored. The model will assume the following: 

   * 1) there will be no loss of plankton due to grazing or death; 
   * 2) there will be no growth of plankton due to photosynthesis; 
   * 3) at any time (t) the horizontal (x, y) distribution of plankton density will be assumed to be uniform in the inlet; 
   * 4) there will be no consideration of changes in density corresponding to the change in volume due to rise and fall of the water               level; 
   * 5) tidal current velocity will be treated as a laminar force perpendicular to the mouth of the inlet (10)
  
The model then seeks only to describe variability in the vertical (z) distribution of plankton density in the water column as a consequence of laminar flow velocity. 

The chlorophyll fluorescence readings used to validate the model are taken at a discrete point in the (x, y) space at a depth of 1 meter from the surface (17). The assumption of uniformity in the (x, y) plane parallel to the surface is made to eliminate the impact of horizontal transport of plankton during the tidal cycle. This assumption follows from the empirical observation that changes in salinity and temperature are dominated by the 12 hour semidiurnal tidal cycle, though remain relatively stable through the 6 hour tidal current cycle (8). Further the assumption allows for the proposition of a specific discrete state space in the (z) direction, which will make a solution by finite difference methods possible. Specifically, the model will attempt to describe the change in plankton density of a cylinder with a height of 15-meters and cross-sectional area of 1m^2, stretching from the surface of the water to the sea floor. The model will describe the change in plankton density at 1m spatial steps in the (z) direction, and at 1-second time steps through the tidal-current period of ~6 hours. The three dimensional distribution (x, y, z) of plankton within each section of the cylinder at any time (t) will be assumed to be uniform. The concentration P(z, t) of plankton at depth z and time t will be the output of the model at each step in time, though it is the concentration P(t(n), zmax) in the top 1 meter of the water column that is of specific interest given the availability of empirical data. 


### Project Overview: 
Philadelphia's Wissahickon Creek is one of its 7 major subwatersheds and drains into the Schuylkill river. I have identified two adjacent sections of the creek, each less than 300 yards long, which -- with the exception of short periods following very heavy rainfall -- are separated by rockbeds where the water level is < 2 inches deep. The effect of this segmentation is that there is an observably distinct population of fish on either side of these spillways, though the sections are nearly touching one another. While Redbreasted Sunfish, Rockbass, Smallmouth Bass and Pumpkinseed Sunfish exist in aproximately equal number, one section has an abundance of very large (>20 lbs) Common Carp, and the other has almost none. In the section without these Carp, there is an abundance large (1-4 pounds) White Suckers, which I have seen only very rarely in the downstream section. Though this is likely an unsubstantial phenomenon from the ecological perspective, it presents an interesting opportunity for data analysis. Namely, I wondered if this population discrepancy could be reproduced by the deployment of remote sensors, without application of any prior assumptions on the distribution of fish across the two sections of the creek. This project is as of this writing incomplete, though significant progress has been made. The following sections provide a breif / high-level description of the methods applied to this point.


### Hardware: 
In order to do this project, two underwater camera sensors were made using Raspberry Pi's (pictured below). The complete set of materials used is as follows: 
  * 2x Raspberry Pi SCB 
  * 2x Logitech C905 USB webcams
  * 2x 16G USB storage devices
  * 2x 5200 mAh lithium ion battery packs
  * 2x SPOR PCB with USB and Micro-USB 
  * 2x Micro-USB to USB cables
  * 2x plastic 7.5 x 3.5 x 2 inch plastic boxes 

Each of the devices was booted on a Raspbian linux image via micro-sd cards. Code for motion detection and subprocess for writing out labeled frames was implemented in python via openCV. Instllation was a bit trying, though this is generally the case with openCv given its significant associated requirements and space contraints of the SCB's. The boxes were sealed with Smarter Adhesive Solutions 16 fast-set medium bodied solvent cement, and mounted to submerged trees in the regions of interest. NB: the seams were insufficient, and on the second deployment one of the boxes leaked and the Pi was destroyed. I have since purchased another Pi and am working towards an improved housing design. 

![alt text](https://github.com/emmettFC/selected-projects/blob/master/fishViz/assets/camera-sensors.png)


### Motion Detection:
The process for detecting and classying the observed fish begins with a light-weight / real time motion detection functionality implemented in openCv. Motion detection is much less computationaly heavy than object-detection & classficiation, which requires the use of deep learning methodology. The classification component of this project is implemented only after image data has been collected by the submerged cameras, with the models being trained locally on labeled images from the stoage devices attatched to the Raspberry Pi's. The figure below, which is an example frame from a test deployment of the sensor in my fishtank, shows the output of the motion detection process. From left to right, the images show: the mask, and improved delta and the resulting video frame with bounding rectangle drawn on screen. Detection is simple in principle: openCv is told what the 'empty' tank looks like, and then a pixel matrix is created for the unoccupied space. Then, any deviations from this structure are marked as occupations, and motion is detected. The contraint here is that the model must therefore be told what the 'empty' frame is so that it can measure disruptions. Since the intent was to built a geneirc sensor, I had to design a process to identify the 'empty' frame without manually providing it. 

![alt text](https://github.com/emmettFC/selected-projects/blob/master/fishViz/assets/delta-mask-monitor.png)


### Application of regular SVD to initialize empty frame: 
For this I applied an implementation of regular SVD found in a solution proposed for one of the standardized videos from the 'Background Models Challenge Dataset'. The solution was presented as part of a Numerical Linear Algebra course originally taught in the University of San Francisco MS in Analytics graduate program, and can be found here: https://nbviewer.jupyter.org/github/fastai/numerical-linear-algebra/blob/master/nbs/3.%20Background%20Removal%20with%20Robust%20PCA.ipynb . The method itself is efficient and relies on the scikitlearn decomposition utility. The image below shows the actual empty tank taken a a frame from the video --on the left -- and the tank background after the application of SVD. The video quality is relatively poor, so the still frame is actually less clear than the one produced by the decomposition, which is awesome. This method is sensitive to changes in lighting, and so if there is vairable cloud cover, one inititalization will fail once the cloud cover changes significanlty. To solve this I included functionality to periodically recalculate the background frame. 

![alt text](https://github.com/emmettFC/selected-projects/blob/master/fishViz/assets/real-tank-vs-svd-tank.png)


### Use of motion detection to automate annotations: 
When motion is detected, the camera then transfers each occupied frame to the USB storage device. Given the number of frames per second that the device sees, and the large number of potentially spurious objects (such as leaves), I had to specify a large threshold value for what was considered to be a detected object. Further, these images are ultimately meant to be used to train a model, so images with many bounding rectangles and objects at multiple depths are difficult to annotate and to use. For this reason I also specified that a frame would only be written to memory if 1) it was of a large enough size and 2) if there was only one detected object in the frame. This may seem like a severe limitation but the camera sees so many objects that it turned out to be an effective method. Moreover, this set of constraints allowed for a very helpful adaptation of the model training process. Instead of using a GUI or command line utility to move through the frames and draw / label each rectangle, I was able to use the dimensions of the bounding rectangles to produce labeled xml annotations that could be used to train models for detection and classification. The figure below shows the output of this process for an adequatey sized frame. 

![alt text](https://github.com/emmettFC/selected-projects/blob/master/fishViz/assets/label-and-image.png)


### Fish classification: 
While efforts at motion detection were successful, classification of the fish images for population analysis proved to be a more stubborn problem. I initially beleived that object-detection was a required step in this process, though after using the motion detection process to generate automatic annotations, I realized it could be leveraged then to crop out regions of interest from the frame. This recognition could provide substantial contribution to this post, detailing a submission to a kaggle competition which asked participants to classify boated-fish from commercial fishing boat cameras (https://flyyufelix.github.io/2017/04/16/kaggle-nature-conservancy.html), on which I based my classification analysis. This paper creates an ensemble method in which the fist layer is object-detection, for which I now think deep learning is not nessecary. Even though they cant put any software on the boat cameras --unlike in my case -- there is enough stationary camera data to apply the same SVD process and reduce the complexity of the problem. For classification I attempted two general methods: 

  * Classify fish into 1) carp 2) sucker 3) other
  * Classify fish into 1) rock bass 2) sunfish 3) smallmouth bass 4) white sucker 5) carp

Domain of classifier: 

![alt text](https://github.com/emmettFC/selected-projects/blob/master/fishViz/assets/domain-fish-images.png)

Both methods were tested using ResNet-152 (Keras implementation with ImageNet pre-trained weights: https://gist.github.com/flyyufelix/7e2eafb149f72f4d38dd661882c554a6) recommended / written up as part of the solution to the very similar problem outlined in the solution referenced above-- indeed the author is the same, and has based his work on a paper: 
```
Deep Residual Learning for Image Recognition.
Kaiming He, Xiangyu Zhang, Shaoqing Ren, Jian Sun
arXiv:1512.03385
```
As expected, the 3 level classification yeilded much better results than the 5 level classification. This is because both the carp and sucker fish are so much larger than the other fish, that the distinction was simplified. Keeping in mind the known discrepancy in population across the two regions, the classification model ultimately learns to assign predictive weight to aspects of the stationary frame in each of the two scenarios (despite much of this context being eliminated by the bounding rectangles). It is very likely that a white sucker, if observed by the camera deployed in the downstream section, would be mislabeled as a carp or vise versa. Though this has not yet been observed, larger rock bass and small mouth bass have been mislabeled as carp in the region with many carp, while they have been labeled as white sucker fish in the region with many white suckers. An example of the problem of spurious frame based learning is pictued in the image below, which was taken from the solution write up linked in the above passage: heatmap shows regions of model focus for a given input image.  

![alt text](https://github.com/emmettFC/selected-projects/blob/master/fishViz/assets/spurious-features.png)

### To do / Expanding on current state: 
Moving forward, the model must be made more robust. Classification, while modestly effective in the 3 level implementation, was impacted significantly by the background features of the two locations as mentioned above. Some potential solutions to this have been proposedon the forum for the Nature Conservancy Fisheries Monitoring competition. Going forward I intend to test some of these methods for my project.  

![alt text](https://github.com/emmettFC/selected-projects/blob/master/fishViz/assets/arduino-temperature-sensor.png)

In the hardware department, there is a lot to be done. The first two cameras -- as evidenced by the destruction of one of them -- were not designed optimally. Work is needed to secure the cameras and insure that they are sealed properly. I have also built two temperature / humidity sensors that I have not yet deployed (one pictured above). For these, some more work on the Arduino sketch used to measure the temperature is required. 

## Appendix: Initital attempt at object-detection

### Object detection & training the SSD (single-shot-detection) mobilenet CNN: 
Before implementing the automated annotation functionality, I thought that an object detection model had to be trained so that fish classification could be accomplished. For this I used the tensorflow object detection API, which can be challenging to stand up. The process is as follows: 
  * Divide images into training and testing sets
  * Convert xml labels to csv
  * Create tf records file that can be read by tensorflow to train the model
  * Train the model and wait for enough steps to get adequate convergence (process illustrated in the figure below) 
  * Export the frozen inference graph to be used in objet detection
  * Run object detection script and observe results

![alt text](https://github.com/emmettFC/selected-projects/blob/master/fishViz/assets/loss-graph.png)

I am running the CPU version of tensorflow, which is tedious and inefficient. For this reason I have not yet been able to train for enough steps to produce a high performant model. That being said, even the CPU version with inadequate convergence produces an ok model. The model was run on the testing subset of images and was able in most cases to find the large carp in the video frames.

![alt text](https://github.com/emmettFC/selected-projects/blob/master/fishViz/assets/mobilenet-applied-carp.png)

### Thanks for reading!! 

![alt text](https://github.com/emmettFC/selected-projects/blob/master/fishViz/assets/me-with-sucker.png)


