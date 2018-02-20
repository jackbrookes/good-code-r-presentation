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
|P_001 |FALSE  | 1987| 13.879390|
|P_002 |FALSE  | 1974| 35.076083|
|P_003 |TRUE   | 1969|  6.965215|
|P_004 |TRUE   | 1965| 14.926262|
|P_005 |FALSE  | 1972| 19.914346|
|P_006 |TRUE   | 1987| 37.925976|
|P_007 |TRUE   | 1989| 20.377774|
|P_008 |FALSE  | 1984| 39.155802|
|P_009 |FALSE  | 1983| 23.355315|
|P_010 |FALSE  | 1965| 32.780991|
|P_011 |FALSE  | 1962| 33.141728|
|P_012 |TRUE   | 1976| 15.938454|
|P_013 |TRUE   | 1989| 43.186279|

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
|P_001          |          1987|   13.879390|Male   |    -0.6627530|  31|
|P_002          |          1974|   35.076083|Male   |     0.7239773|  44|
|P_003          |          1969|    6.965215|Female |    -1.1150924|  49|
|P_004          |          1965|   14.926262|Female |    -0.5942646|  53|
|P_005          |          1972|   19.914346|Male   |    -0.2679341|  46|
|P_006          |          1987|   37.925976|Female |     0.9104230|  31|
|P_007          |          1989|   20.377774|Female |    -0.2376157|  29|
|P_008          |          1984|   39.155802|Male   |     0.9908807|  34|
|P_009          |          1983|   23.355315|Male   |    -0.0428189|  35|
|P_010          |          1965|   32.780991|Male   |     0.5738278|  53|
|P_011          |          1962|   33.141728|Male   |     0.5974279|  56|
|P_012          |          1976|   15.938454|Female |    -0.5280449|  42|
|P_013          |          1989|   43.186279|Female |     1.2545627|  29|
|P_014          |          1969|   43.130715|Male   |     1.2509276|  49|


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
type: sub-section

- Process & visualise some example data
- RMarkdown
- Tidyverse
  - Suite of packages that share the same underlying philosophy
  - Removes some complexities
  - Human-friendly verbs (e.g. `separate`, `filter`, `gather`)
- "Whole game" https://www.youtube.com/watch?v=go5Au01Jrvs

  
  
