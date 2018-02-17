Writing Good Code for Data Processing
===
author: Jack Brookes
date: February 2018
width: 1280
height: 720
font-import: http://fonts.googleapis.com/css?family=Roboto+Condensed|Roboto+Mono
font-family: 'Roboto Condensed'
incremental: true
css: custom.css
  
Good code
===
My definition...
- Does it's job
- Works in different contexts
- Easily understood
- Easily modified by colleague, or yourself in the future
- Efficient

Balance of trade-offs

Example: data
===
incremental: false

CSV file: Columns are participant ID, gender, year of birth, and performance in a task


|p     |female |    y|         x|
|:-----|:------|----:|---------:|
|P_001 |FALSE  | 1957| 51.645948|
|P_002 |FALSE  | 1980| 56.726900|
|P_003 |FALSE  | 1996|  7.394432|
|P_004 |FALSE  | 1964| 40.092979|
|P_005 |TRUE   | 1988| 12.364888|
|P_006 |TRUE   | 1968|  6.438655|
|P_007 |FALSE  | 1959| 14.990385|
|P_008 |TRUE   | 1968|  7.856301|
|P_009 |TRUE   | 1979| 24.247090|
|P_010 |FALSE  | 1986| -2.279847|
|P_011 |TRUE   | 1980| 41.053064|
|P_012 |FALSE  | 1976| 20.728537|
|P_013 |FALSE  | 1987| 34.522459|

Example code
===
incremental: false

```r
data <- read_csv("participant_performance.csv")
data =data %>%
  filter(((x-mean(x))/sd(x) < 3) & ((x-mean(x))/sd(x) > -3))
data$a <- as.integer(format(Sys.Date(), "%Y"))-data$y

data2 <- filter(data, female == F)
```


Add Comments?
===
incremental: false


```r
# read data
# columns are:
# p: Participant ID, y: Year of birth, x: Task performance
data <- read_csv("participant_performance.csv") 
# remove if performance z-score > 3 or < -3
data =data %>%
  filter(((x-mean(x))/sd(x) < 3) & ((x-mean(x))/sd(x) > -3))
# calculate (rough) age using current year
data$a <- as.integer(format(Sys.Date(), "%Y"))-data$y

# data with only males
data2 <- filter(data, female == F)
```

Comments
===

![Push?](images/push.jpg)

***

- Comments can't fix the code
- Comments get left outdated 
- Takes extra time (both reading and writing)
- Think about:
  - What aspect of my code is this comment improving?
  - Can the code be changed to remove the need for a comment?


Naming / Abstraction
===
incremental: false
At the top of our script...

```r
# CONSTANTS

z_cutoff <- 3

current_year <- Sys.Date() %>% 
  format("%Y") %>% 
  as.integer()


# FUNCTIONS

zscore <- function(x){
  (x - mean(x)) / sd(x)
}
```

Naming
===
incremental: false


```r
# READ RAW DATA
perf_data_raw <- read_csv("participant_performance.csv") %>% 
  rename(participantID = p, is_female = female, year_of_birth =y,performance = x) %>%
  mutate(gender = ifelse(is_female, "Female", "Male")) %>% 
  select(-is_female) #remove redundant column

# PREPROCESS
perf_data =perf_data_raw %>%
  filter(abs(performance_z) < z_cutoff) 

perf_data$age <- current_year-perf_data$year_of_birth
perf_Males <- filter(data, gender == "Male")
```
* **P**rinciple **O**f **L**east **A**stonishment
* **D**ont **R**epeat **Y**ourself principle

Consistency & Formatting
===
incremental: false

Keep a consistent style e.g. [Tidyverse style guide](http://style.tidyverse.org/), [Google's R style guide](https://google.github.io/styleguide/Rguide.xml)

```r
# READ RAW DATA
perf_data_raw <- read_csv("participant_performance.csv") %>% 
  rename(participant_id = p,
         is_female = female,
         year_of_birth = y,
         performance = x) %>%
  mutate(gender = ifelse(is_female, "Female", "Male")) %>% 
  select(-is_female) #remove redundant column

# PREPROCESS
perf_data <- perf_data_raw %>%
  mutate(performance_z = zscore(performance)) %>%
  filter(abs(performance_z) < z_cutoff) %>% 
  mutate(age = current_year - year_of_birth)

perf_males <- perf_data %>%
  filter(gender == "Male")
```


Big payoff now our script and data is "tidy"
===
incremental: false


|participant_id | year_of_birth| performance|gender | performance_z| age|
|:--------------|-------------:|-----------:|:------|-------------:|---:|
|P_001          |          1957|   51.645948|Male   |     1.7931049|  61|
|P_002          |          1980|   56.726900|Male   |     2.1279487|  38|
|P_003          |          1996|    7.394432|Male   |    -1.1231498|  22|
|P_004          |          1964|   40.092979|Male   |     1.0317434|  54|
|P_005          |          1988|   12.364888|Female |    -0.7955878|  30|
|P_006          |          1968|    6.438655|Female |    -1.1861372|  50|
|P_007          |          1959|   14.990385|Male   |    -0.6225628|  59|
|P_008          |          1968|    7.856301|Female |    -1.0927118|  50|
|P_009          |          1979|   24.247090|Female |    -0.0125292|  39|
|P_010          |          1986|   -2.279847|Male   |    -1.7607022|  32|
|P_011          |          1980|   41.053064|Female |     1.0950147|  38|
|P_012          |          1976|   20.728537|Male   |    -0.2444082|  42|
|P_013          |          1987|   34.522459|Male   |     0.6646361|  31|
|P_014          |          1992|    7.856127|Female |    -1.0927232|  26|


Programming principles and tips
===
type: sub-section

Abstraction & levels of complexity
=== 

- Human-like description
- Morning routine:
- Don't:
  - Open eyes
  - Move arm 52.8cm towards alarm clock
  - Depress index finger on off button with force of 1.4N
  - Put feet on floor
  - ...


Abstraction & levels of complexity
=== 

- Do:
  - Turn off alarm
  - Put on slippers
  - Brush teeth
  - Make coffee
  - ...


Hide complexity
===

- Break down code into several, abstract steps
  - "Import raw data", "Pre-process", "Create graphs", "Perform statistical tests"
- Hide complexity with user-defined functions
  - `head_pl <- sum((diff(head$x)^2 + diff(head$y)^2 + diff(head$z)^2)^0.5)`
  - `head_pl <- path_length(head)`


Goals of programming
===

- Do not use R (or MATLAB, Python, LabVIEW...) exclusively like a "tool"
- Think of yourself as creator of tools using these languages
- You are both a user and creator of these tools
- Imagine yourself as a naive user of these tools


Good code summary
===

- Code is for humans, not for computers
  - "Any fool can write code that a computer can understand. Good programmers write code that humans can understand." (M. Fowler)
- Comments do not always make better code
- Small amount of extra time early has big payoffs later


Interactive example
===

- Process & visualise some example data
- RMarkdown
- Tidyverse
  - Suite of packages that share the same underlying philosophy
  - Removes some complexities
  - Human-friendly verbs (e.g. `separate`, `filter`, `gather`)
- "Whole game" https://www.youtube.com/watch?v=go5Au01Jrvs

  
  
