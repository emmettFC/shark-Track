# Simple physical plankton model
## Describing coastal plankton density with a one dimensional diffusion-sedimentation model with spatially variable diffusion coefficient and periodic excitation

### Introduction
This aim of this project is to develop a model that describes the variability of plankton density in-phase with the tidal current cycle of ~6 hours and 12 minutes. The model is intended to describe a short-duration localized periodicity of near-surface (top 1m) plankton density in terms of variable tidal current velocity. The behavior of the model will be compared to empirical data provided by 

   * a SMartBuoy deployed in the Warp station estuary in the North Sea by the Center for Environment Fisheries and Aquaculture Science          (CEFAS) and 
   * tidal gauge data from the nearby Sheerness station collected by the British Oceanographic Data Center (BODC)(8)(17)(18) 

The Warp station in the North Sea is located in a shallow tidal inlet, with stable depth of 15 meters and tidal range of 4.3 meters (8)(17). The water column is well mixed due to its shallow depth and turbulent mixing as a result of tidal current. The proposed model will make several simplifying assumptions, which follow from the short-duration temporal scale of the research question and the characteristics of the empirical context that is being explored. The model will assume the following: 

   * there will be no loss of plankton due to grazing or death; 
   * there will be no growth of plankton due to photosynthesis; 
   * at any time (t) the horizontal (x, y) distribution of plankton density will be assumed to be uniform in the inlet; 
   * there will be no consideration of changes in density corresponding to the change in volume due to rise and fall of the water              level; 
   * tidal current velocity will be treated as a laminar force perpendicular to the mouth of the inlet (10)
  
The model then seeks only to describe variability in the vertical (z) distribution of plankton density in the water column as a consequence of diffusion proportional to the periodic laminar flow velocity. 

The chlorophyll fluorescence readings used to validate the model are taken at a discrete point in the (x, y) space at a depth of 1 meter from the surface (17). This is because the SMART buoy takes CHL readings at a depth of 1m. The assumption of uniformity in the (x, y) plane parallel to the surface is made to eliminate the impact of horizontal transport / advection of plankton during the tidal cycle. This assumption follows from the empirical observation that changes in salinity and temperature are dominated by the 12 hour semidiurnal tidal cycle, though remain relatively stable through the 6 hour tidal current cycle (8). This is in contrast to the observed variability in surface chlorophyll, which fluctuates in phase with tidal current velocity. This observation comes from the paper below:

```
Dancing with the Tides: Fluctuations of Coastal Phytoplankton Orchestrated by Different Oscillatory Modes of the Tidal Cycle.
Blauw AN, Beninca` E, Laane RWPM, Greenwood N, Huisman J (2012)
PLoS ONE 7(11): e49319. doi:10.1371/journal.pone.0049319
```

The model will describe the change in plankton density at 1m spatial steps in the (z) direction, and at 1-second time steps through the tidal-current period of ~6 hours. The three dimensional distribution (x, y, z) of plankton density within each 1m section of the water column at any time (t) will be modeled as a uniform quantity. Though in the three dimensional plots, when density is translated into individual particles, they will be random gaussian variables within a 1m^2 cylindrical region. The concentration P(z, t) of plankton at depth (z) and time (t) will be the output of the model at each step in time, though it is the concentration P(t(n), zmax) in the top 1 meter of the water column that is of specific interest given the availability of empirical data. 

### Model derivation

#### Simple one dimensional diffusion over infinite domain (no boundary conditions)
The base of the model is a one dimensional diffusion equation—Fickian diffusion or the heat equation(19). The equation is expressed as the differential equation given by equation (1) below. This equation is not solvable analytically, though it can be solved numerically for the variable concentration P(z, t) given known parameters for D and sufficient boundary conditions (3)(19)(20). The left-hand expression is of order 1 in time, and therefore requires a single boundary condition for t=0 at each interval in z. The right-hand expression is a second order spatial derivative, and therefore requires two boundary conditions at either edge of the domain {x=0, x=dx(n)(19). Boundary conditions and initialization of parameters will be discussed bellow. The simplest and most intuitive method of solving this equation numerically is to use a forward in time centered in space (FTCS) finite difference method(19). This method allows you to discretize the problem in space and time by representing the right and left side as finite differences using Taylor expansions. The left-hand temporal derivative is thus restated and rearranged for P’(x, t). The FTCS approximation for the heat equation is given by equation (2) below. 

   ![alt text](https://github.com/emmettFC/selected-projects/blob/master/plankton_model/assets_README/_plankton_image_set_final_1.png)


In this implementation of the FTCS scheme I have not considered the error terms O(dt) and have disregarded the higher order terms. This is the baseline equation (equation (2)) that is used to build the model for this project, and can be used to describe the sequential change of concentration of particles specified by P(z, t +dt) at each step in time dt.

#### Simple one dimensional diffusion with sedimentation over infinite domain (no boundary conditions)

The one dimensional diffusion equation can be easily modified to incorporate a sedimentation term (10). This term is intended to describe the constant sinking of plankton due to gravity. Planktonic sinking is a well-studied phenomenon, and sinking rates for various species of plankton are correspondingly well documented. A study of suspended macro benthic gradients in submarine caves used a 2-dimensional diffusion-sedimentation model to describe the observed distribution of plankton perpendicular to the mouth of the cave(10). To parameterize the sinking term, the authors used a range of sedimentation rates between 10^-6 and 10^-3. I have experimented with a range of sinking rates based on the species of plankton common the region of the North Sea described by the empirical data (8):

    * Chaeotoceros 
    * Paralia
    * Skeletonema
    * Eucampia
    * Cylindrotheca
    * Plagiogrammopsis

The relatively higher concentration of benthic diatoms in this region requires a higher range of sinking rates, between 10^-5 and 10^-2 ms^-1(8). The continuous one dimensional diffusion equation with a sedimentation term is given  by equation (3). To incorporate the sedimentation term into the FTCS approximation, the first order spatial derivative must be replaced with the central difference approximation of the first derivative.The sedimentation term then can be replaced by the expression given in equation (4). Equation 5 gives the FTCS approxiation of the one dimensional diffusion equation ammended to include the descretized sedimentation term from equation 4. 

The units of Ws are ms^-1, the units of dz are ms^-1, and the units of the concentration P(z,t) are mass(m^-3), so the expression has units mass(m^-3). This is the same as the units of the D * the second spatial derivative, and since the two terms are summed the resulting expression has the correct units. It is illustrative to compare the results of a simulation of this model with the previous diffusion equation. 

![alt text](https://github.com/emmettFC/selected-projects/blob/master/plankton_model/assets_README/sed-and-nosed-graphs.png)

The lefthand plot following shows the output of the model when initialized with two symmetric concentrations (in magnitude and space) at t=0. In the righthand figure, the initial condition of symmetrical peaks flattens out from the maximum of the function on both sides as in the diffusion model, though now the peaks also shift towards the bottom. This is a pretty satisfying result, because what is being illustrated in this graph—and by this model—is the gradual diffusive spreading out of particles in the water column with a simultaneous constant downward drift or sedimentation. This is intuitively what I imagine the motion of suspended plankton would look like in fluid completely free of turbulent forcing. This is then a kind of ‘null model’ of the latent physical movement of plankton, and sets up the next step of adding in a periodic excitation to mimic the cycle of tidal current velocity. 

#### Simple one dimensional diffusion with sedimentation and periodic excitation

Initially I intended to model the vertical fluctuation of plankton as a periodic upward velocity proportional to the tidal current speed. This is however not a physically meaningful approach, since the vertical component of velocity in this context is the result of turbulent mixing / diffusivity that propagates both upward and downward(21). It is therefore preferable to build the periodicity into the model through a spatiotemporal variability in the diffusion coefficient D. The second set of equations given below describe this process: 

![alt text](https://github.com/emmettFC/selected-projects/blob/master/plankton_model/assets_README/_plankton_eq_final_2.png)

The first step is to change the constant D in the model to a function of time and space D(z, t). The addition of D(z, t) yields the expression given by equation 6. The left hand expression is already known by the forward difference approximation, though the right hand expression now has to be expanded by the chain rule. Applying the chain rule to the right hand expression we then get the expression given by equation 7. We now combine equation 7 with the descrete sedimentation term given by equation 4, and are left with expressions for D'(z,t)(partialP/partialz), D(z,t)(partial^2p/partialz^2) and Ws(partialp/partialz). The FTCS approxiamtions of each term are given by equations 8, 9 & 10. They are combined symbolically resulting in the final equation for this model espression by equation 11 above. Finally, the expression for D(z,t) must be specified: 

![alt text](https://github.com/emmettFC/selected-projects/blob/master/plankton_model/assets_README/_plankton_eq_final_3.png)

The point of expressing D as a function is to allow it to vary along with the fluctuation in tidal current speed. To simplify this, I make the assumption that at t=0 the tide is at its high or low extreme, and therefore the derivative of tidal height, or current velocity, is 0. Based on some trial and error, and consideration of the physical context, I chose to represent the function D(z, t) as given by equation 12 above given parameters (zbar = zmean; z = zn; e=proportionality constant; zbar(t;t=0) the initial concentration at zmean or zbar).

Practically, the value of D will vary according to both 1) the tidal current velocity at time t, and 2) the distance from the mean value of z or the middle of the water column. The latter dimension of diffusive variation is based on the principle of wall-bounded diffusion (21). For the sake of simplicity I am considering that the surface of the water and the sea floor are both equivalently static boundaries, and therefore the proportional relationship of D and z is symmetric about the average value of z (zmean). The value (e) or the constant of proportionality can be adjusted to bring about the desired relationship between D and Ws over time. At t=0, the sinusoidal term goes to zero and at z=zmean the function f(zmean, z) is at its maximum value of 1. Therefore D(zmean, 0) = v_min_zmean, and is the maximum value in the vector of D(zn, t=0) used to initialize the forward integration of the equation. The initial vector for D(zn, t=0) is pictured below: 

![alt text](https://github.com/emmettFC/selected-projects/blob/master/plankton_model/assets_README/i-density-vector-plank.png)

In order for the model to make consistent sense, the units of D(z,t) have to be the same as the constant D, m^2s^-1. Conveniently, the units of f(zmean, z) are meters, and therefore the function D(z,t)  is of units m^2s^-1, so it works out in the larger equation. A few more considerations have to be made before the model is in a workable form. First, the issue of the boundary conditions. When you plot the concentration SUM(P(zn, t)) for each time step, the impact of infinite domain on the model performance is very clear: 

![alt text](https://github.com/emmettFC/selected-projects/blob/master/plankton_model/assets_README/loss-of-mass-plankton-.png)

To avoid this loss of mass, I specified a impermeable boundary condition at the sea floor, or z=0, such that D(z=0,t) = -D(dP/dt) (19). This acts as an equal and opposite diffusive force at the low boundary, such that there is an effect of accumulation at the lower boundary. This improves the model both by retaining mass, but also bring the representation closer to the physical reality of particle sedimentation and suspension. 

Finally, the model requires that parameters for d_initial (v_min_zmean) and (e) be chosen. The range of acceptable values for D is constrained by the value of dz and dt. The Courant–Friedrichs–Lewy condition states that D <= dx^2/2dt, or the model will be unstable in time (24). With D=0.5 for dz,dt =1 the model yields negative concentrations. If you set the diffusion coefficient to 1, the model concentrations are completely nonsensical. All this to say that the values for d_initial (v_min_zmean) and (e) were chosen such that at the peak of tidal current speed (for t divisible by 10800 seconds and not 21600), the maximum positional value of D(z,t), which is at z=zmean, will be equal to 0.5. To satisfy this, the values for d_initial and (e) were both set to 0.25. Further, this is defensible parameterization because this means that on average, Ws is an order of magnitude less than D(z,t), which is appropriate given the empirical range of values for those parameters presented in the literature (8)(10).  When this is all put together, the function to iterate is actually quite simple: 

![alt text](https://github.com/emmettFC/selected-projects/blob/master/plankton_model/assets_README/_plank_code_segment_final.png)

This equation is not a particularly heavy operation and was straightforward to simulate. I ran the model for 10 cycles of 21600 time steps, which corresponds to 10 cycles of the tidal current period of 6 hours. 

![alt text](https://github.com/emmettFC/selected-projects/blob/master/plankton_model/assets_README/density-plank-difsed-final.png)

The top lefthand subplot shows the initial symmetric punctuated release (black), at the minimum of tidal current velocity, and then for each subsequent six hour period the density distribution at the max current speed (blue) and min current speed (red). Perhaps unreasonably the model instantly reaches an oscilatory state of periodic excitation once the initial density concentration has diffused towards the upper and lower buondary. The three conditions (initial, max current, min current) plotted in three dimensions with the same color convention is shown below: 

![alt text](https://github.com/emmettFC/selected-projects/blob/master/plankton_model/assets_README/plank-3d-density.png)

#### Comparision to empirical data 
One reason that I built this model is because of the paper referenced above that conducted a statistical analysis of longitudinal near-surface chlorophyll data in the North Sea, and found that tidal periodicities accounted for much of the observed variation(8). The goal of this project was to see if I could build a model that incorporated some basic but physically meaningful components that would mimic the observed behavior. 

The data on near surface chlorophyll fluorescence—which in times of low ambient light is a very good proxy for phytoplankton density—are maintained by the Center for the Environment Fisheries and Aquaculture (CEFAS)(17).  These data were collected from a Smart Buoy that automates the collection of oceanic data at regular intervals. Data on near surface chlorophyll is available for the period 2001-2018. While the CEFAS data has a number of interesting features, the buoy did not record tidal change and so I got the tidal gauge data from a very nearby station called Sheerness located in the same inlet in the North Sea. This dataset is maintained by the British Oceanographic Data Center (BODC)(18).

The tide gauge data is at 15-minute temporal resolution and the chlorophyll fluorescence data is at a 2-hour resolution, and so the analysis had to be done at 2-hour intervals. For each 2-hour interval, I brought in the discrete point tidal height, the discrete point tidal height from 2-hours previous, the chlorophyll fluorescence value at 1meter depth and the chlorophyll fluorescence value from 2-hours previous. Tidal current was then calculated as the absolute value of the difference in tide height between the two measurements: TC = abs(tideXn - tide0).

This is in line with the way that the function for D(z,t) takes the absolute value of the periodic rate of change (abs(sin(pit/6))), which facilitates the comparison. The tidal data is at a fine-scale temporal resolution and produces nearly continuous plots over time (pictured below for week 1 & 2 June 2003). When the data are resolved to the less granular interval of the chlorophyll data, the graph becomes less smooth and the current differential is exaggerated (pictured below for week 1 June 2003):

![alt text](https://github.com/emmettFC/selected-projects/blob/master/plankton_model/assets_README/empirical-data-1.png)

The chlorophyll fluorescence data is noisy and subject to instrumentation error and anomalous patchiness characteristic of phytoplankton. To try and smooth the data in time, the raw values are log transformed, and the change in density is measured as the differential of the natural log over the change in time(8). This expression can be restated through the application of the chain rule: 

![alt text](https://github.com/emmettFC/selected-projects/blob/master/plankton_model/assets_README/change-chl.png)

Which shows it is exactly equivalent to the relative rate of change in density. The final pre-processing step to mention is that ambient light or photosynthetically active radiance (PAR) can distort readings of chlorophyll fluorescence, and so intervals with PAR>1 at 1m depth, which is a proxy for daylight, should be excluded(8).  The PAR values were not populated in the data I received, so I used the Astral module in python to evaluate the sunset and sunrise times for each record. The simple uni-variate OLS model showed reasonably good correspondence between the two values. The clean and merged data set has ~2500 records, so a good heuristic for train and test splits is 20% or 500 records. The data were split into X and Y vectors, with 500 records removed for performance evaluation of the linear model. The model yielded a coefficient of 0.07, a mean squared error of 0.03 and a R^2 of 0.21. The graph of the OLS best-fit line is shown below:

![alt text](https://github.com/emmettFC/selected-projects/blob/master/plankton_model/assets_README/plankton-reg.png)

The regression above is done just to replicate the general finding of the study (referenced above) that chl flouresence fluctuates in phase with the tidal current cycle. The purpose of this analysis though is to formalize this process in a model that could be tuned to best correspond to the observed pattern. At this point I have not yet been able to quantify the comparison and tune the model parameters to best fit the observations. Going forward this would be the nest step in the process. 

In addition to a explicit comparison to empirical data, there are other dynamics components I would like to incorporate into this model.  First I would like to build in a probabilistic initialization of t=0 chl density after each cycle (could be a function of the previous density). The behavior of the model is also most irregular during the first couple of hours after t=0, during which time the diffusive expansion has not yet reached the point at which periodic fluctuation starts to happen. Formalizing this input as a stochastic element would introduce some interesting variability that might yield some insightful results. Second, tuning the parameters of the model to optimize correspondence to the empirical data could help to describe in more detail how the modeled forces may actually be interacting. Specifically, the sinking rate (Ws) is based on individual diatoms, which does not account for flocculation or the forming of aggregates, which have much higher characteristic sinking rates than single organisms(8). There is a class of coagulation-separation models that have been used in interesting ways to describe this process, and treat the formation of planktonic aggregates as a Markov chain(12).  Finally, I think an analysis of time-to-convergence of this model would be illustrative(3). The model is deterministic, but since it is intended to describe such short periods, the behavior of the model in the time before convergence is critical, and is something I would love to explore in more detail. 

The figure below shows a plot of the modelled surface density simulated over cycles of tidal current (bottom) and the empirical values of tide height, current velocity and chl density in the top 1m: 

![alt text](https://github.com/emmettFC/selected-projects/blob/master/plankton_model/assets_README/empirical-data-vs-model.png)



