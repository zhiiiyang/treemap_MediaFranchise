---
title: "Data visualization of School Diversity"
author: "Zhi Yang"
date: "9/23/2019"
output: 
  html_document:
    keep_md: true
---




```r
df <- readr::read_csv("https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2019/2019-09-24/school_diversity.csv")
```

```
## Parsed with column specification:
## cols(
##   LEAID = col_character(),
##   LEA_NAME = col_character(),
##   ST = col_character(),
##   d_Locale_Txt = col_character(),
##   SCHOOL_YEAR = col_character(),
##   AIAN = col_double(),
##   Asian = col_double(),
##   Black = col_double(),
##   Hispanic = col_double(),
##   White = col_double(),
##   Multi = col_double(),
##   Total = col_double(),
##   diverse = col_character(),
##   variance = col_double(),
##   int_group = col_character()
## )
```



```r
df$Multi[is.na(df$Multi)] <- 0

round_preserve_sum <- function(vec) {
  temp <- floor(vec)
  digits <- vec - floor(vec)
  while(sum(temp) < 100){
    temp[which.max(digits)] <- temp[which.max(digits)] +1
    digits[which.max(digits)] <- 0
  }
  return(temp)
}
```



```r
for (group in c("Diverse", "Undiverse", "Extremely undiverse")) {
  for (year in c("1994-1995", "2016-2017")) {
    df2 <- df %>% mutate(Others = AIAN+Asian+Multi) %>%
      group_by(ST, diverse, SCHOOL_YEAR) %>% 
      summarise(White = mean(White),
                Black = mean(Black), 
                Hispanic = mean(Hispanic), 
                AIAN = mean(AIAN),
                Asian = mean(Asian),
                Multi = mean(Multi)) %>% 
      gather("race", "count", -ST, -diverse, -SCHOOL_YEAR) %>% 
      filter(diverse==group & SCHOOL_YEAR==year) %>% 
      ungroup() %>%
      select(-diverse, -SCHOOL_YEAR) %>%
      arrange(ST)
    
    df2$count <- unlist(lapply(1:length(unique(df2$ST)), 
                               function(i) round_preserve_sum(df2$count[(i-1)*6 + 1:6])))
    
    df2[rep(row.names(df2), df2$count),] %>% 
      do(cbind.data.frame(ST = .$ST, 
                          race = .$race,
                          head(expand.grid(x = 1:10, y = 1:10), nrow(.)), row.names = NULL)) %>%
      mutate(race = fct_relevel(race, c("White", "Black", "Hispanic", 
                                        "AIAN", 'Asian', 'Multi'))) %>%
      ggplot()+
        geom_tile(aes(x = x, y = y, fill = race), 
                  color = '#F3F7F7', size = 0.5, alpha = 0.8) +
        scale_fill_manual(
        name = NULL,
        values = c("#e31b1b", "#347fb8", "#4bb04a",  
                   "#fd9069", '#f781bd', '#b2b5bc')
        ) +
      ggtitle(paste0("Population distribution by race/ethnicity\namong ", 
                  tolower(group), " school districts\nduring ", year)) +
      coord_equal() +
      facet_geo(~ST) +
      theme_void()+
      theme(
        legend.position = c(0.9, 0.2),
        legend.justification = 'left',
        legend.text = element_text(size = 12, margin = margin(t = 5)),
        legend.direction = 'vertical',
        legend.key.width = unit(5, 'pt'),
        text = element_text(color = '#5D646F'),
        strip.text = element_text(size = 12, hjust = 0,
                                  margin = margin(l = 5)),
        plot.title = element_text(size = 26, face = 'bold', margin = margin(b = 10)),
        plot.subtitle = element_text(size = 18, margin = margin(b = 20)),
        plot.caption = element_text(size = 12, margin = margin(t = 20)),
        plot.background = element_rect(fill = '#F3F7F7'),
        plot.margin = unit(c(1.5, 1.5, 1.5, 1.5), 'cm')
      )
    ggsave(paste0(here::here(), "/2019-09-24_school-diversity/",group, year,".png"), 
           width = 8, height = 8, dpi = 300, type = "cairo")

    }
}
```

```
## Warning: Unknown levels in `f`: Multi

## Warning: Unknown levels in `f`: Multi

## Warning: Unknown levels in `f`: Multi
```

