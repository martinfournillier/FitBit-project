# R codes
install.packages("tidyverse")
library('tidyverse')
library('readxl')
fitbit = read.csv("fitbit.csv")
head(fitbit)
view(fitbit)

fitbit_long <- fitbit %>%   ## The %>% pipe means “take the dataset on the left and pass it to the next function”.
  pivot_longer(
    cols = -c(Id,ActivityDate),  ## pivot everything except Id and activity date 
    names_to = "Variable",
    values_to = "Value"
  )

#### filter for activity minutes:
activity_data <- fitbit_long %>%
  filter(Variable %in% c("sed_mins", "light_active_mins", "fairly_active_mins", "very_active_mins"))

#### group and sum overall minutes per activity
activity_summary <- activity_data %>%
  group_by(Variable) %>%
  summarise(TotalMinutes = sum(Value, na.rm = TRUE))

ggplot(data = activity_summary) +
  geom_bar(stat = "identity", mapping = aes(x = Variable, y = TotalMinutes, fill = Variable)) +  # (stat = "identity") tells ggplot to “Use the y-values in my dataset exactly as they are, don’t count rows.” because it counts rows by default
  labs(title = "Total Minutes by Activity Type", x = "Activity Type", y = "Total Minutes") +
  theme_minimal() +
  theme(legend.position = "none")  # hide legend if x-axis labels already show variable
  
## Creating a matrix of each users lifestyle and exercise intensity
library(dplyr)
library(lubridate)

#### Lifestyle column
lifestyle <- fitbit %>% 
  group_by(Id) %>% 
  summarise(avg_steps = mean(TotalSteps, na.rm = TRUE)) %>% 
  mutate(
    lifestyle = case_when(
      avg_steps < 5000 ~ "Sedentary",
      avg_steps >= 5000 & avg_steps < 7500 ~ "Low Active",
      avg_steps >= 7500 & avg_steps < 10000 ~ "Somewhat Active",
      avg_steps >= 10000 & avg_steps < 12500 ~ "Active",
      avg_steps >= 12500 ~ "Highly Active"
    )
  )

#### Activity column
Activity <- fitbit %>% 
  group_by(Id) %>% 
  summarise(
    weekly_moderate = sum(fairly_active_mins) / 7,  ### avg per week
    weekly_high = sum(very_active_mins) / 7,
    .groups = "drop"
  ) %>% 
  mutate(
    Activity = ifelse(weekly_moderate >= 150 | weekly_high >= 75,
                      "Active", "Inactive")
  )

#### Join into a matrix
matrix_class <- lifestyle %>% 
  inner_join(Activity, by = "Id") %>% 
  select(Id, lifestyle, Activity)

view(matrix_class)

#### Viz

#### Summarize counts
matrix_summary <- matrix_class %>%
  count(lifestyle, Activity)

#### Heatmap

###### Create numeric activity score
matrix_summary <- matrix_summary %>%
  mutate(
    lifestyle_score = case_when(
      lifestyle == "Sedentary" ~ 1,
      lifestyle == "Low Active" ~ 2,
      lifestyle == "Somewhat Active" ~ 3,
      lifestyle == "Active" ~ 4,
      lifestyle == "Highly Active" ~ 5
    ),
    activity_score = ifelse(Activity == "Active", 1, 0),
    combined_score = lifestyle_score + activity_score
  )

###### Heatmap shaded by combined score

ggplot(matrix_summary, aes(x = Activity, y = lifestyle, fill = combined_score)) +
  geom_tile(color = "white") +
  geom_text(aes(label = n), color = "black", size = 5) +
  scale_fill_gradient(low = "red3", high = "green") +
  labs(title = "Users' Lifestyle",
       x = "Activity Level", y = "Lifestyle",
       fill = "Activity Score") +
  theme_classic()
  
  

