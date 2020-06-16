# Description
This code does live tracking of eye pupil and plays a siren if the eyes are closed.

# Main code
Run the code in "Live pupil detection (Using Webcam) ver 5.0_final.ipynb".

# Requisites
This code has been run and tested on following python libraries:

| Module                | Version       |
|:---------------------:|:-------------:|
| dlib                  | v19.15.0      |
| imutils               | v0.4.6        |
| numpy                 | v1.15.0       |
| opencv-contrib-python | v3.4.2.17     |
| pygame                | v1.9.4        |
| scipy                 | v1.1.0        |

You can download the dlib's pre trained shape detector from here: http://dlib.net/files/shape_predictor_68_face_landmarks.dat.bz2

This code is somewhat combined implementation of these paper: https://pdfs.semanticscholar.org/81d5/c4b49fe17aaa3af837745cafdedb066a067d.pdf
https://ieeexplore.ieee.org/document/6869552    
https://ieeexplore.ieee.org/document/7492591
https://www.researchgate.net/publication/221667667_Eye_Pupil_Location_Using_Webcam
https://www.researchgate.net/publication/327907429_Automatic_Adaptive_Facial_Feature_Extraction_Using_CDF_Analysis

# Documentation
Inside the while loop, the webcam continuously captures images, the eye image processing is done and the required actions are carried out. The line "time.sleep(1.0)" ensures that a single image is captured in every 2 seconds. If this is not done, the white pupil locator on image will be unstable. The functions sound_alarm_on(path) and sound_alarm_off() are used to play and stop the alarm respectively when called.

The function eye_aspect_ratio(eye) calculates aspect ratio and based on that it is determined whether the eye is open or not. The “EYE_AR_THRESH = 0.2” says that we have set the threshold value of aspect ratio to 2.0. This threshold value is determined experimentally and may differ to some extent from one camera to other. The line “EYE_AR_CONSEC_FRAMES = 10” says that when we detect that the eye is closed, we will wait for next 10 frames. If the eye is still closed, the alarm is played.

The images of right and left eye are extracted using dlib’s pretrained facial landmarks detector. These images are sent to the function finding_pupil_centre(right_eye, ear) to locate the approximate  centre of the pupil.

NOTE: The parameter right_eye to the function finding_pupil_centre(right_eye, ear) doesn’t imply that only the right eye image is to be sent to the function. It can work on both right eye as well as left eye image. The name given is just a variable to store the image.
 
Inside the function finding_pupil_centre(right_eye, ear), we first perform Histogram Equalization on the input image. This is done to ensure that maximum no. of pixels belonging to the pupil are detected. Then using the line “hist, bin_edges = np.histogram(equ.flatten(), 256, [0,256])”, we create an array ‘hist’ having 256 elements which stores the no. of pixels having intensity value equal to the index of the element. Now, as given in a paper titled “Eye Closure and Open Detection Using Adaptive Thresholding Histogram Enhancement (ATHE) Technique and Connected Components Utilisation”, iris area of eye opening constitutes approximately within 8% to 16% of the entire extracted eye region. Based on this, we calculate the number of pixels belonging to pupil by multiplying the number of pixels in image with 0.08. Now using a for loop, we calculate the threshold intensity value. It is the intensity value at which the cumulative frequency of pixels in ‘hist’ becomes equal to the number of pupil pixels. Using the threshold intensity value, we distinguish between the pupil and the rest of the image. The pupil is white and the rest image is black and it is stored in new_image. 

NOTE: In the nested for loop in which we perform the binarization of image, the ranges used are (0, equ.shape[0]-5) and (0, equ.shape[1]-5). This is done to neglect the pixels belonging to eye-brows.

Then erosion is performed on the image using a 2X2 window to remove the small white noises and is stored in new_img_er1. Now even after this, it is possible that there will be more than one white blob in the image. To select the pupil, we choose that blob shape which has the largest area. This is done using findContours function of opencv. We find contours of various blobs in the image, find the blob having largest area and find the moments of that blob using moments function of opencv. The coordinates of the centre is calculated using those moments and stored in (cX, cY).  This centre is our approximation to the pupil centre. At this point, a small white circle is made on the original eye image to show the pupil centre. If ear<0.2 (i.e. eye is closed), (cX,cY) store (0,0). 

# Unit testing
1. Unit testing of finding_pupil_centre(): Give an input image having a face to the code in the file "Unit testing of finding_pupil_centre().ipynb". It will output two images, right eye and left eye found on the face with their pupils located with a white dot.

2. Unit testing of playing Alarm: Run the code in "Unit testing of Alarm playing code.ipynb". A screen will come showing your face. Close your eyes for 10-11 seconds. An alarm will be played alerting that your eyes are closed for some time. Open your eyes to stop the alarm.

# Limitations
Following are the limitations found in the code till now:
1. The code heavily depends on dlib's face detection capability. While testing in dark conditions, dlib was not able to perfectly capture the complete face of some users.
2. When the code doesn't detect the face of a user completely, it iterates over the same eye images and gives output with more than one white dots until a new and complete eye image is detected.

# Tested datasets
The code was tested on three users in three lighting conditions, which were daylight, light source from above (evening) and a single light source in a dark room. The datasets with their resulting output images are uploaded in nine zip files.
