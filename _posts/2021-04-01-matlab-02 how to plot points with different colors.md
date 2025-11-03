# Plot



## How to plot points with different colors





[Change the code]

```matlab
% Demo to very color of scatterpoints depending on their location
clc;
clear
x = rand(1000,1);
y = rand(1000,1);
distances = ((x-.5).^2 + (y-0.5).^2).^0.5;
% Normalize - divide my sqrt(maxX^2 + maxY^2)
distances = distances / sqrt(.5^2 + .5^2);
[sortedDistances sortIndexes] = sort(distances);
% Arrange the data so that points close to the center
% use the blue end of the colormap, and points 
% close to the edge use the red end of the colormap.
xs = x(sortIndexes);
ys = y(sortIndexes);
cmap = jet(length(x)); % Make 1000 colors.
scatter(xs, ys, 10, cmap, 'filled')
grid on;
title('Points where color depends on distance from center', ...
  'FontSize', 30);
% Enlarge figure to full screen.
set(gcf, 'units','normalized','outerposition',[0 0 1 1]);
% Give a name to the title bar.
set(gcf,'name','Demo by ImageAnalyst','numbertitle','off')
```



![image-20210401152412645](e:\_ygkim\_Doc\_future\\_work\github pages\ygstudio7\ygstudio7.github.io\images\image-20210401152412645.png)



## Reference

https://www.mathworks.com/matlabcentral/answers/119704-how-to-set-for-each-point-a-different-color