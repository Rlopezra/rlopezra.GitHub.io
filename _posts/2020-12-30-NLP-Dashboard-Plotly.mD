---
layout: single
classes: wide
title: "A NLP Dashboard of Hotel Review using Plotly"
date: 2020-12-30
tags:
  - Python
  - NLP
  - Plotly
  - Dashboard
  - Visualization
comments: true
---

I created a dashboard analysing the [Kaggle 515K Hotel Reviews Data in Europe](https://www.kaggle.com/jiashenliu/515k-hotel-reviews-data-in-europe) dataset 
using Natural Language Processing (NLP) techniques. The dashboard was created using Plotly and is being hosted by PythonAnywhere.

[You can view the dashboard by following this link](http://rlopezra.pythonanywhere.com/)

[You can view the source code on my Github](https://github.com/Rlopezra/Hotel-Reviews-Plotly-Dashboard)

The dashboard has two components, the first compares the most frequently used and most important bigrams between two nationalities. The second component visualizes a Word2Vec word embedding in two dimensions using t-SNE.
## Word Frequencies

I created the word frequencies using the TidyText approach, [I had to created my own function to do it](https://rlopezra.github.io/TidyText-Approach-in-Python/), to visualize the most frequently used and most important bigrams for each nationality in their hotel reviews. Additionally, I'm using bigrams (2 words) instead of tokens (1 word) since they are more informative and provide more contextual information. 

I also created a scatterplot that compares how frequently a token is used by each nationality. The way to interpret this graph is words that are along the diagonal line are equally used by each nationality. The further the token deviates from the diagonal line, the more it used by a nationality. In our example, words that are above the diagonal line are more frequently used by Belgiums.


## Visualizing Word Embeddings
To be written

## What I Learned
This project started with the goal of familiarizing myself with Plotly’s Dash platform and being comfortable creating a complex dashboard with multiple callbacks. Even though that goal was met, what I got the most from this project was the knowledge of how to host the dashboard. It was pretty easy and straightforward to build the dashboard due to great [documentation]( https://dash.plotly.com/) and the numerous public [apps]( https://dash-gallery.plotly.host/Portal/) whose code is available. Any proficient Python programmer could build a comparable dashboard in less than a day.

For me, what I learned the most from this project is having to learn on the spot how to use Bash, create a virtual environment, check for dependencies and how to read error logs. I’m confident that I spent more time figuring out how to get the dashboard deployed than I did creating it. Additionally, I also had to learn to adapt my design. I had to change the layout and functionality of my original design since PythonAnywhere does not support multi-threading. Because of this, my original design took a good minute to boot up. I had to learn how to make my code more efficient and pre-process some of the tasks to speed up loading the dashboard. 
