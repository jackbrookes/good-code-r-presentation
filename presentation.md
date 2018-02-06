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

- Does it's job
- Works in different contexts
- Easily understood
- Easily modified by colleague, or yourself in the future
- Efficient


Example data
===
incremental: false

CSV file: Columns are participant ID, gender, age, and performance in a task


```
       p female    y         x
1  P_001  FALSE 1968 43.694022
2  P_002  FALSE 1977 18.068902
3  P_003   TRUE 1966 41.103788
4  P_004  FALSE 1995 20.111353
5  P_005   TRUE 1996 46.189773
6  P_006   TRUE 1971 49.175048
7  P_007   TRUE 1991 13.659537
8  P_008  FALSE 1963  8.391421
9  P_009  FALSE 1990 15.038065
10 P_010   TRUE 1962 20.704209
11 P_011   TRUE 1967 -5.849036
12 P_012   TRUE 1973 17.571438
13 P_013  FALSE 1972 21.349949
14 P_014  FALSE 1993 32.912939
```

Example code
===
incremental: false

```r
data <- read_csv("participant_performance.csv")
data =data %>%
  filter((x-mean(x)/sd(x) < 3) & (x-mean(x)/sd(x) > -3))
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

- Comments can't fix the code
- Comments get left outdated 
- Takes extra time (both reading and writing)
- Think about:
  - What aspect of my code is this comment improving?
  - Can the code be changed to remove the need for a comment?


***

![Push?](images/push.jpg)

At the top of our script..
===
incremental: false

```r
# CONSTANTS
z_cutoff <- 3
current_year <- Sys.Date() %>% 
  format("%Y") %>% 
  as.integer()

# FUNCTIONS
zscore <- function(x){
  (x-mean(x))/sd(x)
}
```

Naming
===
incremental: false
And the **D**ont **R**epeat **Y**ourself principle

```r
# READ RAW DATA
scores_data_raw <- read_csv("participant_performance.csv") %>% 
  rename(participantID = p, is_female = female, year_of_birth =y,performance = x) %>%
  mutate(gender = ifelse(is_female, "Female", "Male")) %>% 
  select(-is_female) #remove redundant column

# PREPROCESS
scores_data =scores_data_raw %>%
  filter((zscore(performance) < z_cutoff) &
         (zscore(performance) > -z_cutoff))

scores_data$age <- current_year-scores_data$year_of_birth
scores_Males <- filter(data, gender == "Male")
```


Consistency & Formatting
===
incremental: false

Keep a consistent style e.g. [Tidyverse style guide](http://style.tidyverse.org/), [Google's R style guide](https://google.github.io/styleguide/Rguide.xml)

```r
# READ RAW DATA
scores_data_raw <- read_csv("participant_performance.csv") %>% 
  rename(participant_id = p,
         is_female = female,
         year_of_birth = y,
         performance = x) %>%
  mutate(gender = ifelse(is_female, "Female", "Male")) %>% 
  select(-is_female) #remove redundant column

# PREPROCESS
scores_data <- scores_data_raw %>%
  mutate(performance_z = zscore(performance)) %>%
  filter(abs(performance_z) < z_cutoff) %>% 
  mutate(age = current_year - year_of_birth)

scores_males <- scores_data %>%
  filter(gender == "Male")
```


Big payoff now our script and data is "tidy"
===
incremental: false


```
   participant_id year_of_birth performance gender performance_z age
1           P_001          1968   43.694022   Male     1.2940755  50
2           P_002          1977   18.068902   Male    -0.4718779  41
3           P_003          1966   41.103788 Female     1.1155697  52
4           P_004          1995   20.111353   Male    -0.3311226  23
5           P_005          1996   46.189773 Female     1.4660699  22
6           P_006          1971   49.175048 Female     1.6718000  47
7           P_007          1991   13.659537 Female    -0.7757490  27
8           P_008          1963    8.391421   Male    -1.1388008  55
9           P_009          1990   15.038065   Male    -0.6807479  28
10          P_010          1962   20.704209 Female    -0.2902659  56
11          P_011          1967   -5.849036 Female    -2.1201809  51
12          P_012          1973   17.571438 Female    -0.5061606  45
13          P_013          1972   21.349949   Male    -0.2457648  46
14          P_014          1993   32.912939   Male     0.5510979  25
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


Hide complexity
===

- Break down code into several, abstract steps
  - "Import raw data", "Pre-process", "Create graphs", "Perform statistical tests"
- Hide complexity with user-defined functions
  - `head_pl <- sum((diff(head$x)^2 + diff(head$y)^2 + diff(head$z)^2)^0.5)`
  - `head_pl <- path_length(head)`


Creating tools
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

  
  
