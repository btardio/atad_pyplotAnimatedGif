# Tutorial: Build an animated GIF using matplotlib

In this tutorial we will build an animated GIF that shows a line graph being filled in from left to right. 

* The solution uses the imageio library which is part of Anaconda. Using imageio we can accomplish this task without file I/O.
* The solution uses BytesIO, a native Python library, to facilitate reading and writing of a pyplot image.
* This solution also uses PIL, part of Anaconda, for conversion of raw bytes to an np array which is expected by imageio.

I was able to create a 400 image animated .gif without having to wait for a long time, but you should be able to fit in 500-700 comfortably.

## The code:

```python

#!/usr/bin/env python
# coding: utf-8

get_ipython().run_line_magic('matplotlib', 'auto')

import pandas as pd
import numpy as np
import imageio
from datetime import datetime, time, timedelta
import random
from matplotlib import pyplot as plt
from io import BytesIO
from PIL import Image

N_INTERVALS = 10 # specifies how many intervals on the X axis for plotting

# setup the plt, set ticks, bounds, params, grid
# note: since pyplot is a state machine, we do not need to pass in the fig or axes 
# or return them
def setupplt(xlist, ylist):
    # get the current graphics context figure
    fig = plt.gcf()

    # get the current graphics context axes
    axes = plt.gca()

    # disable autoscaling
    axes.autoscale(enable=False, axis='both')


    # x axis
    # create tick labels 1 to len(xlist) + 1
    axes.set_xticklabels([x for x in range(1,len(xlist)+1)]) 

    # create ticks, 1 to len(xlist) + 1
    axes.set_xticks([x for x in range(1,len(xlist)+1)]) 

    axes.set_xbound(1, len(xlist)) # set bounds, 1 to len(xlist)

    axes.tick_params(axis='x', direction='inout', rotation=45, 
                     grid_alpha=0.75, grid_linewidth=2) # set params

    axes.grid(b=True, axis='x') # turn the grid on

    

    # y axis

    # determine yintervals, number of y ticks, by taking the max of ylist and 
    # dividing by 10
    n_yintervals = int(np.floor(max(ylist)/10))
    
    # set ticks every n_yintervals from 0 to ceil(max(ylist)) + 1
    axes.set_yticklabels([y for y in range(0,int(np.ceil(max(ylist)))+1,
                                                 n_yintervals)])

    # set ticks every n_yintervals from 0 to ceil(max(ylist)) + 1
    axes.set_yticks([y for y in range(0,int(np.ceil(max(ylist)))+1,n_yintervals)]) 
    
    # set y bound as 0 to the ceiling of the maximum y value in the list + 1
    axes.set_ybound(0, int(np.ceil(max(ylist)))+1) 

    axes.tick_params(axis='y', direction='inout', rotation=45, 
                     grid_alpha=0.75, grid_linewidth=2) # set params

    axes.grid(b=True, axis='y') # turn the grid on



# create x axis
xintervals = [x for x in range(1,N_INTERVALS+1)]



# create y axis
sales = [ x*10*(1+random.random()) for x in range(1,N_INTERVALS+1) ]



# turn off interactive plotting so we don't see all of the images in a jupyter
plt.ioff()

# get the current graphics context figure
fig = plt.gcf()

# get the current graphics context axes
axes = plt.gca()

# initialize the images list
images = []

# create a bytes stream for storing the image in memory
# streams work by reading long data in intervals, possibly yielding to other 
# processes by reading large data in intervals it is less likely to fill ram or swap 
# and thus less likely to crash on systems with limited resources. to read more 
# about streams: 
# https://en.wikipedia.org/wiki/Stream_(computing)
# In addition, the BytesIO object acts as a data buffer, more about data buffers: 
# https://en.wikipedia.org/wiki/Data_buffer
iostream = BytesIO()

# iterate over the xintervals list, creating a picture for every slice of the list 
# and storing this picture as an np array
for z in range(0,N_INTERVALS+1):

    # clear axis
    plt.cla()

    # clear figure
    plt.clf()
    
    setupplt(xintervals, sales)
    
    # split our xintervals from 0 to z for animation
    fordays = xintervals[0:z]
    
    # split our sales list from 0 to z for animation
    forsales = sales[0:z]

    # plot the split lists
    plt.plot(fordays,forsales)
    
    # because we are not reinstantiating the iostream every iteration of the loop, 
    # seek to position 0, this will occur after Image.open(iostream) has read
    iostream.seek(0)
    
    # it's most likely that every image in the gif will be the sames size, but 
    # truncating the iostream gaurantees that header information that could change 
    # from one iostream to another is not added from a previous iostream if you were 
    # to use a lossy compression such as jpg, the files sizes would be different and 
    # this truncating might or might not be required based on how Image.open reads 
    # end of file information, in any case, it's a good practice to put this in
    iostream.truncate(0)
    
    # save the plot to the iostream in tiff format
    # note: png format was not being read by PIL.Image
    plt.savefig(iostream, format='tiff')
    
    # after saving the figure the seek position is at the end of the stream, 
    # reset it to the beginning for reading
    iostream.seek(0)
    
    # finally open the image and store the object as im
    im = Image.open(iostream)
    
    # convert the PIL Image to an np.array, which is what imageio expects
    # it's important to note that with the right patience and time you could convert 
    # the plot directly to an np.array knowing the format that is used by tiff, 
    # but this is essentially what PIL Image is doing
    pic = np.array(im)
    
    # append the np array to the images list
    images.append(pic)
    
    # This is nice except it doesn't use alpha channel
    #data = np.fromstring(fig.canvas.tostring_rgb(), dtype=np.uint8, sep='')
    #data = data.reshape(fig.canvas.get_width_height()[::-1] + (3,))

# close the stream
iostream.close()

# finally save the gif as a file on the hard drive
# imageio expects a numpy array. using mimsave, a list of numpy arrays
imageio.mimsave('movie.gif', images)

```


## Conclusion

Imageio's use is varied. In this example the only feature of imageio that was used was the animated gif feature. Imageio should be explored further if you are considering doing many operations on files with varying file types. Although beyond the scope of this tutorial, imageio is said to be used for scientific formats.

PIL is a widely used image library, much more popular than Imageio. It is also capable of creating animated gifs, which would be considered a better solution than the one used here. For purposes of exposure to multiple image processing libraries and to show data conversion techniques the code was unmodified. Using only PIL, the conversion from Image to np.array could have been skipped and an Image list could have been written directly to disk, speeding the solution significantly.
