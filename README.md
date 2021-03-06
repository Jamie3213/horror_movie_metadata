Horror Movie Metadata
================
Jamie Hargreaves

``` r
library(tidyverse)
library(magrittr)

# get data
horror_movies <- readr::read_csv("https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2019/2019-10-22/horror_movies.csv")

# get the release year
horror_movies %<>%
  mutate(release_year = str_sub(release_date, start = -2))

# add '20'
horror_movies %<>%
  mutate(release_year = str_c("20", release_year))

# only consider observations with a month in the release date
horror_movies %<>%
  filter(nchar(release_date) != 4)

# get the release month
horror_movies %<>%
  mutate(release_month = str_match(release_date, "-...-")) %>%
  mutate(release_month = str_remove_all(release_month, "-"))

# make it a 'number'
horror_movies %<>%
  mutate(release_month_num = case_when(
    release_month == "Jan" ~ "01",
    release_month == "Feb" ~ "02",
    release_month == "Mar" ~ "03",
    release_month == "Apr" ~ "04",
    release_month == "May" ~ "05",
    release_month == "Jun" ~ "06",
    release_month == "Jul" ~ "07",
    release_month == "Aug" ~ "08",
    release_month == "Sep" ~ "09",
    release_month == "Oct" ~ "10",
    release_month == "Nov" ~ "11",
    release_month == "Dec" ~ "12"
  )
)

# get the release day
horror_movies %<>%
  mutate(release_day = str_sub(release_date, start = 1, end = 2)) %>%
  mutate(release_day = str_remove(release_day, "-")) %>%
  mutate(
    release_day = ifelse(
      nchar(release_day) == 1, 
      str_c("0", release_day),
      release_day
    )
  )

# recreate the date as a date time object
horror_movies %<>%
  mutate(release_date = as.Date(str_c(release_year, release_month_num, release_day,
        sep = "-")))

# get day of week of release
horror_movies %<>%
  mutate(day_of_week = weekdays(release_date))

# make run time numeric
horror_movies %<>%
  mutate(movie_run_time = as.numeric(str_remove(movie_run_time, " min")))

# create a variable that captures how many genres the movie spans
horror_movies %<>%
  mutate(genres_spanned = str_count(genres, "\\|") + 1)

# create a variable that captures how many languages the movie spans
horror_movies %<>%
  mutate(languages_spanned = str_count(language, "\\|") + 1)

# make a new tibble with just the features we want
horror_movies_final <- horror_movies %>%
  select(
    release_year, release_month, release_day = day_of_week, release_country,
    run_time = movie_run_time, languages_spanned, genres_spanned, rating = review_rating
  )

# filter NAs
horror_movies_final %<>%
  drop_na()

# train a multiple linear regression
mlr <- lm(rating ~., data = horror_movies_final)

# plot the top 10 most influential predictors
library(caret)
library(hrbrthemes)

tibble(
  variable = rownames(varImp(mlr)),
  score = varImp(mlr)$Overall
) %>%
  top_n(10) %>%
  ggplot(aes(x = reorder(variable, score))) + 
  geom_segment(aes(xend = reorder(variable, score), y = 0, yend = score),
               colour = "grey") +
  geom_point(aes(y = score), size = 4, colour = "orange2") + 
  coord_flip() + 
  labs(
    x = NULL,
    y = "Variable Importance Score",
    title = "What Factors Influence a Movie's Rating?",
    subtitle = "Top 10 Most Influential Predictors Using Multiple Linear Regression",
    caption = "Predictors considered: release year, month, day, country,
    movie run time, number of languages and number of genres spanned."
  ) + 
  theme_ipsum()
```

<img src="README_files/figure-gfm/unnamed-chunk-1-1.png" width="672" style="display: block; margin: auto;" />
