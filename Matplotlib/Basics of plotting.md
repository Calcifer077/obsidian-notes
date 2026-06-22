---
title: Basics of plotting
source: https://www.w3schools.com/python/matplotlib_intro.asp
created: 2026-06-11
---

The `plot()` function is used to draw points in a diagram. By default, this function draws from a line from point to point. It takes two parameters for specifying points in the diagram.

```python
xpoints = np.array([1, 8])
ypoints = np.array([3, 10])

plt.plot(xpoints, ypoints);
plt.show();
```

Output:
![](../assets/Pasted%20image%2020260611220553.png)

If you don't want to plot the line and only wish to show points, you can pass a third parameter `'o'`. `'o'` is a marker here, you can pass some predefined marker here.

```python
import matplotlib.pyplot as plt
import numpy as np

xpoints = np.array([1, 8])
ypoints = np.array([3, 10])

plt.plot(xpoints, ypoints, 'o')
plt.show()
```


![](../assets/Pasted%20image%2020260611220742.png)

Some predefined markers are:

| Marker | Description    |
| ------ | -------------- |
| 'o'    | Circle         |
| '*'    | Star           |
| '.'    | Point          |
| ','    | Pixel          |
| 'x'    | X              |
| 'X'    | X (filled)     |
| '+'    | Plus           |
| 'P'    | Plus (filled)  |
| 's'    | Square         |
| 'D'    | Diamond        |
| 'd'    | Diamond (thin) |
| 'p'    | Pentagon       |
| 'H'    | Hexagon        |
| 'v'    | Triangle Down  |
| '^'    | Triangle Up    |
| '<'    | Triangle Left  |
| '>'    | Triangle Right |
| '1'    | Tri Down       |
| '2'    | Tri UP         |
| '3'    | Tri Left       |
| '4'    | Tri Right      |
| '\|'   | Vline          |
| '_'    | Hline          |
You can also format the line used to plot points using a parameter called `fmt`, which is written with this syntax: `maker|line|color`.

```python
import matplotlib.pyplot as plt
import numpy as np

ypoints = np.array([3, 8, 1, 10])

plt.plot(ypoints, 'o:r')
plt.show()
```

![](../assets/Pasted%20image%2020260611221550.png)

The marker value can be anything from the marker reference above. The line value can be one of the following:

| Line Syntax | Description          |
| ----------- | -------------------- |
| '-'         | Solid line           |
| ':'         | Dotted line          |
| '--'        | Dashed line          |
| '-.'        | Dashed / dotted line |
If you skip the line value in `fmt` parameter, no line will be plotted.

You can also change the line colour, by using different shorthand like `'r'`  for red, `'g'` for green and so on.

## Customizing Marker
You can use the keyword `markerSize` or the shorter version `ms` to set the size of the markers:

```python
import matplotlib.pyplot as plt
import numpy as np

ypoints = np.array([3, 8, 1, 10])

plt.plot(ypoints, marker = 'o', ms = 20)
plt.show()
```

![](../assets/Pasted%20image%2020260611222026.png)

You can use the keyword `markeredgecolor` or the shorter version `mec` to set the colour of the _edge_ of the markers.

You can use the keyword `markerfacecolor` or the shorter `mfc` to set the colour inside the edge of the markers.

```python
import matplotlib.pyplot as plt
import numpy as np

ypoints = np.array([3, 8, 1, 10])

plt.plot(ypoints, marker = 'o', ms = 20, mec = 'r', mfc = 'r')
plt.show()
```

![](../assets/Pasted%20image%2020260611222159.png)

Even Hexadecimal values are allowed above.

## Customizing line
### Linestyle
You can use the keyword argument `linestyle`, or shorter `ls`, to change the style of the plotted line:
```python
import matplotlib.pyplot as plt
import numpy as np

ypoints = np.array([3, 8, 1, 10])

plt.plot(ypoints, linestyle = 'dotted')
plt.show()
```

![](../assets/Pasted%20image%2020260611223705.png)

You can choose the following linestyles:

| Style             | Or        |
| ----------------- | --------- |
| 'solid' (default) | '-'       |
| 'dotted'          | ':'       |
| 'dashed'          | '--'      |
| 'dashdot'         | '-.'      |
| 'None'            | '' or ' ' |
### Line colour
You can use the keyword argument `color` or the shorter `c` to set the color of the line:
```python
import matplotlib.pyplot as plt
import numpy as np

ypoints = np.array([3, 8, 1, 10])

plt.plot(ypoints, color = 'r')
plt.show()
```
![](../assets/Pasted%20image%2020260611224313.png)

You can also use Hexadecimal colour values.

### Line width
You can use the keyword argument `linewidth` or `lw` to change the width of the line.

```python
import matplotlib.pyplot as plt
import numpy as np

ypoints = np.array([3, 8, 1, 10])

plt.plot(ypoints, linewidth = '20.5')
plt.show()
```

![](../assets/Pasted%20image%2020260611225454.png)

### Multiple lines
You can plot as many lines as you like by simply adding more `plt.plot()` functions:
```python
import matplotlib.pyplot as plt
import numpy as np

y1 = np.array([3, 8, 1, 10])
y2 = np.array([6, 2, 7, 11])

plt.plot(y1)
plt.plot(y2)

plt.show()
```

![](../assets/Pasted%20image%2020260611225554.png)

As we didn't specify any x-coordinate array, matplotlib took them as default value which is [0, 1, 2, 3 ....].

## Labels and Title

With Pyplot, you can use the `xlabel()` and `ylabel()` functions to set a label for the x-axis and y-axis.

With `title()` function you can set a title for the plot.

You can use `fontdict` parameter in `xlabel`, `ylabel` and `title` to set font properties for the title and labels.

`loc` parameter in `title` can be used to set the position of the title. You can use `left`, `right` and `center`. By default `center` is used.

```python
x = np.array([80, 85, 90, 95, 100, 105, 110, 115, 120, 125])
y = np.array([240, 250, 260, 270, 280, 290, 300, 310, 320, 330])

font1 = {'family':'serif','color':'blue','size':20}
font2 = {'family':'serif','color':'darkred','size':15}

plt.title("Sports Watch Data", fontdict = font1, loc="left")
plt.xlabel("Average Pulse", fontdict = font2)
plt.ylabel("Calorie Burnage", fontdict = font2)

plt.plot(x, y)
plt.show()
```

![](../assets/Pasted%20image%2020260612184716.png)


## Grid lines 

You can use the `grid()` function to add grid lines to the plot. You can use a optional parameter to specify which grid lines to display. Use parameter `axis` for it. Legal values for `axis` parameter are `x`, `y` and `both`. Default value is `both`.

You can also set the line properties of the grid, like this: `grid(color = 'color', linestyle= 'linestyle', linewidth=number)`.

```python
x = np.array([80, 85, 90, 95, 100, 105, 110, 115, 120, 125])
y = np.array([240, 250, 260, 270, 280, 290, 300, 310, 320, 330])

plt.title("Sports Watch Data")
plt.xlabel("Average Pulse")
plt.ylabel("Calorie Burnage")

plt.plot(x, y)

plt.grid(axis='both', color = 'green', linestyle = '--', linewidth = 0.5)

plt.show()
```

![](../assets/Pasted%20image%2020260612185008.png)


## Subplot

### Display multiple plots
With the `subplot()` function you can draw multiple plots in one figure.

```python
#plot 1:
x = np.array([0, 1, 2, 3])
y = np.array([3, 8, 1, 10])

plt.subplot(1, 2, 1)
plt.plot(x,y)

#plot 2:
x = np.array([0, 1, 2, 3])
y = np.array([10, 20, 30, 40])

plt.subplot(1, 2, 2)
plt.plot(x,y)

plt.show()
```

![](../assets/Pasted%20image%2020260612190015.png)

### About `subplot()` function
The `subplot()` function takes three arguments that describes the layout of the figure. The layout is organized in rows and columns, which are represented by the _first_ and _second_ argument. The third argument represents the index of the current plot.

```python
plt.subplot(1, 2, 1)
#the figure has 1 row, 2 columns, and this plot is the first plot.

plt.subplot(1, 2, 2)
#the figure has 1 row, 2 columns, and this plot is the second plot.
```

So, if we want a figure with 2 rows and 1 column (meaning that the two plots will be displayed on top of each other instead of side-by-side), we can use the following code:

```python
#plot 1:
x = np.array([0, 1, 2, 3])
y = np.array([3, 8, 1, 10])

plt.subplot(2, 1, 1)
plt.plot(x,y)
plt.title("SALES")

#plot 2:
x = np.array([0, 1, 2, 3])
y = np.array([10, 20, 30, 40])

plt.subplot(2, 1, 2)
plt.plot(x,y)
plt.title("INCOME")

plt.suptitle("MY SHOP")

plt.show()
```

![](../assets/Pasted%20image%2020260612190450.png)

You can also set title for each plot using `title` and for the entire figure using `suptitle`.

## Scatter 

### Creating Scatter plots
We can use the `scatter()` function to draw a scatter plot. The `scatter` function plots one dot for each observation. 

```python
x = np.array([5,7,8,7,2,17,2,9,4,11,12,9,6])
y = np.array([99,86,87,88,111,86,103,87,94,78,77,85,86])

plt.scatter(x, y)
plt.show()
```

![](../assets/Pasted%20image%2020260612191632.png)

We can scatter any number of arrays by `scatter` on the same plot.

```python
x = np.array([5,7,8,7,2,17,2,9,4,11,12,9,6])
y = np.array([99,86,87,88,111,86,103,87,94,78,77,85,86])
plt.scatter(x, y)

x = np.array([2,2,8,1,15,8,12,9,7,3,11,4,7,14,12])
y = np.array([100,105,84,105,90,99,90,95,94,100,79,112,91,80,85])
plt.scatter(x, y)

plt.show()
```

![](../assets/Pasted%20image%2020260612191739.png)

Above we could have also used `subplot` function, to show different scatters in different plots.

We can set our own colour  for the dots using `color` or the `c` argument in `scatter`. You can go as far as specifying colour for each dot by providing a array of colours.

```python
x = np.array([5,7,8,7,2,17,2,9,4,11,12,9,6])
y = np.array([99,86,87,88,111,86,103,87,94,78,77,85,86])
colors = np.array(["red","green","blue","yellow","pink","black","orange","purple","beige","brown","gray","cyan","magenta"])

plt.scatter(x, y, c=colors)

plt.show()
```

![](../assets/Pasted%20image%2020260612192008.png)

### ColorMap

Matplotlib module has a number of available colormaps. A colormap is like a list of colours, where each colour has a value that ranges from 0 to 100.

You can see all the available colormaps available using following:
```python
from matplotlib import colormaps
list(colormaps)
```

### How to use the ColorMap
You can specify the colormap with the keyword `cmap` with the value of the colormap, in this case `virdis` which is one of the built-in colormaps available in matplotlib.

In addition you have to create an array with values (from 0 to 100), one value for each point in the scatter plot.

You can include the colormap in the drawing by including the `plt.colorbar()` statement.

```python
x = np.array([5,7,8,7,2,17,2,9,4,11,12,9,6])
y = np.array([99,86,87,88,111,86,103,87,94,78,77,85,86])
colors = np.array([0, 10, 20, 30, 40, 45, 50, 55, 60, 70, 80, 90, 100])

plt.scatter(x, y, c=colors, cmap='viridis')
  
plt.colorbar()

plt.show()
```

![](../assets/Pasted%20image%2020260612192337.png)

### Customizing dots
You can change the size of the dots with the `s` argument. 

Just like colours, make sure the array for sizes has the same length as the arrays for the x- and y-axis.

You can adjust the transparency of the dots with the `aplha` argument. You also have to give arrays for the dots. 

```python
x = np.random.randint(100, size=(100))
y = np.random.randint(100, size=(100))
colors = np.random.randint(100, size=(100))
sizes = 10 * np.random.randint(100, size=(100))

plt.scatter(x, y, c=colors, s=sizes, alpha=0.5, cmap='nipy_spectral')

plt.colorbar()

plt.show()
```

![](../assets/Pasted%20image%2020260612192525.png)

## Bars

### Creating Bars
With Pyplot, we can use the `bar()` function to draw bar graphs:

```python
x = np.array(["A", "B", "C", "D"])
y = np.array([3, 8, 1, 10])

plt.bar(x,y)
plt.show()
```

![](../assets/Pasted%20image%2020260612221818.png)

You can create horizontal bars using `hbar()`.
![](../assets/Pasted%20image%2020260612222257.png)

You can set colour of the bar using `color` argument.

In case of vertical bars you can increase their width by using `width` argument. For horizontal bars you would use `height`.

The default value of both `height` and `width` is `0.8`.

## Histograms

We can use the `hist()` function to create histograms.

```python
# Below 'normal' means that random numbers will be generated such that they form a normal gaussian distribution. 
# The one that goes up, reaches its peak and than goes back to its lowest from where it started.
x = np.random.normal(170, 10, 250)

plt.hist(x)
plt.show()
```

![](../assets/Pasted%20image%2020260612222617.png)


## Pie charts

### Creating Pie charts
We can use the `pie()` function to draw pie charts.

```python
y = np.array([35, 25, 25, 16])

plt.pie(y)
plt.show()
```

![](../assets/Pasted%20image%2020260612222842.png)

As we can see the pie charts draws one piece (called a wedge) for each value in the array (in this case [35, 25, 25, 15]).

By default the plotting of the first wedge starts from the x-axis and moves _counterclockwise_.

### Labels and start angle
You can give **label** to each wedge by passing in a array for argument to `labels`. This array should be the same size of the array that was created to create the pie chart itself.

By default **start angle** is at the x-axis, but you can change the start angle by specifying a `startangle` parameter.

The `startangle` parameter is defined with an angle in degrees, default angle is 0.

```python
y = np.array([35, 25, 25, 15])
mylabels = ["Apples", "Bananas", "Cherries", "Dates"]

plt.pie(y, labels = mylabels, startangle = 90)
plt.show() 
```

![](../assets/Pasted%20image%2020260612223334.png)

### Explode and Shadow
Maybe we want one of the wedges to stand out? The `explode` parameter allows us to do that.

The `explode` parameter, if specified, and not `None`, must be an array with one value for each wedge.

Each value in that array represents how far from the centre each wedge is displayed.

You can add a shadow to the pie chart by setting the `shadows` argument to `True`.

```python
y = np.array([35, 25, 25, 15])
mylabels = ["Apples", "Bananas", "Cherries", "Dates"]
myexplode = [0.2, 0, 0, 0]

plt.pie(y, labels = mylabels, explode = myexplode, shadow = True)
plt.show() 
```

![](../assets/Pasted%20image%2020260612223711.png)

You can define colours using a array of colours.

### Legend
To add a list of explanation for each wedge, use `legend()` function.

```python
y = np.array([35, 25, 25, 15])
mylabels = ["Apples", "Bananas", "Cherries", "Dates"]

plt.pie(y, labels = mylabels)
plt.legend()
plt.show() 
```

![](../assets/Pasted%20image%2020260612223831.png)

To add a header to the legend, add the `title` argument to the `legend` function.

```python
y = np.array([35, 25, 25, 15])
mylabels = ["Apples", "Bananas", "Cherries", "Dates"]

plt.pie(y, labels = mylabels)
plt.legend(title = "Four Fruits:")
plt.show()
```

![](../assets/Pasted%20image%2020260612223917.png)

