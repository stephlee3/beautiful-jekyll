---
layout: post
title: Interesting Residual Plots
permalink: /posts/residual-plots
---
<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>


Consider you are fitting a linear regression on the given dataset, and draw the scatter plot of *residuals* vs *predicted values*. However, you might feel confused when you get the following interesting plots.  

<center>
<img src="/img/blog_img/simpson_plt.png">
</center>

How to make a plot like this??  The key idea is just shown in the above picture: the projection matrix in the regression model. 

Assume we want to fit a linear model: 
$$y= X\beta + \epsilon$$. 


