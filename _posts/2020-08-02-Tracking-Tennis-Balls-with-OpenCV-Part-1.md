---
title: "Tracking Tennis Balls with OpenCV: Part 1"
layout: post
excerpt_separator: <!--more-->
---

I wanted to a buy a radar gun to measure the speed of my serve, but spending $100 on something I would only use a couple times didn't seem worth it. I ended up trying to make my own.

<!--more-->

Before making my own, I had tried downloading a couple apps that claimed to measure the speed of a tennis ball. None of them worked as expected. They occasionally spit out a number that seemed reasonable, but it was impossible to tell if it actually measured the speed or just randomly sppit out a number. My goal was to make something that was at least better than this.

This is not a tutorial of how to create a functioning app, but more my story of the process of making an app, including what I learned,

**Attempt #1**\\
After some preliminary research online on how to proceed, I decided to try using OpenCV to detect a color to track an object. Specifically, the goal would be to detect the color of a tennis ball in each frame of a camera feed. 

The first step was to figure out how to detect the correct color. I learned that converting the color space from RGB to HSV would be beneficial in this situation. HSV values represent the hue, saturation, and value of a color. Instead of a combination of red, green, and blue, the hue essentially already encodes what "color" the pixels are. Saturation and value basically describe the shade of that "color". This allows for an easier description of the target color range of a tennis ball, as it will usually be about the same hue with different saturation and value depending on lighting.

I converted the frames to the HSV color space thresholded them to the desired color range using OpenCV's "cvtColor" and "inRange" methods. The resulting frame would contain a contour that represents the tennis ball, which is determined using OpenCV's "findContours" or "HoughCircles" methods. Using successive frames, I was able to approximate the speed using the position of the ball and frame rate. The results are shown below.

![Color Tracking](/images/blog/color-tracking.png){:class="blog-img"}

This actually worked! Unfortunately, only in very specific situations. I was able to wave a tennis ball in front of my laptop's webcam, and it would output a speed. If I waved it faster, the values would increase. But once I took it outside and the ball moved faster than around 15mph, it would fail. The lighting and background noise made it too difficult to detect the ball and only the ball once it started moving fast.

**Attempt #1.5**\\
Even though my first attempt was not robust enough for real situations, I wanted to see how it worked in a mobile format. I created an Android app with the same functionality by using JNI calls to the OpenCV Java API. Below are a couple screenshots from the app.

![Mobile Tracking](/images/blog/mobile-tracking-1.png){:class="blog-img"}
![Mobile Tracking Threshold](/images/blog/mobile-tracking-2.png){:class="blog-img"}

Like before, it could also track tennis balls in good lighting that were not moving fast. Unfortunately, the app ran at around 4 fps. It was not optimized for mobile performance, but to make it more robust, I would probably have to use more complex tracking methods. This would most definitely worsen the performance, so I decided to try a different approach.

**Attempt #2**\\
Live video analysis did not seem to be viable for such fast moving objects. However, moving to a non-live approach opened up other tracking methods like background subtraction and optical flow that took more computational power than just filtering colors. 

Background subtraction does essentially what the name suggests. It takes the previous frames in a video, determines what is the background based on what pixels are changing. I specifically used OpenCV's "createBackgroundSubtractorMOG2". It creates a background model with first parameter "history", representing the length of history to determine a background model, and "varThreshold", representing the distance used to determine whether a pixel is represented by the background model or not.

I do not understand optical flow too well yet, but will read more about it. I just used OpenCV's "calcOpticalFlowFarneback" which calculates the optical flow vector of every point in the frame. The general idea is that it detects and understands the motion of objects in a scene based movements of patterns. I take it to be kind of the opposite of background subtraction, where instead of figuring out what doesn't move and subtracting it, it figures out what is moving. 

Here are links to my [original video](https://youtu.be/0XUFD0Nu7Ec){:target="_blank"}, [background subtraction video](https://youtu.be/pMITgscYxYE){:target="_blank"}, and [optical flow video](https://youtu.be/s_3exlzpEoc){:target="_blank"}.

These definitely worked much better than detecting color at tracking the tennis ball. In each video, it is pretty clear where the ball is to the human eye. The challenge I faced now was isolate which contours actually represented the tennis ball. I tried finding which patch best fit an ellipse and which patch had approximately the correct area for its dimensions. I wasn't able to reliably pick out the ball. I was able to correctly pick out the ball on one sample, and it outputted 130mph. This is definitely not correct but not too far off from the correct value, which I believe to be around 100mph to 110mph (I realize I don't have a ground truth to compare too without an actual radar gun).

In the end, this attempt was also unsatisfactory. It was much more robust than the color detector, but did not reliably track only the tennis ball. I am currently working on my next iteration, which I believe will be much more promising. The only problem is that it is much "dumber" than my initial idea, in that it requires more user input. 

Despite my failure to produce a working app so far, this has been the project I am most proud of, and had the most fun working on. I have learned a lot through this process, both in computer vision as well as app development. I look forward to actually creating a usable app around this idea and avoiding buying a radar gun.





