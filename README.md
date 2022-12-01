# bimm 143
Fall 2022 Bioinformatics Lab @ UC San Diego

Here is a portfolio of my work for [BIMM143](https://bioboot.github.io/bimm143_F22/)

- Class 10 [Halloween Candy Mini-Project](https://github.com/tylercyang/bimm143/blob/main/class10/class10.md) | <details>
  <summary>Code</summary>

## Importing Candy Data

```{r}
candy_file <- "candy-data.csv"

candy = read.csv(candy_file, row.names=1)
head(candy)
```

> Q1. How many different candy types are in this dataset?

To find this we can use `dim()`

```{r}
dim(candy)
```

There are 85 candy types.

> Q2. How many fruity candy types are in the dataset?

To find the fruity candy types, we can use `sum(candy$fruity)` to add the number of fruity candy because true is equal to 1

```{r}
sum(candy$fruity)
```


## What is your favorite candy?

We can find the winpercent value for Twix by using its name to access the corresponding row of the dataset. This is because the dataset has each candy name as rownames (recall that we set this when we imported the original CSV file). For example the code for Twix is:

```{r}
candy["Twix", ]$winpercent
```

> Q3. What is your favorite candy in the dataset and what is it’s winpercent value?

My favorite candy are Sour Patch Kids. Its win percent is...

```{r}
candy["Sour Patch Kids", ]$winpercent
```

> Q4. What is the winpercent value for “Kit Kat”?

```{r}
candy["Kit Kat", ]$winpercent
```

> Q5. What is the winpercent value for “Tootsie Roll Snack Bars”?

```{r}
candy["Tootsie Roll Snack Bars", ]$winpercent
```

Installing Skimr

```{r}
library("skimr")
x <- skim(candy)
x
```

> Q6. Is there any variable/column that looks to be on a different scale to the majority of the other columns in the dataset?

The winpercent variable is on a different scale to the majority of the other columns because it has a scale between 0 and 100 while the other variables are between 0 and 1.

> Q7. What do you think a zero and one represent for the candy$chocolate column?

For the candy$chocolate column, a zero represents that there is no chocolate in that particular candy and a one represents that there is chocolate in that particular candy.

Plot a histogram of candy.

```{r}
library(ggplot2)

ggplot(candy) + 
  aes(winpercent) +
  geom_histogram(bins = 15)
```

> Q9. Is the distribution of winpercent values symmetrical?

No, the distribution of winpercent values are not symmetrical as they look to be skewed to the left.

> Q10. Is the center of the distribution above or below 50%?

The center of the distribution seems to be below 50%.

> Q11. On average is chocolate candy higher or lower ranked than fruit candy?

We can find the means of the winpercent of the different candies.

```{r}
winPercentChoco <- mean(candy$winpercent[as.logical(candy$chocolate)])

winPercentFruit <- mean(candy$winpercent[as.logical(candy$fruity)])

winPercentChoco > winPercentFruit
```

> Q12. Is this difference statistically significant?

```{r}
t.test(candy$winpercent[as.logical(candy$chocolate)], candy$winpercent[as.logical(candy$fruity)])
```

We can assume that the null hypothesis is false and that people prefer chocolate over fruity candies.

## Overall candy Rankings

> Q13. What are the five least liked candy types in this set?

```{r}
head(candy[order(candy$winpercent),], n=5)
```

> Q14. What are the top 5 all time favorite candy types out of this set?

```{r}
tail(candy[order(candy$winpercent),], n=5)
```

> Q15. Make a first barplot of candy ranking based on winpercent values.

```{r}

ggplot(candy) + 
  aes(winpercent, rownames(candy)) +
  geom_col()
```

> Q16. This is quite ugly, use the reorder() function to get the bars sorted by winpercent?

```{r}
my_cols=rep("black", nrow(candy))
my_cols[as.logical(candy$chocolate)] = "chocolate"
my_cols[as.logical(candy$bar)] = "blue"
my_cols[as.logical(candy$fruity)] = "pink"


ggplot(candy) + 
  aes(winpercent, reorder(rownames(candy),winpercent)) +
  geom_col(fill = my_cols)

ggsave("candy_winpercent.png", height = 12, width = 12)
```

![colored of candy winpercents](candy_winpercent.png)

> Q17. What is the worst ranked chocolate candy?

Nik L Lips

> Q18. What is the best ranked fruity candy?

Reese's Peanut Butter Cups

## Taking a look at pricepercent

Comparing the pricepercent value which ranks the candy based on how expensive it is with the winpercent to try and find the best candy for the most value.

```{r}
library(ggrepel)

# How about a plot of price vs win
ggplot(candy) +
  aes(winpercent, pricepercent, label=rownames(candy)) +
  geom_point(col=my_cols) + 
  geom_text_repel(col=my_cols, size=3.3, max.overlaps = 40)

ggsave("price_vs_win.png", height = 15, width = 15)
```

![price vs win plot](price_vs_win.png)

> Q19. Which candy type is the highest ranked in terms of winpercent for the least money - i.e. offers the most bang for your buck?

It looks like Reese's Minatures would have the most bang for your buck.

> Q20. What are the top 5 most expensive candy types in the dataset and of these which is the least popular?

```{r}
ord <- order(candy$pricepercent, decreasing = TRUE)
head( candy[ord,c(11,12)], n=5 )
```


## Exploring the Correlation Structure

```{r}
library(corrplot)

cij <- cor(candy)
corrplot(cij)
```

> Q22. Examining this plot what two variables are anti-correlated (i.e. have minus values)?

Two variables are anti-correlated are chocolate and fruity as if a candy is chocolate it is not fruity.

> Q23. Similarly, what two variables are most positively correlated?

It looks like winpercent and chocolate are most positively correlated as people are more likely to choose chocolate candies.

## Principal Component Analysis

Let's do PCA on this dataset. We will use `prcomp()` function and set `scale = T` because the winpercent and pricepercent values are on a different scale.

```{r}
pca <- prcomp(candy, scale = T)
summary(pca)
```

```{r}
plot(pca$x[,])
```

Change plotting character and add some color.

```{r}
plot(pca$x[,1:2], col=my_cols, pch=16)
```

Use ggplot to make a nicer plot. 

```{r}
# Make a new data-frame with our PCA results and candy data
my_data <- cbind(candy, pca$x[,1:3])

p <- ggplot(my_data) + 
        aes(x=PC1, y=PC2, 
            size=winpercent/100,  
            text=rownames(my_data),
            label=rownames(my_data)) +
        geom_point(col=my_cols)

p
```

Use the ggrepel package with labels on the points aswell.

```{r}
p + geom_text_repel(size=3.3, col=my_cols, max.overlaps = 50)  + 
  theme(legend.position = "none") +
  labs(title="Halloween Candy PCA Space",
       subtitle="Colored by type: chocolate bar (dark brown), chocolate other (light brown), fruity (red), other (black)",
       caption="Data from 538")

ggsave("Halloween_Candy_PCA_Space.png", height = 20, width = 20)
```

![Halloween Candy PCA Space](Halloween_Candy_PCA_Space.png)

> Q24. What original variables are picked up strongly by PC1 in the positive direction? Do these make sense to you?

The variables fruity, hard, and pluribus are picked up strongly by PC1 in the positive direction because usually if a candy is fruity, it is also hard and comes in a lot, which is the opposite of chocolate candy.
  
</details>

- Class 17 [Covid Vaccination Mini-Project](https://github.com/tylercyang/bimm143/blob/main/class17/class17.md) | [code]()

- Class 19 [Pertussis Resurgence Mini-Project](https://github.com/tylercyang/bimm143/blob/main/class19/class19.md) | [code]()
