# Understanding Data Visualization

## VISUALIZING DISTRIBUTIONS

3 ways of getting insights:

- calculating summary statistics (mean, median, std)
- running models (linear and logistic regression)
- drawing plots

Types of variables:

- Continuous: usually numbers
  - heights, temperatures, revenues
- Categorical: usually text
  - eye color, countries, industry
- Can be either:
  - age is continuous, but age _group_ is categorical
  - time is continuous, month of year is categorical

### Histograms

Use when ...:

- You have single continuous variable
- You want to ask questions about the shape of its distribution

### Box plots

Use when ...:

- You have a continuous variable, split by a categorical variable
- You want to compare the distributions of the continuous variable for each category


---

## VISUALIZING TWO VARIABLES

### Scatter plots

Use when ...:

- You have two continuous variables
- You want to answer questions about the relationship between the two variables

### Line plots

Use when ...:

- You have two continuous variables
- You want to answer questions about their relationship
- Consecutive observations are connected somehow (like date/time)

### Bar plots

Use when ...:

- You have a categorical variable
- You want counts or percentages for each category
- Occasionally: you want another numeric score for each category, and need to include zero in the plot.

### Dot plots

Use when ...:

- You have a categorical variable
- You want to display numeric scores for each category on a log scale, OR
- You want to display _multiple_ numeric scores for each category

---

## THE COLOR AND THE SHAPE

### Shape

### Color
- The colorspace used for data viz is HCL, Hue-Chroma-Luminance
  - Hue: color
  - Chroma: intensity (going from grey to the color of the hue)
  - Luminance: brightness (black -> color -> white)

#### Qualitative color scale

- Used to distinguish unordered categories
- change the hue

#### Sequential color scale

- Used to show ordering
- change chroma or luminance
- veridis color scale is designed to be easily read by colorblind
  and to print well in black and white.

#### Diverging color scale 

- Used to show above or below a midpoint
- change chroma or luminance, and use two different hues
  - Ex (changing luminance): magenta -> white -> green
    - white would be the midpoint

### Plotting many variables at once

#### Pair plot

Use when ...:

- you have up to 10 variables (continuous, categorical, mix)
- you want to see the distribution for each variable
- you want to see the relationship between each pair of variables

##### Interpreting

- Panels on the diagonal show distributions of the variables
  - categorical vars are represented as bar plot of counts
  - continous vars are represented by histograms
- When both vars are continuous you see scatter plots of each pair of variables and their correlation
- When comparing a categorical variable to a continuous var you get a box plot and histogram

#### Correlation heatmap

Use when ...:

- you have lots of continuous variables
- you want a simple overview of how each pair is related

#### Parallel coordinates

Use when ...:

- you have lots of continuous vars
- you want to find patterns accross these vars OR
- you want to visualize clusters of observations

---

## Plots

### Polar coordinates

- When should you use it? **ALMOST NEVER**
- Only use it if you have a variable that is naturally circular
  - time of day
  - compass direction

#### Pie plot

- looks like a pie chart
- bar plot + polar coords = pie plot

#### Rose plot

- histogram + polar coords = rose plot

### Axes

- Changing the axis range can lead to misleading viz
  - not including zero in the y axis can distort the relative heights of bars

### Visual overstimulation

Good viz can be measured by:

- how many interesting insights can your reader get from the plot
- how quickly can they get those insights

> Chartjunk:
> Any element of the plot that distracts the reader from getting insight

- pics
- skueomorphism: refelctions, shadows, etc.
- extra dims
- ostentatious colors/lines

It is better to create a dashboard with different simpler plots than to pick
one "perfect plot" that answers everything.

