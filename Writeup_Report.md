# **Finding Lane Lines on the Road** 

## Self-Driving Car Nanodegree Program Project 1

---

**Project Goals**
* Make a pipeline that finds lane lines on the road for a single image
* Apply the pipeline to video


[startingImage]: ./test_images_output/starting.png "Starting"
[grayscale]: ./test_images_output/grayscale.png "Grayscale"
[gradient]: ./test_images_output/gradient.png "Gradient"
[houghLines]: ./test_images_output/lines.png "Lines"
[masked]: ./test_images_output/masked.png "Masked"
[maskedLines]: ./test_images_output/maskedFinal.png "Masked Lines"
[final]: ./test_images_output/simpleSolutionFinal.png "Final"


[topMask]: ./solution_videos/pipeline_improvement/topMask.png "Top Mask"
[bottomLines]: ./solution_videos/pipeline_improvement/bottomLines.png "Bottom Lines"
[extendedLines]: ./solution_videos/pipeline_improvement/extendedLines.png "Extended Lines"
[slopeFiltered]: ./solution_videos/pipeline_improvement/slopeFiltered.png "Slope Filtered Lines"
[twoBestGrouped]: ./solution_videos/pipeline_improvement/twoBestGrouped.png "Two Best Grouped Lines"

---

## How the Pipeline Works

We begin with an image taken from the camera on top of the car.
![Street image][startingImage]

### 1. Convert the image to grayscale

Lane lines are not always the same color. Some are white, some are yellow. In various lighting scenarios they may appear other shades. In order to account for the variations we first tranfsform the image to grayscale.

![Grayscale image][grayscale]

### 2. Find the Gradient

To a human driver, lane lines are visible because they are a different color than the road around them. Thus the computer needs to identify where in the image the pixels change from one color to another. Change in color per pixel is also called the derivative or gradient. We use an existing algorithm called canny to select only pixels where the gradient is large enough. This gives us an image that looks like so.

![Gradient image][gradient]

### 3. Identify Lines

So far we have points, put they are called lane LINES not lane POINTS. We use a technique calle a hough transform to find points that are arranged in a line. A hough transform takes a point in X,Y space and transforms it into a line in A,B space (where A and B are the constants in the traditional y = ax + b defintion of a line). The line in hough space corresponds to all the possible lines that contain the point. To find the lines contained in the collection of points, we transform all the points to hough space and then look for intersections. Intersections in hough space indicate that a single line in X,Y space contains multiple points from our collection. Using an existing python library, we obtain this image.

![Lines image][houghLines]

### 4. Mask the Image

The previous image does correctly identify the lane lines, but it also has a lot of extraneous lines as well. In order to only get the lines we care about, we apply a mask to the image. This essentially means we make everything black except for the parts of the image we care about. (This step is actually done before we do the hough transform)

![Masked image][masked]

Below is the final masked image. We are left with only the lane lines (plus some extra ones at the top due to the car ahead of us, but we will deal with that in the next step).

![Final masked image][maskedLines]

### 5. Combine the Lines

We now have a collection of lines. But what we really want is a single line for each lane marker (one on the left and one on the right). To identify which lines belong to which side we look at the slope of each line and group the positive slopes together and the negative slopes together. We use an existing python function (polyfit) to fit a first order polynomial (also known as a line) to all the end points of each group of lines. Those pesky extra lines I mentioned in the last step? The points are so close to the end points of the actual lines they don't affect the final solution much. But one easy way to get rid of them is to filter out lines with small slopes (the actual lane lines have slopes greater than 0.4). The lane is the portion beneath the two lines, so we find where the lines intersect eachother and where they intersect the bottom of the image. The final image looks like this. 

![Final image][final]

### 6. Apply to Video

Applying the pipeline to a video is simple. Just treat each frame of the video independently then combine all the frames. You can see the final product in the solution_videos/simple_solution folder (it appears it isn't possible to embed the video in the markdown itself). 

The first two videos work pretty well. There is some slightly wobbling of the lane line that isn't a solid line. The wobbling occurs when there isn't a piece of the lane line near the bottom of the image so there aren't any points to anchor the polynomial fit there. But the wobbling is slight and it still correctly identifies the lane lines.

For the challange video, the pipeline fails entirely. Let's look at why it failed and how we can fix it.

## Changes to Make Pipeline More Robust

I attempted to improve the pipeline entirely in the line processing side. I wanted to see if it was possible to accomplish without modifying the image processing at all. In the end I decided it wasn't possible, but here is a description of what I tried.

### No Masking

The most obvious problem is that the lane is not in the same location in the challenge video as it was in the other two. The mask is looking at the completely wrong part of the image. That can be seen in the Original Mask video in the solution_videos/pipeline_improvements folder.

A quick fix would be to simply change the mask. But we need a solution that will work regardless of the camera orientation and doesn't require a human to define a mask every time the camera moves. I start by using a mask to only retain the bottom half of the image (I'm assuming the camera is rightside up and the lane lines aren't in the sky).

![Top Mask image][topMask]

I then go through the same process to identify lines in the image.

![Bottom Lines image][bottomLines]

### Group the lines

From here, I calculate the slope and y-intersect for all the lines and extend them out. This makes it easy to see groupings of lines

![Extended Lines image][extendedLines]

To reduce the noise, I remove lines whose slopes are too small.

![Slope Filtered Lines image][slopeFiltered]

Then I group the lines that have similar slopes and intersects (essentially doing another hough transform and looking for points that are close together in hough space). As a first attempt, I simply select the two largest groups (ones that contain the most lines). This works a lot of the time, but isn't perfect.

![Two Best Grouped Lines image][twoBestGrouped]

It mostly works for the two normal videos, there are some frames where one lane dissapears becuase the lines aren't similar enough to be considered the same group of lines. This could be fixed by playing with the parameters for how close the slope and intersects must be for each line to be considered the same as another line.

It does not work for the challange video. It gets the left lane line correct, and sometimes gets the right one. But when it goes over the bridge with the white asphalt it fails completely. The pipeline isn't able to see the lines well enough with the lighter background.

## Ways to Improve

Here are some ways the pipeline could be improved.

### Look for Opposite Lines

In my improvement attempt I just selected the two groups of lines with the most lines. Instead, I could look for groups of lines that look like a lane (they have opposing slopes, they are on opposite sides of the middle of the image, they are a fair distance apart).

### Higher Degree Polynomials

Our solution always looked for straight lines. But lanes can turn, as it did in the challange video. We could get a better solution by looking for solutions that aren't just straight lines, but higher degree polynomials instead.

### Remember Previous State

Since we are dealing with video and not single images, the solution from previous frames should impact our current solution. Lane lines should be in a similar location from one frame to the next so we should look in that general area.

### Color Filtering

Instead of just applying a grayscale we could be more intelligent. Lane lines are always yellow or white so we could select for just those colors when looking for the gradient.
