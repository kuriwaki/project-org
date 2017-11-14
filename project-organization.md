
A Template for Project Organization in the Empirical Social Sciences
--------------

__Shiro Kuriwaki__ (kuriwaki@g.harvard.edu)


> Saying [social science researchers] should spend more time thinking about the way they write code would be like telling a novelist that she should spend more time thinking about how best to use Microsoft Word.

> -- Gentzkow and Shapiro (2014)


This brief memo outlines one way to organize an empirical project in the Social Sciences, an area that is increasingly data-driven, program-driven, and collaborative. I rely most on the excellent writeup by economists Matthew Gentzkow and Jesse Shapiro, "Code and Data for the Social Sciences: A Practitioner’s Guide", as well as other resources that have established best practices. My intent is to summarize the key points and update with new resources, rather than re-invent the wheel.

(prepared for API 201-Z optional session )

# Motivation

Learning how to write on new apps may sound like an optional hobby that "just" makes things look nice (as the opening quote indicates). That is only partially true -- good organization and the use of appropriate tools improve the quality of your work, and more prominently, reduce the number of errors you make, thereby optimizing on your time.

The environment for data analysis and presentation is rapidly increasing. Only recently have we seen:
- Many organizations handling __more data__ at cheaper cost
- Many data analysis and version control tools becoming __open source__
- Online __collaboration__ becoming more prevalent
- Computer science and statistics attracting more students

and with these challenges come new tools to pick up.

Learning these tricks is an investment, but a little training on well-tested best practices can go a long way.


# Original Principles by Gentzkow and Shapiro

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


# Gentzkow and Shapiro,  modified

A version of the above tailored to those whose main job is not research, but data analysis.

## Priniciples
1. Automation
  * Automate everything that can be automated
  * Each script should executes all code from beginning to end
  * Embed Figures and Tables to auto-update in your document
2. Abstraction
  * Abstract to eliminate redundancy.
  * Abstract to improve clarity.
  * Otherwise, don’t abstract.

## Organization
1. Directories (Folder Structure)
  * Separate directories by function
  * Separate files into inputs and outputs
  * Make directories portable
2. Datasets
  * Store cleaned data in tables with unique, non-missing keys
3. Code organization
  * One script for one step (build, analyze, figures)

## Management
6. Documentation
  * Don’t write documentation you will not maintain.
  * Code should be self-documenting.
7. Management
  * Manage tasks with a task management system.
  * E-mail is not _the best_ task management system: Git is the best, with a middle-ground with Dropbox Paper, Tasks in Google Docs, Slack.
  * Use a version control system
8. Open-sourcing
  * Open source your work

## Production
1. Writing Software
  * Use an editor you have the most control over (preferably not WYSIWIG)
2. Figures and Tables
    * One message per figure/table
    * Maximize the data-ink ratio
    * Drop unnnecessary digits
    * Modify aspect-ratios of figures
3. Code style
  * Use descriptive names
  * Don't repeat code more than twice
  * Keep it short and purposeful
  * Make your functions shy
  * Be consistent
  * Write tests
4. Typesetting
  * Use generous margins
  * Use a consistent font
