## LCD Display jpg images

<iframe width="560" height="315" src="https://www.youtube.com/embed/jR3LpkfWWy8" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

The sst7789 driver library has a method to display pictures in jpg format, which is very friendly to us who are learning for the first time.

### jpg method

`jpg(jpg_filename, x, y)`

Draws a JPG file at the given x and y coordinates, which are the upper left corner of the image.

This method requires an additional 3100 bytes of memory for its working buffer.

### Prepare jpg files of appropriate size

Choose any picture you like, and crop it to a picture with a length of 320 pixels and a width of 170 pixels, or a picture smaller than this size.

There are a large number of optional picture editing tools in various smart terminal devices and various operating systems, and you can use your favorite tools for editing.

Here is a random web online image editing tool that can be used for free, [Pixlr X](https://pixlr.com/x/).

Put the cropped picture into our local MicroPython working folder, rename it to `pic_1.jpg`, and refer to the method of uploading the picture to the MicroPython device [Use mpbridge in the terminal](#use-mpbridge-in-terminal) .

A cropped image is prepared here.

![](../assets/images/pic_1.jpg)

### jpg method use case

Use the jpg method in the main.py script.

```py
""" BPI-Centi-S3 170x320 ST7789 display """

import st7789
import tft_config
import gc

def main():
     try:
         tft = tft_config.config(rotation=1)
         tft.init()
         tft.jpg("pic_1.jpg", 0, 0)
         tft. show()
         gc. collect()

     except BaseException as err:
         err_type = err.__class__.__name__
         print('Err type:', err_type)
         from sys import print_exception
         print_exception(err)

     finally:
         tft.deinit()
         print("tft deinit")


main()

```

After uploading main.py, reset the device and you can see the picture on the screen.

Let's prepare a few more jpg files of appropriate size, and then we can design a loop, and play the pictures on the screen of BPI-Centi-S3 in a loop like a slide show.

![](../assets/images/pic_2.jpg)
![](../assets/images/pic_3.jpg)
![](../assets/images/pic_4.jpg)
![](../assets/images/pic_5.jpg)

```py
""" BPI-Centi-S3 170x320 ST7789 display """

import st7789
import tft_config
import gc
import time

pic_list = ["pic_1.jpg", "pic_2.jpg", "pic_3.jpg", "pic_4.jpg", "pic_5.jpg"]


def main():
    try:
        tft = tft_config.config(rotation=1)
        tft.init()
        while True:
            for pic in pic_list:
                tft.jpg(pic, 0, 0)
                tft.show()
                gc.collect()
                time.sleep(1)

    except BaseException as err:
        err_type = err.__class__.__name__
        print('Err type:', err_type)
        from sys import print_exception
        print_exception(err)

    finally:
        tft.deinit()
        print("tft deinit")


main()

```

