# **Finding Lane Lines on the Road** 

---

In this project we intended to identify lane lines on the road, both in an image and in a video stream. 
We started by writing a pipeline that could identify such lines in a few test images, applying that same process to a video file.
Lastly, we refined our pipeline by taking those lines and extrapolated them to detect the entire lane lines, not just isolated segments.

[//]: # (Image References)

[pipeline_input_image]: ./submission/images/pipeline_input.jpg "Pipeline input"
[pipeline_output_image]: ./submission/images/pipeline_output.jpg "Pipeline output"
[grayscale_image]: ./submission/images/grayscale.jpg "Grayscale"
[blurred_image]: ./submission/images/blurred.jpg "Blurred"
[edges_image]: ./submission/images/edges.jpg "Edges"
[roi_image]: ./submission/images/masked.jpg "Masked region"
[hough_lines_image]: ./submission/images/hough_lines.jpg "Hough Lines"
[solid_white_curve]: ./submission/images/solidWhiteCurve.jpg "Solid White Curve"
[solid_white_curve_output]: ./submission/images/solidWhiteCurveOutput.jpg "Solid White Curve Output"
[solid_white_right]: ./submission/images/solidWhiteRight.jpg "Solid White Right"
[solid_white_right_ouptput]: ./submission/images/solidWhiteRightOutput.jpg "Solid White Right Output"
[solid_white_right_ouptput]: ./submission/images/solidWhiteRightOutput.jpg "Solid White Right Output"
[solid_yellow_curve]: ./submission/images/solidYellowCurve.jpg "Solid Yellow Curve"
[solid_yellow_curve_output]: ./submission/images/solidYellowCurveOutput.jpg "Solid Yellow Curve Output"
[solid_yellow_curve_2]: ./submission/images/solidYellowCurve2.jpg "Solid Yellow Curve 2"
[solid_yellow_curve_output_2]: ./submission/images/solidYellowCurve2Output.jpg "Solid Yellow Curve 2 Output"
[solid_yellow_left]: ./submission/images/solidYellowLeft.jpg "Solid Yellow Left"
[solid_yellow_left_output]: ./submission/images/solidYellowLeftOutput.jpg "Solid Yellow Left Output"
[white_car_lane_switch]: ./submission/images/whiteCarLaneSwitch.jpg "White Car Lane Switch"
[white_car_lane_switch_output]: ./submission/images/whiteCarLaneSwitchOutput.jpg "White Car Lane Switch Output"

---

### 1. Pipeline description

In this section we will describe the basic pipeline that meets the project specifications. It is implemented in the function `process_image()` and consists of eight steps. It expects an RGB image as input and will output an RGB image with lane lines drawn in red on top of it.

Input | Output 
- | -
![alt text][pipeline_input_image] | ![alt text][pipeline_output_image]

#### 1.1 Conversion to grayscale

The pipeline starts off by converting the input image to grayscale, in order to get a suitable input for the next step, Canny Edge Detection. This is easily done with the provided helper function `grayscale()`, which makes use of the function `cvtColor()` from OpenCV.
Original | Grayscale
- | -
![alt text][pipeline_input_image] | ![alt text][grayscale_image]

#### 1.2 Gaussian blurring

Next, we apply Gaussian Blurring (smoothing), which will suppress noise from the image. This is done with the provided function `gaussian_blur()`, which itself calls the function `GaussianBlur()` from OpenCV. We noticed a kernel size of 5 is a good enough choice for smoothing a reasonably good area.

Grayscale | Blurred
- | -
![alt text][grayscale_image] | ![alt text][blurred_image]

#### 1.3 Canny Edge Detection

After blurring the image, we feed it to the provided function `canny()`, which calls the function `Canny()` from OpenCV. We use the value 50 for parameter `low_threshold` and 150 for `high_threshold`. By maintaining the recommended 1:3 ratio, we obtained good detection results

Blurred | Edges
- | -
![alt text][blurred_image] | ![alt text][edges_image]

#### 1.4 Region of Interest masking

Next, in order to filter out most edges that aren't lane lines, we apply a ROI segmentation process. For this we assume that lane lines will always appear in roughly the same area of the picture taken by the front-facing camera. By using the provided function `region_of_interest()`, we are able to select a region contained by vertices {(0, height), (width/2 - 64, height/2 + 64), (width/2 + 64, height/2 + 64), (width, height)}. This helper function uses OpenCV's `fillPoly()` and `bitwise_and()` functions. 


![alt text][roi_image]
Keep in mind the image's (0,0) point is at it's top left, and that the *Y* coordinate increases as we move down

#### 1.5 Hough Lines detection

We applied the Hough Transform to those segmented edges. To achieve this, we modified the function `hough_lines()`, which uses OpenCV's `HoughLinesP()`, and made it directly return an array of the calculated Hough lines. This modification will come in handy at step 1.7.

We found that using a `rho` of 4, `theta` of one degree (PI/180), a `threshold` of 10, `min_line_length` of 8 and `max_lane_gap` of 16 produced adequate results.

![alt text][hough_lines_image]

#### 1.7 Outlier lines filtering

In order to improve our results, we added a filter of the obtained Hough Lines based on their slope. Every line with a slope greater than 1 or smaller than -1 will be discarded. This helped us prune erroneous lines that would make the final extrapolated lane lines less stable. While not particularly helpful for the mandatory videos, this proved to be most useful when tackling the optional challenge. You can see only the outliers being highlighted in red [here](https://youtu.be/jtcAqA08jIo)

#### 1.8 Lane lines extrapolation

In order to draw a single line for each of the two lanes, we need to extrapolate them from the edges we have. We started by separating our lines in two groups, for both the left and right lanes. We did this based on their starting position, since even though we were hinted to use the lines' slope, we found this method to be highly unreliable for determining which side a line corresponds to. 

After this, we used NumPy's `poly1d()` and `polyfit()`, which allow us to fit a polynomial of degree 1 to a set of coordinates, to generate the final lines for each lane. 

Lastly, we draw these lines with the function `line()` from OpenCV. 

All this functionality is inside `draw_lane_lines()`.

This gives us our pipeline's output:

![alt text][pipeline_output_image]

#### 1.9 Testing and results

This pipeline was tested on all provided test_images, giving us the following output.

Input | Output
- | -
![alt text][solid_white_curve] | ![alt text][solid_white_curve_output]
![alt text][solid_white_right] | ![alt text][solid_white_right_ouptput]
![alt text][solid_yellow_curve] | ![alt text][solid_yellow_curve_output]
![alt text][solid_yellow_curve_2] | ![alt text][solid_yellow_curve_output_2]
![alt text][solid_yellow_left] | ![alt text][solid_yellow_left_output]
![alt text][white_car_lane_switch] | ![alt text][white_car_lane_switch_output]

We also tested it successfully on videos [Solid White Right](https://youtu.be/RXybb0Zx470) and [Solid Yellow Left](https://youtu.be/DVckYXeCkX4)

This pipeline can also detect individual edges, just by changing the call to `draw_lane_lines()` inside `process_image()` to `draw_lines()`. A demonstration of this can be found [here](https://youtu.be/ms0FZ6Qe6Ps) and [here](https://youtu.be/Jf9McDjteOY)

### 2. Potential shortcomings with our current pipeline

* Curved lanes are not well detected nor extrapolated
* Yellow lines are hard to detect if not clearly different from the road color
* Extrapolated lane lines are shaky in video
* Variations in road color make detection of both yellow and white lines more difficult
* Cars close ahead may be erroneously detected as lane lines
* ROI masking does not work properly in road turns, since it will extend towards the horizon


### 3. Suggest possible improvements to your pipeline

* Using Histogram Equalization for enhancing contrast, since low contrast is one of the main obstacles in the challenge video
* Replacing grayscale intensity threshold with bi-color (white and yellow) color masking
* Adaptive ROI that updates it's vertices based on video optical flow or steering angle for turns
* Averaging detected lane lines over a number of frames to reduce shakiness
