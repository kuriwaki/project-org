# Best Practices for Project Organization in the Empirical Policy and Social Science Analhysis
Shiro Kuriwaki (kuriwaki@g.harvard.edu)  
`r Sys.Date()`  





> Saying [social science researchers] should spend more time thinking about the way they write code would be like telling a novelist that she should spend more time thinking about how best to use Microsoft Word.
> -- Gentzkow and Shapiro (2014)


This brief memo outlines one way to organize an empirical project in the Social Sciences, an area that is increasingly data-driven, program-driven, and collaborative. I rely most on the excellent writeup by economists Matthew Gentzkow and Jesse Shapiro, "Code and Data for the Social Sciences: A Practitioner’s Guide", as well as other resources that have established best practices. My intent is to summarize the key points and update with new resources, rather than re-invent the wheel.

(prepared for API 201-Z optional session )

## Motivation

Learning how to write on new apps may sound like an optional hobby that "just" makes things look nice (as the opening quote indicates). That is only partially true -- good organization and the use of appropriate tools improve the quality of your work, and more prominently, reduce the number of errors you make, thereby optimizing on your time.

The environment for data analysis and presentation is rapidly increasing. Only recently have we seen:
- Many organizations handling __more data__ at cheaper cost
- Many data analysis and version control tools becoming __open source__
- Online __collaboration__ becoming more prevalent
- Computer science and statistics attracting more students

and with these challenges come new tools to pick up.

Learning these tricks is an investment, but a little training on well-tested best practices can go a long way.


## Original Principles by Gentzkow and Shapiro

The recommendations by Gentzkow and Shapiro are central:

1. Automation
   * Automate everything that can be automated
  * Write a single script that executes all code from beginning to end
2. Version Control
   * Store code and data under version control
   * Run the whole directory before checking it back in
3. Directories (Folder Structure)
   * Separate directories by function
   * Separate files into inputs and outputs
   * Make directories portable
4. Keys
   * Store cleaned data in tables with unique, non-missing keys
   * Keep data normalized as far into your code pipeline as you can.
5. Abstraction
   * Abstract to eliminate redundancy.
   * Abstract to improve clarity.
   * Otherwise, don’t abstract.
6. Documentation
   * Don’t write documentation you will not maintain.
   * Code should be self-documenting.
7. Management
   * Manage tasks with a task management system.
   * E-mail is not a task management system.
8. Code style
   * Keep it short and purposeful
   * Make your functions shy
   * Order your functions for linear reading
   * Use descriptive names
   * Pay special attention to coding algebra
   * Make logical switches intuitive
   * Be consistent
   * Check for errors
   * Write tests
   * Profile slow code relentlessly.
   * Store “too much” output from slow code
   * Separate slow code from fast code


## Gentzkow and Shapiro,  modified

A version of the above tailored to those whose main job is not research, but data analysis.

### Priniciples

1. Automation
   i) Automate everything that can be automated
   ii) Each script should executes all code from beginning to end
   iii) Embed Figures and Tables to auto-update in your document

2. Abstraction
   i) Abstract to eliminate redundancy.
   ii) Abstract to improve clarity.
   iii) Otherwise, don’t abstract.

### Organization

3. Directories (Folder Structure)
   i) Separate directories by function
   ii) Separate files into inputs and outputs
   iii) Make directories portable

4. Datasets
   i) Store cleaned data in tables with unique, non-missing keys

5. Code organization
   i) One script for one step (build, analyze, figures)

### Management

6. Documentation
   i) Don’t write documentation you will not maintain.
   ii) Code should be self-documenting.

7. Management
   i) Manage tasks with a task management system.
   ii) E-mail is not _the best_ task management system.
   iii) Use a version control system

8. Open-sourcing
   i) Consider Open-sourcing your datasets, figures, or code

### Production
9. Writing Software
   i) Use an editor you have the most control over (preferably not WYSIWIG)

10. Figures and Tables

   i) One message per figure/table
   ii) Maximize the data-ink ratio
   iii) Drop unnnecessary digits
   iv) Modify aspect-ratios of figures

11. Code style
   i) Use descriptive names
   ii) Don't repeat code more than twice
   iii) Keep it short and purposeful
   iv) Make your functions shy
   v) Be consistent
   vi) Write tests

12. Typesetting
   i) Use generous margins
   ii) Use a consistent font



## Automation

   i) Automate everything that can be automated
   ii) Each script should executes all code from beginning to end
   iii) Embed Figures and Tables to auto-update in your document


You will be revising your analysis and writeup many more times than you anticipate. That's why short and automated code is essential. 

For example, these lines of code reads in data from Google Sheets, formats it, and loads the necessary packages, and saves a figure of a printable size. 


```r
library(googlesheets)

gapminder_sheet <- gs_gap()
gap_africa <- gs_read(gapminder_sheet, ws = "Africa", col_types = cols())

glimpse(gap_africa)

gap_africa$log_gdppercap <- log(gap_africa$gdpPercap)

ggplot(gap_africa, aes(x = log_gdppercap, y = lifeExp, size = pop)) +
  facet_wrap(~year, nrow = 2) +
  geom_point() +
  geom_label(data = filter(gap_africa, country %in% "Rwanda"), aes(label = country)) +
  guides(size = FALSE) +
  labs(x = "log GDP per Capita", y = "Average Life Expectancy")

ggsave("figures/gapminder_africa.pdf", width = 5, height = 3)
```





Then, an text editor like Rmarkdown or LaTeX compiles a plain text file that embeds this Figure.

```sh
Project Report
============
Shiro Kuriwaki

# Introduction

# Data


# Analysis
![Rwanda's Civil War led to a substnantial Drop in its Life Expectancy](figures/gapminder_africa.pdf)

```


# Abstraction

   i) Abstract to eliminate redundancy.
   ii) Abstract to improve clarity.
   iii) Otherwise, don’t abstract.


It is also normal for you to try something out and then do another version of that. Suppose that, in analyzing Brazilian municipality data, we want to generate a new variable that computes the difference in a municipality's literacy level with its state-average. In Stata you would try


```sh
 egen mean_literate91 = mean(literate91), by(state)
 generate relative_lit_state = mean_literate91  - literate
```

and then look at a histogram,

```sh
histogram relative_lit_state
```


Intrigued, suppose you want to do the same type of calculation but with a different variable, the poverty rate. A redundant but common next step is to copy-paste the two lines of code and make sure to replace the relevant parts:



```sh
 egen mean_poverty91 = mean(poverty91), by(state)
 generate relative_pov_state = mean_poverty91  - poverty91
 histogram relative_pov_state
```


What about doing the same thing not by state, but by region? And so on and so on.. you end up with lines of code that is redundant in terms of the key operation (compute means, subtract the mean from the value); only the variables have changed. After a while, you get tired of copy-pasting and tediously editing so you stop analyzing the data and move out. Even worse, you make a copy-paste error and forget to swap a variable. 

Here's where the abstraction via defining your own function becomes important:

```sh
program hist_relative_rate
         syntax, invar(varname) outvar(name) byvar(varname)
         tempvar mean_invar
         egen `mean_invar'= mean(`invar'), by(`byvar')
         gen `outvar' = `mean_invar' - `invar'
         histogram `outvar'
end
```

Now we can do the same function with only one line of code!

```sh
hist_relative_rate, invar(poverty80) outvar(relative_pov80_state) byvar(state)
```

Moreover, repeating the same procedure with different variables requires you only write as many lines as there are different specifications, rather than having the extra `egen`. 


```r
hist_relative_rate, invar(poverty80) outvar(relative_pov80_state) byvar(state)
hist_relative_rate, invar(poverty91) outvar(relative_pov91_state) byvar(state)
hist_relative_rate, invar(poverty80) outvar(relative_pov80_region) byvar(region)
```


Not only is this fewer keystrokes, it is much more **readable**. It is clear what the function `hist_relative_rate` is essentially doing, whereas understanding three lines takes more time than people have time for. We'll see later that adding comments to this code is also not a great idea, because that documentation can quickly become deprecated.


# Directories

   i) Separate directories by function
   ii) Separate files into inputs and outputs
   iii) Make directories portable
   

Directories are just a computer engineering name for folders. Being told how to organize folders may seem pedantic at first, but this habit-forming practice is pretty consequential for all the work that you do later. And especially if you are collaborating with others, using well-established practices save you a lot of confusion and errors. 

Copying directly from Gentzkow and Shapiro,

![A template for your project folder](figures/GS_directory.png)

notice that

1. There is one "project directory", which may be a Dropbox folder for a shared project,
2. That folder has `build` and `analyze` folders, which are verbs, not nouns,
3. Within each folder the subfolders are exactly parallel
4. The sequence is clear: build, then analyze. Input, then output.
5. Anything in `input` **should not be overwritten**
6. Anything in `output` should be **reproducible by the code**


Whether or not to maintain a strict difference between `build` and `analyze`, even to the point of having a separate `data` subfolder for each, can be debatable. For example my directory for one project looks like:

![data directory divided into input and output](figures/directory_example.png)

Breaking out of old habits are hard, but give this a try. The hope is that this will become natural as you use it.


## Datasets

Building your own dataset is an intermediate/advanced-level skill. But it is the bread and butter of data analysis in the type of work Gentzkow and Shapiro do. A lot of the power in data analysis comes from merging, and setting "keys" -- unique identifiers is a principle that makes this possible. 

The concept of keys is also critical when discussing data because it is the database equivalent of asking what your **unit of analysis** is.

For example, in the gapminder dataset, what is the key?

```r
gap_africa
```

```
## # A tibble: 624 x 7
##    country continent  year lifeExp      pop gdpPercap log_gdppercap
##      <chr>     <chr> <int>   <dbl>    <int>     <dbl>         <dbl>
##  1 Algeria    Africa  1952  43.077  9279525  2449.008      7.803438
##  2 Algeria    Africa  1957  45.685 10270856  3013.976      8.011015
##  3 Algeria    Africa  1962  48.303 11000948  2550.817      7.844169
##  4 Algeria    Africa  1967  51.407 12760499  3246.992      8.085484
##  5 Algeria    Africa  1972  54.518 14760787  4182.664      8.338704
##  6 Algeria    Africa  1977  58.014 17152804  4910.417      8.499114
##  7 Algeria    Africa  1982  61.368 20033753  5745.160      8.656113
##  8 Algeria    Africa  1987  65.799 23254956  5681.359      8.644946
##  9 Algeria    Africa  1992  67.744 26298373  5023.217      8.521826
## 10 Algeria    Africa  1997  69.152 29072015  4797.295      8.475808
## # ... with 614 more rows
```
Note that in this case you need a key of two variables to uniquely identify a row -- `country` and `year`


Large databases (e.g. in `SQL`) are organized in this way too -- each entry in a table (which can be tens of millions of rows) is identifiable by a unique identifier^[Ansolabehere and Hersh (2017), ``ADGN: An Algorithm for Record Linkage Using Address, Date of Birth, Gender and Name''. _Statistics and Public Policy_. http://amstat.tandfonline.com/doi/abs/10.1080/2330443X.2017.1389620#.WgvTIxOPLuA].


## Code

Long code (about ~300 lines or more) can get hard to scroll through. It probably makes your life easier to split your code up into separate parts. (`read`, `clean`, `deduplicate`, `merge`, for example.)


## Documentation

   i) Don’t write documentation you will not maintain.
   ii) Code should be self-documenting.


A common misperception is that one needs to annotate (a.k.a. "comment") code extensively to be helpful. While this is good in theory, it almost always fails to come through in practice. Analysts and programmers are humans, after all -- we fix typos in place without updating the documentation, we make changes on a whim and forget to get back to them, we copy and paste code from elsewhere along with the irrelevant comments, etc.. The lesson is to _embrace_ and adjust your code to this tendency, rather than setting yourself up for unrealistic levels of fastiduousness.

The Gentzkow and Shapiro example is excellent. What's wrong with this Stata code?


```sh
* Elasticity = Percent Change in Quantity / Percent Change in Price
* Elasticity = 0.4 / 0.2 = 2
* See Shapiro (2005), The Economics of Potato Chips,
* Harvard University Mimeo, Table 2A.
 compute_welfare_loss, elasticity(2)
```

These four lines of notes are informative, but three days later after multiple stages of editing, you're bound to see something like this:


```sh
* Elasticity = Percent Change in Quantity / Percent Change in Price
* Elasticity = 0.4 / 0.2 = 2
* See Shapiro (2005), The Economics of Potato Chips,
* Harvard University Mimeo, Table 2A.
  compute_welfare_loss, elasticity(3)
```

Notice the elasticity in the notes are inconsisit with the ones in the values. Changing values like this happens out, but after a couple of days you won't remember whether your comment or your code is the newer version, and you are stuck.



```sh
* See Shapiro (2005), The Economics of Potato Chips,
* Harvard University Mimeo, Table 2A.
local percent_change_in_quantity = -0.4
local percent_change_in_price = 0.2
local elasticity = `percent_change_in_quantity'/`percent_change_in_price'
compute_welfare_loss, elasticity(`elasticity')
```

This is much better, because ``it has far less scope for internal inconsistency. You can’t change the percent change in quantity without also changing the elasticity, and you can’t get a different elasticity number with these percent changes.''


## Task and Version Management

   i) Manage tasks with a task management system.
   ii) E-mail is not _the best_ task management system.
   iii) Use a version control system

``E-mail is not a task management system''. There are plenty of Apps now that allow you to tag individuals with tasks, making individual responsibilities clear. Just to name a few: Asana, Slack, Dropbox Paper, Basecamp, Google Docs, ...


A version control system, typically on Github, is a great way to keep track of your files without being overloaded with versions and versions of confusingly named files. This may be an advanced topic for an introduction to data analysis, but tutorials like https://try.github.io/ is a good start, and learning as you go is a valuable skillset.



## Open-sourcing

Data analytics -- both the datasets and the programs and functions that analyze them -- is increasingly open source. Posting your own code and findings on the public web is daunting at first, but it has a couple of benefits. Primarily, you learn how to code and analyze data through the best possible ways -- seeing others more advanced than you actually write code for the same type of problems.


##  Writing Software
   i) Use an editor you have the most control over (try something not WYSIWIG)
   
WYSIWIG stands for "What you see is what you get", and software like Microsoft Office and Google Docs fall in this category. A non-WYSIWIG editor requires you to explicitly control your formatting (such as boldface, font size, margin size, Figure placement, captioning, etc..), often in plain-text. Plain text is just what it sounds like -- text files that have no formatting in them. Although they have different file extensions, `.Rmd`, `.do`, `.R`, is plain text.

A good, "lightweight" non-WYSIWIG language is Markdown. The best interface for using Markdown with data I know is RMarkdown. That's how I made this document. To make my slides and notes, I also use a verison of Rmarkdown with a more involved typesetting engine called `TeX` underneath it. 
   
   

## Figures and Tables

   i) One message per figure/table
   ii) Maximize the data-ink ratio
   iii) Drop unnnecessary digits
   iv) Modify aspect-ratios of figures
   
As you saw in the Anscombe dataset problem, making visualizations is key. In some disciplines it is not at all controversional to skim a paper or report by just reading the figures and tables, skipping the text.

## Code style
   i) Use descriptive names
   ii) Don't repeat code more than twice
   iii) Keep it short and purposeful
   iv) Make your functions shy
   v) Be consistent
   vi) Write tests


Small things can help you focus on the more important things.  Karl Broman's writeup of this, focused on spreadsheet data, is excellent. https://github.com/kbroman/Paper_DataOrg/blob/master/manuscript.md

Once you decide on a capitalization method, stick to it. Don't name one thing `FigureAfrica.pdf`, another file `Asia_Figure`, and then another `fig-europe`. Stay consistent in your use of capitalization, ordering of terms, and uses of dashes vs. underscores.

For files starting with number for ordering, consider padding a `0` in front of single digits so that `09_file.R` comes before `10_file.R`, not after.

It's also good to stick to a consistent way of referring to dates in your filenames and code, and preferably a standard one. As Karl Broman writes, "If sometimes you write `8/1/2015`
and sometimes `8-1-15`, it will be more difficult to use the dates in
analyses or data visualizations."

![](https://imgs.xkcd.com/comics/iso_8601.png)


##  Typesetting
   i) Use generous margins
   ii) Use a consistent font
