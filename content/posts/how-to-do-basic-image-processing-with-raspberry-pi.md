---
title: 'How to Do Basic Image Processing with Raspberry Pi?'
date: Tue, 12 Dec 2017 19:04:26 +0000
draft: false
tags: [Tech]
---

### **Required Materials**

You need to have these components for getting started with basic image processing.

*   16GB Micro SD Card.
*   HDMI cable for display
*   A USB camera that is compatible with Raspberry Pi.

 

### Installing the Image Processing Program

Plug in all the components and open the Linux terminal in Raspbian and run the following commands:

*   sudo apt-get install update
*   sudo apt-get upgrade
*   sudo apt-get install python-pygame
*   wget http://www.cl.cam.ac.uk/projects/raspberrypi/tutorials/robot/resources/imgproc.zip
*   unzip imgproc.zip
*   cd library
*   sudo make install
*   cd..

Now we’re ready to program the Raspberry Pi for image processing!  

### Programming a Raspberry Pi for Image Processing

*   Make sure that you have connected the USB camera to Raspberry Pi
*   Open Python 3 and press CTRL+N to open a new window. In this window write the following code.

    from imgproc import *
    # open the webcam
    my_camera = Camera(320, 240)
    # grab an image from the camera
    my_image = my_camera.grabImage()
    # open a view, setting the view to the size of the captured image
    my_view = Viewer(my_image.width, my_image.height, "Basic image processing")
    # display the image on the screen
    my_view.displayImage(my_image)
     
    # wait for 5 seconds, so we can see the image
    waitTime(5000)

   

*   Save this code somewhere on the desktop and run it.
*   The code will capture an image and will display it on the screen.

 

### Turning Red Pixels to Blue with Raspberry Pi

Now we are going to turn all red pixels in the image to blue. Write this snippet of code below the code shown in the section above.

    # iterate over ever pixel in the image by iterating over each row and each column
    for x in range(0, my_image.width):
      for y in range(0, my_image.height):
        # get the value of the current pixel
        red, green, blue = my_image[x, y]
     
        # check if the red intensity is greater than the green
        if red > green:
          # check if red is also more intense than blue
          if red > blue:
          # this pixel is predominantly red
          # let's set it to blue
          my_image[x, y] = 0, 0, 255

  You now know how to do basic image processing on your Raspberry Pi!