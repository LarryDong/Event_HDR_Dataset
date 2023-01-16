
# Event_HDR_Dataset
Real-time HDR video fusion with events and frames. Released dataset.  
Download from [google drive](https://drive.google.com/drive/folders/13p4EpDHO1oHKcUhPM0wUvVc_7u0fVzfU?usp=sharing)  
Codes in the paper is also avaliable (released after paper accepted): [EvRe-HDR](https://github.com/LarryDong/EvRe-HDR)


## Recording Platform

Our recording platform includes one event camera (DAVIS346), two RGB cameras (HIKROBOT MV-CA013-21UC), and a hardware synchronization circuit.  
Two beam splitters are used to reduce the installation parallex: the first divides the half of radiance to DAVIS346, and the second divides the other half to RGB cameras again.  


Three cameras have the same lens: 12mm, f/1.f aperture, distortion is neglectable (<1%).
The exposure time of DAVIS346 is automatically set by its built-in algorithm, and exposure time of two RGB cameras are set to half and double of the DAVIS346's.  
Two RGB cameras' gain is set to be 6.02dB to compensate the impairment of the 2nd splitter.

The synchronization circuit emit 10Hz PWM.  
RGB cameras capture images based on this PWM.  
DAVIS346's frames are not controlled by this PWM, but a special trigger is recorded. This means that the DAVIS346's frames are not synchronized with RGB's frames, which we adopt a interpolation strategy to align them.




## Dataset
The dataset includes 4 sequences: `window`, `poster`, `tree`, `mountain`. For each sequence, we have:   
```
calib/
cam0/
cam1/
davis_frame/
gt/
events.csv
events_image_ts.csv
rgb_ts.csv
triggers.csv
```


`calib/`: includes `calib.yaml` and chessboard captured by 3 cameras. We adopt a homography transformation to reduce the installation parallex to achieve pixel-level alignment.  
`cam0/` and `cam1/` includes LDR images by two RGB cameras.  
`davis_frame/` includes intensity frame captured by DAVIS346.  
`gt/` includes ground truth (GT) HDR by: 1) merging LDR images in `cam0/` and `cam1/` first, and then 2) use interpolation to find HDR images at timestamp of `davis_frame/`. That is, the number of GTs is the same as davis_frame, but not LDR images.  
`events.csv` is raw events by DAVIS346. Dataformat: ts(in us), x, y, polarity.  
`events_image_ts.csv` is the timestamp of `davis_frame`. 
`rgb_ts.csv` includes index and filename of `cam0/` and `cam1/`.  
`triggers.csv` includes the timestamp of synchronization circuit triggers, which `cam0/` and `cam1/` are captured based on this timestamp.


Attention that each sequence includes all raw data, which you may not need to use all of them.  
More details are as follow.

### Simple usage
You can just use events in `events.csv` and DAVIS346's frame in `davis_frame/` (whose timestamp are listed in `events_image_ts.csv`).  
HDR GTs are provided in `gt/`, whose timestamp are exactly the same as `davis_frame/`.  

### Generate HDR GTs
The GT HDR provided are merged using Mertens method: Mertens, T., Kautz, J., & Van Reeth, F. (2007, October). Exposure fusion. In 15th Pacific Conference on Computer Graphics and Applications (PG'07) (pp. 382-390)  
If you want to generate HDR by your own, you need LDR images in `cam0/` and `cam1/`, and the calibration file `calib.yaml` in `calib/`.  
Moreover, LDRs are captured based on `triggers.csv`, but not `events_image_ts.csv`, then you need to apply interpolation (e.g. homography) to get HDRs at DAVIS346's exposure moment.  
Our interpolation is: feature matching between two images and calculate the homography, and then interpolate based on the timestamp to find the interpolation homography. Check our paper for more details.  

### Geometric calibration
Although we try to install three cameras with splitter to eliminate the parallex, there is always some misalignment.  
We use chessboard to find the homography between 3 cameras to achieve pixel-level alignment.  
Homography are provided in `calib/calib.yaml`, or you can generate the homography using raw images by your own.





