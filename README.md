<style>
    .fattrib{
        color: green;
    }
</style>

# ddr_davis_data
Pakcage uses lvreader(1.2.0) to read and write .set file of Davis. lvreader is not installed with the package and hence it has to be installed separately. lvreader is not available on pypi (as of Sept 2022).

### Download and install lvreader
<a href="https://www.lavision.de/en/downloads/software/python_add_ons.php" target="_blank">Download *.zip* file of lvreader(1.2.0) from here</a>

Extract the *.zip* file. lvreader has good manual to understand its usage. For independent use of lvreader, user can follow the manual. We will install the lvreader in our sytem from the *.whl* (wheel) files.
Inside there are multiple *.whl* files. According to the python version, the resective file needs to be installed. for Python 3.9.0 install *lvreader-1.2.0-cp39-cp39-win_amd64.whl*. For Python 3.10 install *lvreader-1.2.0-cp310-cp310-win_amd64.whl*.

<sub>Above instructions assumes Windows as OS. For Linux the *.whl* file name changes which is easily distinguishable in the list of files.</sub>

Then open Anaconda powershell or cmd.exe. navigate to the extracted folder and perform the installation using pip.
```py
pip install lvreader-1.2.0-cp310-cp310-win_amd64.whl
```
<sub>Select the file name according to the Python version or-else the error will come.</sub>

We have installed lvreader and all its required dependecies in the system. Now we will install ddr_davis_data

### Install ddr_davis_data
Install ddr_davis_data using pip.
```py
pip install ddr_davis_data
```
### Instantiation


```python
import ddr_davis_data
import matplotlib.pyplot as plt
import numpy as np
import os
```

We need the filepath to Davis set. Here, we will take one average velocity set file. In Davis the files are arranges in chronological manner. The base or the first set (folder containing files) is of recorded images. Then folder inside the base set can be anything depending upon the processing performed. If background image is subtracted then, the next folder would contain subtracted images. Then next folder inside that folder would be instantaneous and then average velocity folder. The hierarchy of the folder depends upon the processing sequence performed. For the case shown here, the hierarchy is as follows.

```
recorded images
│   .im7 files - all images (from 0 to 100 or 1000)
│   ...
|   background subtracted images.set
└───background subtracted images
    │   .im7 files
    │   ...
    |   instantaneous velocity fields.set
    └───instantaneous velocity fields
        │   .vc7 files containing Vx, Vy or Vz (depending upon type of PIV)
        │   ...
        |   average velocity field.set
        └───average velocity field
            │   .vc7 files containing average of Vx, Vy or Vz (depending upon type of PIV)
            │   ... another files depending upon the processing
```
In this case, we will take instantaneous set file. The set file is located in the parent directory of the folder as shown above. For the filepath, we can either give the path to *.set* file or folder path.

```py
filepath1 = r'D:\recorded images\background subtracted images\instantaneous velocity fields'
```


```python
s1 = ddr_davis_data.velocity_set(filepath = filepath1, load=True, rec_path=None, load_rec=False)
```

s1 is the velocity set object. If the set folder is in hierarchy of the recorded image, then load_rec=True will load the recording set as well. Loading recording set helps in accessing recoded or raw images. To make the instantiation of velocity_set faster with recording set, it is better to give recoding set folder-path directly to <sec class="fattrib">rec_path</sec> attribute above.

## Accessing data

Every set file has many of attributes such as *scales, offsets, limits, U0, V0, camera exposure time, Laser power, recording time, recording rate etc.*.
```py
s1.attributes
```
This will give the dictionary-like data of all the attributes attached with the s1 set file.
There are default and intuitive functions to get certain important attributes as follows.


```python
s1.recording_rate
```




    10.0



gives the recording rate of the s1 set file. similary the time delta in between the frames can be accesssed by,


```python
s1.dt
```




    49.0



Depending upon the way Davis packs the data, sometimes the attribute name changes and then the above functions might not work. At that time it is better to access data using
```py
s1.attributes
```

### Velocity data
Velocity data can be accessed by giving comonent name. Method returns the numpy_masked_array like data.


```python
s1.u(n=0)
```




    masked_array(
      data=[[--, --, --, ..., --, --, --],
            [--, --, --, ..., --, --, --],
            [--, --, --, ..., --, --, --],
            ...,
            [--, --, --, ..., --, --, --],
            [--, --, --, ..., --, --, --],
            [--, --, --, ..., --, --, --]],
      mask=[[ True,  True,  True, ...,  True,  True,  True],
            [ True,  True,  True, ...,  True,  True,  True],
            [ True,  True,  True, ...,  True,  True,  True],
            ...,
            [ True,  True,  True, ...,  True,  True,  True],
            [ True,  True,  True, ...,  True,  True,  True],
            [ True,  True,  True, ...,  True,  True,  True]],
      fill_value=0.0)



This is the $V_x$ velocity with masks. Mask crops the useful data from the overall image. Mostly PIV frames are not 100% used for velocity measurements. Meaning if jet is flowing from center of the image then the corners of the image is rendered useless for velocity measurement. Masks helps in neglecting those areas. It only considers the area of interest.

Similarly $V_y$ can be accessed as follows.


```python
s1.v(n=21)
```




    masked_array(
      data=[[--, --, --, ..., --, --, --],
            [--, --, --, ..., --, --, --],
            [--, --, --, ..., --, --, --],
            ...,
            [--, --, --, ..., --, --, --],
            [--, --, --, ..., --, --, --],
            [--, --, --, ..., --, --, --]],
      mask=[[ True,  True,  True, ...,  True,  True,  True],
            [ True,  True,  True, ...,  True,  True,  True],
            [ True,  True,  True, ...,  True,  True,  True],
            ...,
            [ True,  True,  True, ...,  True,  True,  True],
            [ True,  True,  True, ...,  True,  True,  True],
            [ True,  True,  True, ...,  True,  True,  True]],
      fill_value=0.0)



n is the image number for which velocity is enquired. If there are 100 images then n ranges from 0 to 99. As s1 is instantaneous set file, it contains many *.vc7* files. But for average set file, there will be only one .vc7 file which will have average of velocity components, then n should be 0 (which is its default value).

```py
s1.vector_coords(n=0)
```

    x= [[-1.         -0.99659864 -0.99319728 ... -0.00680272 -0.00340136
       0.        ]
     [-1.         -0.99659864 -0.99319728 ... -0.00680272 -0.00340136
       0.        ]
     [-1.         -0.99659864 -0.99319728 ... -0.00680272 -0.00340136
       0.        ]
     ...
     [-1.         -0.99659864 -0.99319728 ... -0.00680272 -0.00340136
       0.        ]
     [-1.         -0.99659864 -0.99319728 ... -0.00680272 -0.00340136
       0.        ]
     [-1.         -0.99659864 -0.99319728 ... -0.00680272 -0.00340136
       0.        ]]
    y= [[ 0.          0.          0.         ...  0.          0.
       0.        ]
     [-0.00452489 -0.00452489 -0.00452489 ... -0.00452489 -0.00452489
      -0.00452489]
     [-0.00904977 -0.00904977 -0.00904977 ... -0.00904977 -0.00904977
      -0.00904977]
     ...
     [-0.99095023 -0.99095023 -0.99095023 ... -0.99095023 -0.99095023
      -0.99095023]
     [-0.99547511 -0.99547511 -0.99547511 ... -0.99547511 -0.99547511
      -0.99547511]
     [-1.         -1.         -1.         ... -1.         -1.
      -1.        ]]
    

Returns tuples of numpy-array like data. The data is mesh-grid of $X$ and $Y$ coordinates of velocity vectors.

# Plotting
Contour, Streamline, Quiver etc. plots can be made from the meshgrid coordinate data and velocity vector data which we accessed above. There are special methods which is described below to output the data for plots and also there are plotting functions in the library to directly plot the data.

```py
s1.plot(n=10)
```


    
![png](README_files/README_20_0.png)
    


Above code directly access the ` lvreader.frame.VectorFrame.plot()` function. There are separate functions in the module to plot various informations.

### Plot filled contour

`ddr_davis_data.plot_contourf(*args,**kwargs)` is used to plot filled contour. There are 2 ways to use the function.

#### 1) Giving velocity_set as input
```py
fig = plt.figure(figsize=(3,1.5))
ax = fig.add_subplot(111)

ddr_davis_data.plot_contourf(ax=ax, vel_set=s1, n=10, z='velocity_magnitude', font_size=7, ctitle='|V|[m/s]')
plt.show()
```


    
![png](README_files/README_22_0.png)
    


`n` is the image/frame number. `z` determines which scalar to plot. There are many options to that depending upon the Davis output file. Currently 'u,v,velocity_magnitude, omega_z, Wz, TKE' options are available. There is a workaround this method of plotting which we will see shortly. Other attributes can be understood from its name or from plot outcome.

##### Multiple axes
`ax` is the axes on which to plot the contour. If there are multiple subplots then this attribute is very helpful. Below code plots the $V_x$ and $V_y$ in two different subplots.

```py
fig = plt.figure(figsize=(8,2))

ax1 = fig.add_subplot(121)
ddr_davis_data.plot_contourf(ax=ax1, vel_set=s1, n=7, z='u', font_size=10, ctitle='$V_x$[m/s]')

ax2 = fig.add_subplot(122)
ddr_davis_data.plot_contourf(ax=ax2, vel_set=s1, n=7, z='v', font_size=10, ctitle='$V_y$[m/s]')

plt.show()
```


    
![png](README_files/README_24_0.png)
    


#### 2) Giving data as input
This method of plotting gives better control over the data to be plotted. In the previous example, we gave velocity_set as input. The plotting function makes the data inside using `z` value. If we make the data outside the function and give the data to plotting function then it will do the same thing, but we could do multiple mathematical manipulation of data before plotting.

`data` should be dictionary like object. 
```py
d1 = s1.make_data(n=10)
``` 
Gives the dict output with 'x','y','u' and 'v' keys with numpy_masked array like values. To make the data ready for plotting `d1['z']` should be set to the scalar which we want to plot.

```py
fig = plt.figure(figsize=(8,2))
d1 = s1.make_data(n=10)

ax1 = fig.add_subplot(121)
d1['z'] = d1['u']
ddr_davis_data.plot_contourf(ax=ax1,data=d1,font_size=7,ctitle='$V_x$ [m/s]')

ax2 = fig.add_subplot(122)
d1['z'] = d1['v']
ddr_davis_data.plot_contourf(ax=ax2,data=d1,font_size=7,ctitle='$V_y$ [m/s]')

plt.show()
```


    
![png](README_files/README_26_0.png)
    


This will plot the same figure as plotted previously using velocity_set. Now here we could do mathematical manipulation of data before storing it to `d1['z']`. If both `vel_set` and `data` are defined then `data` is given the priority above `vel_set`.

#### Common colorbar and use of `vmax` and `vmin`
`vmax` and `vmin` comes handy when we want to plot common colorbar for multiple subfigures. below code shows the example for the same.
```py

vmin= -1
vmax= 0
colormap='hot'

fig = plt.figure(figsize=(5,1.5))

d1 = s1.make_data(n=10)

ax1 = fig.add_subplot(121)
d1['z'] = d1['u']
ddr_davis_data.plot_contourf(ax=ax1,data=d1,
                             vmax=vmax,vmin=vmin,add_colorbar=False,colormap=colormap)


ax2 = fig.add_subplot(122)
d1['z'] = d1['v']
ddr_davis_data.plot_contourf(ax=ax2,data=d1,
                             vmax=vmax,vmin=vmin,add_colorbar=False,colormap=colormap)

plt.tight_layout()
#adding colorbar to the right
plt.subplots_adjust(right=0.85)
ax = fig.add_axes([0.92,0.05,0.015,0.85])
ddr_davis_data.plot_colorbar(vmin=vmin,vmax=vmax,cax=ax,font_size=7,colormap=colormap,roundto=2,ctitle='$Common$ [m/s]')

plt.show()
```


    
![png](README_files/README_28_0.png)
    


In the above plots we have changed the type of `colormap` to *hot*.

### Plot quiver (vectors)
The method to plot the quiver is almost same as above. It shares the same code architechture. There is difference in `fracx` and `fracy` attributes, which controlls the amount of vectors displayed in $X$ and $Y$ axes respectively. The higher the number, less the amount of vectors displayed.

Vectors makes more sense when displayed over the contour plot. Hence below figure will plot vectors over the contour plot.
```py
fig = plt.figure(figsize=(3,1.5))
ax1 = fig.add_subplot(111)

d1 = s1.make_data(n=10)
#calculating velocity_magnitude
d1['z'] = (d1['u']**2 + d1['v']**2)**0.5

#plotting contour of velocity magnitude
ddr_davis_data.plot_contourf(ax=ax1,data=d1,font_size=7,ctitle='|V| [m/s]')
#plotting vectors, here the width and scale are adjusted for better vision
ddr_davis_data.plot_quiver(ax=ax1,data=d1,fracx=5,fracy=5,scale=100,width=0.005,color='#000000ff')
plt.show()
```


    
![png](README_files/README_30_0.png)
    


### Plot streamlines
When data from `s1.make_data()` is supplied to `matplotlib.pyplot.streamplot()` then error of `strictly increasing array` pops up. Hence turnaround was to use 1D linear array. There is special function within the velocity_set object which gives the data to plot for streamlines.
```py
fig = plt.figure(figsize=(3,1.5))

ax1 = fig.add_subplot(111)
d1 = s1.make_data(n=10)
d1['z'] = (d1['u']**2 + d1['v']**2)**0.5

ddr_davis_data.plot_contourf(ax=ax1,data=d1,font_size=7,ctitle='|V| [m/s]')
d1 = s1.make_streamline_data(n=10)
ddr_davis_data.plot_streamlines(ax=ax1,data=d1,density=(5,5),linewidth=1,color='white',arrowsize=0.01)
plt.show()
```


    
![png](README_files/README_32_0.png)
    


here `arrowsize` is 0.01 for better visualization. But for some cases, default `arrowsize` gives good results.

## Accessing recording images
Recording images can be accessed if `load_rec=True` in instantiation of velocity_set object. Also there should be recording *.set* files in the parent directory (at any level) of the velocity set file.


```python
s1 = ddr_davis_data.velocity_set(filepath = filepath1, load=True, rec_path=None, load_rec=True)
```

### plotting the image
The structure to plot image is similar to contour plot.
- make the data `d1 = s1.make_image_data(n=10)`
- plot the data `ddr_davis_data.plot_image(data=d1)`

```py
fig = plt.figure(figsize=(3,1.5))
ax1 = fig.add_subplot(111)

d1 = s1.make_image_data(n=10,frame=0)
ddr_davis_data.plot_image(ax=ax1,data=d1,vmin=0,vmax=60)
plt.show()
```


    
![png](README_files/README_37_0.png)
    

