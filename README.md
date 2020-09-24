# Crawling-lines-animation
_How to create animated lines and labels_

I created an animatied line graph with animated labels for a Data Viz Storytelling competition. The data included Denver eviction data from Jan 2019 - June 2019 and Jan 2020 - Aug 2020. I wanted to show the difference between the two years since eviction moratorium was implemented in response to COVID in March 2020 (and exteneded through the end of 2020). 

## Final Image:
Below is my final figure:

![Southard - Show me the data Submission](https://user-images.githubusercontent.com/51967620/94171207-39de5e00-fe4e-11ea-9927-f3fd534c4b5b.gif)

This code was inspired from the following two links
* [Smooth animation with tweenr](https://www.r-graph-gallery.com/287-smooth-animation-with-tweenr.html)
* [Stack Overflow: Labels](https://stackoverflow.com/questions/55723567/how-to-stop-ggrepel-labels-moving-between-gganimate-frames-in-r-ggplot2)

## Set-up and the Data
The data can be downloaded [here](https://observablehq.com/@pstuffa/denver-data-storytellers-dataviz-challenge-data). 
I used the 2 .csv files for Denver 2019 and Denver 2020. 

```
#import libraries
library(readxl)
library(dplyr)
library(ggplot2)
library(gganimate)
library(ggrepel)

#set your working directory to wherever your files are stored.
setwd("C:/Users/lpantlin/Desktop/Other Projects/Eviction Data")

#bring in data
den.filed19   <- read_excel("1. Denver FEDs Filed 20192020.xlsx")
den.filed20   <- read_excel("1.2 Denver FEDs Filed 2020.xlsx")
```
Both data files will appear like this:

![den filed19](https://user-images.githubusercontent.com/51967620/94171697-cd179380-fe4e-11ea-944d-4e1e5e3a3366.PNG)

## Format the data for the visualization

For the figure, I wanted to do the following:
* Pull a part the date into year and month 
** Looking ahead: I want to group by year and have the x-axis be month
* Summarize the counts by month
* Get a cummulative sum for each succeeding month (by year)

I ended up doing this for each file then joining the data back together. There are a lot of ways to do this and probably some better ways, but I think it shows the logic well:

```
d19 <- den.filed19 %>% 
  mutate(date = format(as.POSIXct(`Filing Date`,format='%Y-%m-%d %H:%M:%S'),format='%Y-%m-%d'),
         year = format(as.POSIXct(date,format='%Y-%m-%d'),format='%Y'),
         month = format(as.POSIXct(date,format='%Y-%m-%d'),format='%m')) %>%
  group_by(year, month) %>% 
  summarise(n = n()) %>% 
  mutate(cumsum = cumsum(n)) %>% 
  ungroup() %>% 
  dplyr::select(year, month, cumsum, n) 

d20 <- den.filed20 %>% 
  mutate(date = format(as.POSIXct(Filing_Date,format='%Y-%m-%d %H:%M:%S'),format='%Y-%m-%d'),         
         year = format(as.POSIXct(date,format='%Y-%m-%d'),format='%Y'),
         month = format(as.POSIXct(date,format='%Y-%m-%d'),format='%m')) %>%
  group_by(year, month) %>% 
  summarise(n = n()) %>% 
  mutate(cumsum = cumsum(n)) %>% 
  ungroup() %>% 
  dplyr::select(year, month, cumsum, n) 

#join the data
plot.df <- rbind(d19, d20)
```
The result of the above join will look like this:

![plot df](https://user-images.githubusercontent.com/51967620/94172689-0b618280-fe50-11ea-831b-b4ce77b9f7b7.PNG)

_Note: I used the line of code ```format(as.POSIXct(Filing_Date,format='%Y-%m-%d %H:%M:%S'),format='%Y-%m-%d')``` to strip the timestamp from the date and then tell R what format I wanted as a result. I wanted R to know that both year and month were dates so R would understand how to transition the animation later on._

## Create labels for the animation
First, I filtered out the months that weren't in common across the two years. Then I used ```case_when``` to determine the labels. There are a lot of ways to do this, I created it in a pretty manual way. Happy to hear updates:

```
data <- plot.df %>% 
  filter(as.numeric(month) < 07) %>% 
  mutate(month1 = case_when(month == "01" ~ "Jan",
                            month == "02" ~ "Feb",
                            month == "03" ~ "Mar",
                            month == "04" ~ "Apr",
                            month == "05" ~ "May",
                            month == "06" ~ "June",
                            T ~ month),
         Year = as.factor(year),
         text = case_when((month == "01" && year == "2019") ~ as.character(paste(year, paste(month1, ":", sep = ""), paste("+", n, sep = ""), sep = " ")),
                          (month == "01" && year == "2020") ~ as.character(paste(year, paste(month1, ":", sep = ""), paste("+", n, sep = ""), sep = " ")),
                          (month == "02" && year == "2019") ~ as.character(paste(year, paste(month1, ":", sep = ""), paste("+", n, sep = ""), sep = " ")),
                          (month == "02" && year == "2020") ~ as.character(paste(year, paste(month1, ":", sep = ""), paste("+", n, sep = ""), sep = " ")),
                          (month == "03" && year == "2019") ~ as.character(paste(year, paste(month1, ":", sep = ""), paste("+", n, sep = ""), sep = " ")),
                          (month == "03" && year == "2020") ~ as.character(paste(year, paste(month1, ":", sep = ""), paste("+", n, sep = ""), sep = " ")),
                          (month == "04" && year == "2019") ~ as.character(paste(year, paste(month1, ":", sep = ""), paste("+", n, sep = ""), sep = " ")),
                          (month == "04" && year == "2020") ~ as.character(paste(year, paste(month1, ":", sep = ""), paste("+", n, sep = ""), sep = " ")),
                          (month == "05" && year == "2019") ~ as.character(paste(year, paste(month1, ":", sep = ""), paste("+", n, sep = ""), sep = " ")),
                          (month == "05" && year == "2020") ~ as.character(paste(year, paste(month1, ":", sep = ""), paste("+", n, sep = ""), sep = " ")),
                          (month == "06" && year == "2019") ~ as.character(paste(year, paste(month1, ":", sep = ""), paste("+", n, sep = ""), sep = " ")),
                          (month == "06" && year == "2020") ~ as.character(paste(year, paste(month1, ":", sep = ""), paste("+", n, sep = ""), sep = " ")),
                          T ~ ""))
 ```
The output will look like this:

![data](https://user-images.githubusercontent.com/51967620/94176724-bfb1d780-fe55-11ea-8063-fc5f0686b46d.PNG)

## Some figure parameters

```
# colours for use in plot
condition_colours <- c("grey42", "royalblue")

#so my x-axis labels aren't numbers
labels <-  c("January", "February", "March", "April", "May", "June")

```

## Plot a static background layer
This layer includes anything I wanted to remain the **same** during the transition. You can also define a theme or customize one with ```theme_set()``` , but I had some specifics I wanted to accomplish with this code and I only needed to do it once. Therefore, I put in my theme parameters into ```p1```. 

For purposes of what I was doing, I needed my x-axis limits to go past the months I was plotting so the labels didn't get cut off (hence the extra ticks on the x-axis that have no labels). This is why I used the ```panel.background = element_blank()``` and the ``` axis.line = element_blank()``` arguments. 

```
p1 <- data %>% 
  ggplot(aes(x = as.numeric(month), y = cumsum, colour = Year))+
  scale_y_continuous(expand = c(0, 0), limits = c(min(data$cumsum-200), 5000))+
  scale_x_continuous(limits= c(1, 7), breaks = seq(01, 06, by = 1), 
                     labels = labels)+
  labs(x="Month of Filing", y = "Cumulative Eviction Filings", 
       title = "Denver Eviction Filing Rates Plateau Due to March 2020 Moratorium", 
       fill = "Year"
  )+ theme(axis.line = element_blank(),
           panel.background = element_blank(),
           axis.title.y = element_text(size =16), 
           axis.title.x = element_text(size=  16, hjust = .4),
           axis.text.x = element_text(color = "black", size = 14),
           axis.text.y = element_text(size = 14, color = "black"),
           plot.title = element_text(hjust = 0.5,
                                     size = 18,
                                     face = "bold"),
           text=element_text(size = 14, family = "sans"),
           legend.key = element_blank(),
           legend.position = "top")
  ```
The output will show a base layer like this:

![p1](https://user-images.githubusercontent.com/51967620/94178416-4cf62b80-fe58-11ea-95bf-7349418740e6.png)

## Add a dynamic (animated) layer

```
p2 <- p1 +
  geom_line(data = data, aes(x = as.numeric(month), y = cumsum, colour = Year, group = year),
            size = 1)+ 
  geom_point(data = data, aes(x = as.numeric(month), y = cumsum, colour = Year, group = year),
             size =2)+
  scale_color_manual(values = condition_colours) + 
  geom_segment(data = data, 
               aes(xend = as.numeric(month), yend = cumsum, y = cumsum, colour = Year),
               linetype = 3, show.legend = FALSE) + 
  geom_text(data = data, 
            aes(x = as.numeric(month) + .1, 
                #for my data the lines/points were on top of each other so I wanted to manually nudge them
                # you can also use padding.
                y = case_when((Year == "2019" & month == "01") ~ as.numeric(cumsum+300),
                              (Year == "2020" & month == "01") ~ as.numeric(cumsum+100),
                              (Year == "2019" & month == "02") ~ as.numeric(cumsum+200),
                              T ~ as.numeric(cumsum)),
                Label = text, colour = Year), 
            hjust = 0, size = 4, show.legend = FALSE) +
  #to keep points as we go add the following line of text
  #removing it will give you just the lines
  geom_point(aes(group = seq_along(as.numeric(month)))) +
  #transition_reveal() requires an input that's numeric, date, (and some other options)
  transition_reveal(as.numeric(month)) +
  ease_aes('linear')
```

## Save the image
Render animation using ```anim_save```. Change the speed using nframes and call the animation with the ```animation``` argument.   
```
anim_save(filename = "Eviction-data.gif", nframes =150, animation = p2, end_pause = 5, height = 1000, width = 1250, res = 120) 
```

And this should produce the following .gif file:


![Southard - Show me the data Submission](https://user-images.githubusercontent.com/51967620/94171207-39de5e00-fe4e-11ea-9927-f3fd534c4b5b.gif)
