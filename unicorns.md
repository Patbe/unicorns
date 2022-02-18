unicorns
================

`{r setup, include=FALSE} knitr::opts_chunk$set(echo = TRUE)`

<style type="text/css">
  body{
  font-size: 14pt;
}
</style>

Source: Unicorn-startups-dataset, cleaned, Author: Niek van der Zwaag,
based on a dataset shared by @ramjasmaurya,
<https://www.kaggle.com/niekvanderzwaag/unicorn-startups-cleaned>

This is a short analysis of the unicorns dataset in R.The dataset
contains data on privately held startups that are valued at minimum of 1
billion USD. The dataset is from 2021 and not completely up to date, as
at least one company of that list has since gone public.

Questions:

1.  Which countries have the most unicorns?
2.  What are the unicorns with the highest market values?
3.  In what industries do these unicorns operate?

Since the data is already cleaned by the creator of the dataset I go
right into the analysis.

## Load Libraries

``` {r}
library(tidyverse)
library(dplyr)
library(janitor)
library(ggplot2)
library(forcats)
```

## Read the data

I drop na values from the dataset.

\`\`\`{r echo=FALSE, message=FALSE, warning=FALSE, paged.print=FALSE}

uni \<- read.csv(“Unicorn_Clean.csv”)

uni \<- uni %>% drop_na()

str(uni)



    ## What countries have the most unicorns?

    Look at the list of countries in the dataset:


    ```{r countries_group, echo=FALSE}
    distinct(uni,Country)

\`\`\`{r include=FALSE}

# get the number of countries

countries \<- uni %>% distinct(Country)

countries

num_countries \<- NROW(countries$Country) num_countries


    </br >
    There are in total `r num_countries` Countries in the dataset.


    ```{r companies_by_country, include=FALSE}

    # startup companies by country

    n_uni_country <- aggregate(uni$Company, by =list(uni$Country), FUN=length)

\`\`\`{r country_by_country_sort, include=FALSE}

# sort with arrange

n_uni_country_2 \<-n_uni_country %>% arrange(-x) %>% # First sort by
val. This sort the dataframe but NOT the factor levels
mutate(country=factor(Group.1, levels=Group.1))

n_uni_country_2



    </br >
    </br >


    Next I plot the number of unicorns per country:


    ```{r echo=FALSE}
    n <- ggplot(data=n_uni_country_2)+geom_count(aes(x=country,y=x,color = Group.1),show.legend = FALSE) +
       theme(axis.text.x = element_text(angle = 90, vjust = 0.5, hjust=1))
    #+
     # theme(legend.position="bottom")
     

\`\`\`{r echo=FALSE}

## override count with the command stat=“identity”

n2 \<- ggplot(data=n_uni_country_2)+geom_bar(stat = “identity”,
aes(x=country,y=x,fill = Group.1),show.legend = FALSE) +
theme(axis.text.x = element_text(angle = 90, vjust = 0.5, hjust=1))+
ylab(“number of unicorns”) #+ # theme(legend.position=“bottom”)

n2


    </br >

    As can be seen in the chart, the USA has by far the most unicorns, followed by China and India. 
    Of all 45 countries, only 11 have more than 10 unicorn, as can be seen in the following table:

    ```{r unicorns_country_top15, echo=FALSE}
    # slice the top 15

    n_uni_country_3 <- n_uni_country_2 %>% 
      select(Group.1,x) %>% 
      slice(1:20)
    n_uni_country_3

</br >

Interestingly, out of these 11 countries 4 are in Asia, and of the top
15 countries 7 are asian countries, perhaps a reflection of the
increasing economic importance of the continent.

</br >

## Market Values

\`\`\`{r include=FALSE}

# prepare data for charts

new_df \<-
aggregate(uni*V**a**l**u**a**t**i**o**n*, *l**i**s**t*(*u**n**i*Industry),
FUN=sum) new_df \<- new_df %>% arrange(-x)

slices \<- new_df$x

total_value \<-sum(slices) number_companies \<-
sum(complete.cases(uni$Company)) avg_value_company \<-
floor(total_value/number_companies)


    The total market value of all unicorns is `r total_value` billion USD. Since there are overall `r number_companies` companies with market values in the dataset, the average value of a unicorn is around `r avg_value_company` billion USD.
    </br >

    Next I want to see a visualization of the largest unicorns, a horizontal bar chart would be suitable for that.

    I set the cutoff at 20 to keep the chart neat.

    </br >


    ```{r barchart_companies, echo=FALSE}
    top_20 <- uni %>%
      arrange(desc(Valuation)) %>% 
      slice(1:20)
      
    barChart_company <-ggplot(top_20, aes(x=reorder(Company,Valuation),Valuation,labels=floor(Valuation))) +   
      

      geom_bar(stat = "identity",fill="lightblue") + coord_flip() + 
        geom_text(aes(label = floor(Valuation)),color="black")+ggtitle("Market Value in Billion USD by Company")+ 
            xlab("Company")+ylab("Market Value")

    barChart_company

Next, I want to see what industries hold the highest market values. For
that, I create a pie chart to visualize the percentages.

\`\`\`{r data_piechart_industries, include=FALSE}

percent \<- round(slices/sum(slices)\*100)

ind \<- new_df$Group.1  
lbls\<- paste(ind,percent) lbls \<- paste(lbls,“%”,sep=““)


    ```{r piechart_industries, echo=FALSE, results='hide'}
    pieChart <- pie(slices, labels = lbls, main="Industries",radius=1.05, cex = 0.6)
    pieChart

It can be seen that fintech accounts for the largest share in terms of
market value. Next I look at the absolute values with a horizontal bar
chart:

\`\`\`{r barchart_industries, echo=FALSE} # add the fill at geom_bar,
not the AES, otherwise it would be assigned to variables

barChart \<-ggplot(new_df, aes(x=reorder(Group.1,x),x,labels=floor(x)))
+

geom_bar(stat = “identity”,fill=“lightblue”) + coord_flip() +
geom_text(aes(label = floor(x)),color=“black”)+ggtitle(“Market Value in
Billion USD by Industry”)+ ylab(“Market Value”) + xlab(“Industry”)

barChart \`\`\`

Overall more than 50 percent of all market value in the dataset is
concentrated in technology related industries.
