---
layout: post
title: "Building Shiny Apps from the Ground Up"
author: "Ben Jefferies"
output:
  md_document:
    variant: markdown_github
    preserve_yaml: true
    toc: yes
---

-   [Introduction](#introduction)
    -   [Learning Objectives](#learning-objectives)
-   [Shiny App Dashboards](#shiny-app-dashboards)
    -   [The Basic Building Blocks](#the-basic-building-blocks)
    -   [Answer Your Data Questions](#answer-your-data-questions)
    -   [Create the Server Side Plot and
        Data](#create-the-server-side-plot-and-data)
    -   [Integrate Plot into Server](#integrate-plot-into-server)
    -   [Advanced Examples](#advanced-examples)
-   [Further Resources](#further-resources)
-   [Works Cited](#works-cited)

<br> <br>

# Introduction

This code through will demonstrate how to create a shiny app in R which
is an interactive application allowing users to explore data through
simple, graphical inputs.

Shiny app dashboards are a great, interactive way to display data to
stakeholders and can help in professional storytelling. Understanding
how the apps work from the ground up is essential knowledge when it
comes to building more complex applications. The processes outlined in
this tutorial should help you streamline your workflow.

<br>

## Learning Objectives

By the end of the tutorial you will know the basics of creating a Shiny
App dashboard. You will be able to…

-   Establish a fluidPage() app outline with a title panel, a sidebar
    panel and a main panel.
-   Create simple plots with hardcoding and then transform them to
    accept dynamic variables from user input.
-   Understand and apply the UI to Server back to UI process flow of a
    Shiny App.
-   Create the app pictured below.

<br>

![](Shiny%20App%20Codethrough%20image.png)

<br> <br>

# Shiny App Dashboards

## The Basic Building Blocks

Shiny Apps have two parts, the UI (User Interface) which the end user
sees and interacts with and the Server side which is where you develop
code for plots and other outputs. The two sides interact with each
other, with user inputs from the UI being sent to the Server and the
output from the server being sent back to the UI.

Because we will be creating a visualization and those often require a
bit of trial and error to get just right, you will want two different R
scripts open for this tutorial. We will start in one which you can name
“App”. The other you can name “vis”.

Here are the most basic building blocks. Copy this code into the app R
script or markdown to follow along.

``` r
library( shiny )

ui <- fluidPage( )   #create ui object

server <- function( input, output ) {}  #create server object

shinyApp(ui = ui, server = server)  #calls ShinyApp with above objects
```

**Note:** *If you run the code above as it is, it already contains
everything to open a function, though totally blank and boring, Shiny
App. Do this regularly throughout the process to catch mistakes as they
**inevitably** happen.*

<br>

## Answer Your Data Questions

For this example we will use a simple data set available within base R,
the mtcars dataset. Our question to answer is: “How are weight and gas
mileage related?” And, we want the user to to be able to select the
number of cylinders in the engine and filter down to just that data.

Now, we ask ourselves, what kind of input will we need from the user and
we can start building our UI. Within the fluidPage call, we will opt for
the sidebarLayout and start to define our inputs within a sidebarPanel.

We want the user to have the option to select either 4, 6, or 8
cylinders, so the radioButtons object will create that choice. More
button option can be quickly idenfitied on the [ShinyApp
cheatsheet](https://shiny.posit.co/r/articles/start/cheatsheet/).

In addition, we will need to define the titlePanel and mainPanel of our
ui display. Calling plotOutput tells the shiny app to put a plot there,
and we are calling it “efficiency” which we will see more about below.

*If you run this code now you can check to ensure that it opens an app
and you haven’t made any mistakes yet.*

``` r
# User Interface with additions
ui <- fluidPage( 
  
  titlePanel("Relationship between MPG and Weight in Select Cars"),
  
  sidebarLayout(
        sidebarPanel(
            radioButtons(inputId = 'cylinders',       
                         label = 'Number of Cylinders:',
                         choices = c(4,6,8),
                         selected = 6)  
        ),
        
        mainPanel(
           plotOutput("efficiency")
        )
  )  
)

#create server object
server <- function( input, output ) {}  

#calls ShinyApp with above objects
shinyApp(ui = ui, server = server)  
```

<br>

## Create the Server Side Plot and Data

Ultimately, we want this app to display a plot. It is good practice to
create the plot you want in another R script first and test it
statically as I have done below filtering for 4 cylinder cars. This is
simply a base to build off.

``` r
subcars <- mtcars[mtcars$cyl == 4, ]

plot(x = subcars$mpg,
     y = subcars$wt,
     pch = 20,
     col = 'red',
     cex = 2,
     xlab = "Miles per Gallon",
     ylab = "Weight in Tons")
```

![](Shiny-CodeThrough-Jefferies_files/figure-markdown_github/unnamed-chunk-4-1.png)

**Note:** *You will want to play around and develop your plot in this
other r script using hard coded variables first, make sure it works and
then we can copy it over to our ShinyApp to replace variables with the
user generated inputs.*

<br>

## Integrate Plot into Server

Now we are looking at the server created object in our Shiny App R
script. Notice that all we have done below is expanded the {} brackets
and began by establishing the output$effeciency object as a result of
the renderPlot({}) call. This tells our app that we will render a plot
and it will be stored in the variable “output” with the name efficiency.
The UI part of our app has already been told in the mainPanel() to look
for “efficiency” in the plotOutput() call.

Within the renderPlot({}) call we simply paste in our plot from our plot
R file. Then, we need to make the number of cylinders variable and
dependent on the user input in our ui section. Below, you can see the
only change is in the subcars \<- call where we replaced the hardcoded 4
with input$cylinders which we got from the inputID = “cylinders” within
our sidePanel call.

``` r
# Looking at the server section isolated from ui
server <- function(input, output) {

    output$efficiency <- renderPlot({
      
      subcars <- mtcars[mtcars$cyl == input$cylinders, ]

      plot(x = subcars$mpg,
           y = subcars$wt,
           pch = 20,
           col = 'red',
           cex = 2,
           xlab = "MPG",
           ylab = "Weight in Tons")
      
    })
}
```

Here is all of the code in one panel which you can copy and run. Compare
this to your own code if you are experimenting and you get any errors.

``` r
ui <- fluidPage( 
  
  titlePanel("Relationship between MPG and Weight in Select Cars"),
  
  sidebarLayout(
        sidebarPanel(
            radioButtons(inputId = 'cylinders',       
                         label = 'Number of Cylinders:',
                         choices = c(4,6,8),
                         selected = 6)  
        ),
        
        mainPanel(
           plotOutput("efficiency")
        )
  )  
)


server <- function(input, output) {

    output$efficiency <- renderPlot({
      
      subcars <- mtcars[mtcars$cyl == input$cylinders, ]

      plot(x = subcars$mpg,
           y = subcars$wt,
           pch = 20,
           col = 'red',
           cex = 2,
           xlab = "MPG",
           ylab = "Weight in Tons")
      
    })
}

#calls ShinyApp with above objects
shinyApp(ui = ui, server = server)  
```

<br>

## Advanced Examples

**(A)** One can make plots much more complicated with added features.
Below is code producing an app which also regresses a linear model onto
the resulting relationship between mpg and weight.

``` r
ui <- fluidPage( 
  
  titlePanel("Relationship between MPG and Weight in Select Cars"),
  
  sidebarLayout(
        sidebarPanel(
            radioButtons(inputId = 'cylinders',       
                         label = 'Number of Cylinders:',
                         choices = c(4,6,8),
                         selected = 6)  
        ),
        
        mainPanel(
           plotOutput("efficiency")
        )
  )  
)


server <- function(input, output) {

    output$efficiency <- renderPlot({
      
      subcars <- mtcars[mtcars$cyl == input$cylinders, ]

      plot(x = subcars$mpg,
           y = subcars$wt,
           pch = 20,
           col = 'red',
           cex = 2,
           xlab = "MPG",
           ylab = "Weight in Tons")
      
      lm <- lm(wt ~ mpg, subcars)
      
      abline(a = lm$coefficients[1],
       b = lm$coefficients[2],
       col = 'gray20')
      
      slope <- as.character(round(lm$coefficients[2], 2))
      
      myText <- paste('For 1 mpg increase, weight changes', slope, 'tons.')
      
      mtext(text = myText,
            side = 3)
      
    })
}

#calls ShinyApp with above objects
shinyApp(ui = ui, server = server)  
```

<br>

**(B)** Another example with some much more complex user inputs is
below. You can create this exact app following along with [Lisa
Lendway’s video](https://www.youtube.com/watch?v=ak_NJCVrJXY)

![](Babynames%20App.png)

``` r
library(shiny)
library(tidyverse)
library(babynames)

ui <- fluidPage(textInput(inputId = "name",
                          label = "Name:",
                          value = "",
                          placeholder = "Ben"),
                selectInput(inputId = "sex",
                            label = "Sex:",
                            choices = list(Male = "M", 
                                           Female = "F")),
                sliderInput(inputId = "year",
                            label = "Year Range:",
                            min = min(babynames$year),
                            max = max(babynames$year),
                            value = c(min(babynames$year),
                                      max(babynames$year)),
                            sep = ""),
                
                submitButton(text = "Apply Changes"),
                
                plotOutput(outputId = "namePlot"),
                
                )

server <- function(input, output) {
  
  output$namePlot <- renderPlot(
    babynames %>% 
      filter(sex == input$sex,
             name == input$name) %>% 
      ggplot(aes(x=year, y=n)) +
      geom_line() +
      scale_x_continuous(limits = input$year) +
      theme_minimal()
  )
  
}

shinyApp(ui = ui, server = server)
```

<br>

# Further Resources

Learn more about Shiny Apps with the following:

<br>

-   Joe Cheng (2020). Mastering Shiny. [Mastering
    Shiny](https://hadley.github.io/mastering-shiny/your-first-shiny-app.html#exercises)

    *A thorough and lengthy look into all aspects of Shiny.*

-   Edinburgh University.
    [OurCodingClub](https://ourcodingclub.github.io/tutorials/shiny/)

    *A short and concise tutorial with another great example project.*

<br> <br>

# Works Cited

This code through references and cites the following sources:

<br>

-   Lisa Lendway (2020). YouTube. [Creating Your First Shiny App in
    R](https://www.youtube.com/watch?v=ak_NJCVrJXY)

-   Garrett Grolemund (2021). Shiny for R Cheatsheet.
    [Cheatsheet](https://shiny.posit.co/r/articles/start/cheatsheet/)
