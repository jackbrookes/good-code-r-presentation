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

Trade-offs

Example: data
===
incremental: false

CSV file: Columns are participant ID, gender, year of birth, and performance in a task


```
       p female    y         x
1  P_001   TRUE 1995 15.165281
2  P_002   TRUE 1986 40.151915
3  P_003  FALSE 1973 23.657257
4  P_004  FALSE 1984 12.494387
5  P_005  FALSE 1978 21.257417
6  P_006   TRUE 1988 -2.333408
7  P_007  FALSE 1961 35.544256
8  P_008  FALSE 1983 31.367408
9  P_009  FALSE 1968 18.485300
10 P_010  FALSE 1975 30.785857
11 P_011  FALSE 1973  5.632823
12 P_012  FALSE 1996 15.562307
13 P_013   TRUE 1984 37.171223
14 P_014   TRUE 1981 31.615556
```

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


```
   participant_id year_of_birth performance gender performance_z age
1           P_001          1995   15.165281 Female   -0.68743390  23
2           P_002          1986   40.151915 Female    1.15942576  32
3           P_003          1973   23.657257   Male   -0.05975882  45
4           P_004          1984   12.494387   Male   -0.88485006  34
5           P_005          1978   21.257417   Male   -0.23714031  40
6           P_006          1988   -2.333408 Female   -1.98083027  30
7           P_007          1961   35.544256   Male    0.81885572  57
8           P_008          1983   31.367408   Male    0.51012857  35
9           P_009          1968   18.485300   Male   -0.44203833  50
10          P_010          1975   30.785857   Male    0.46714387  43
11          P_011          1973    5.632823   Male   -1.39201505  45
12          P_012          1996   15.562307   Male   -0.65808816  22
13          P_013          1984   37.171223 Female    0.93911120  34
14          P_014          1981   31.615556 Female    0.52847014  37
```



Abstraction & levels of complexity
=== 

- Human-like description
- Morning routine:
- Don't:
  - Open eyes
  - Move arm 50cm towards alarm clock
  - Depress index finger on off button
  - Put feet on floor
  - ...


Abstraction & levels of complexity
=== 

- Do:
  - Turn off alarm
  - Put on slippers
  - Brush teeth
  - Make coffee
  _ ...


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

  
  
