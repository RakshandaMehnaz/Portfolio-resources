---
id: litvis

narrative-schemas:
  - ../../lectures/narrative-schemas/project.yml

elm:
  dependencies:
    gicentre/elm-vegalite: latest
    gicentre/tidy: latest
---

@import "../../lectures/css/datavis.less"

```elm {l=hidden}
import Tidy exposing (..)
import VegaLite exposing (..)
```

# Data Visualization Project Summary

{(whoami|}
Raksanda Mehnaz Huq
raksanda.huq@city.ac.uk
{|whoami)}

{(task|}

You should complete this datavis project summary document and submit it, along with any necessary supplementary files to **Moodle** by **Sunday 30th April, 5pm UK time**. Submissions will be awarded up to **80 marks** towards your coursework assessment total.

You are also encouraged to regularly commit and push changes to your datavis project throughout the term as you develop your project.

{|task)}

{(questions|}

- What are the trends and distribution of housing Tenure in England and Wales, and how have they changed since 2011 Census?
- How has the affordability of housing changed in the different regions and how did it affect Tenure distribution?
- Is there any correlation between changes in affordability and well-being measures over the last decade?

{|questions)}

{(visualization|}

**Visualization 1: Faceted Choropleth Maps of England and Wales showing Tenure distribution**
These maps show the four categories of Housing Tenure accross the Local Authority (LA) areas. The first four maps show the percentage distribution according to the 2021 Census and the second set of maps show the percentage change since 2011 Census. Hovering over an area will display the Tooltip including information on LA area, number of Residents living in that area and the percentages.

```elm {l=hidden v interactive}
tenureMaps2021 : Spec
tenureMaps2021 =
    let
        tenureData =
            dataFromUrl "https://raw.githubusercontent.com/RakshandaMehnaz/Data-Viz-datasets/main/TenurePercentChange.csv" []

        boundaryData =
            dataFromUrl "https://gicentre.github.io/data/census21/ewLAs.json"
                [ topojsonFeature "las" ]

        trans =
            transform
                << lookup "label" boundaryData "properties.label" (luAs "geo")

        enc =
            encoding
                << shape [ mName "geo", mGeo ]
                << color
                    [ mName "Percentage"
                    , mQuant
                    , mLegend
                        [ leTitle "Residents %"
                        , leOrient loNone
                        , leX 350
                        , leY 650
                        , leDirection moHorizontal
                        ]
                    ]
                << tooltips
                    [ [ tName "label", tTitle "Local Authority Area" ]
                    , [ tName "Value", tTitle "Number of Residents" ]
                    , [ tName "Percentage" ]
                    ]

        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
                << configuration (coFacet [ facoSpacing 3 ])
    in
    toVegaLite
        [ cfg []
        , tenureData
        , trans []
        , columns 2
        , facetFlow [ fName "Type", fHeader [ hdTitle "Tenure Distribution Percentage 2021" ] ]
        , specification (asSpec [ width 400, height 300, enc [], geoshape [ maStroke "white" ] ])
        ]
```

```elm {l=hidden v interactive}
tenureMaps2011 : Spec
tenureMaps2011 =
    let
        tenureData =
            dataFromUrl "https://raw.githubusercontent.com/RakshandaMehnaz/Data-Viz-datasets/main/TenurePercentChange.csv" []

        boundaryData =
            dataFromUrl "https://gicentre.github.io/data/census21/ewLAs.json"
                [ topojsonFeature "las" ]

        trans =
            transform
                << lookup "label" boundaryData "properties.label" (luAs "geo")

        enc =
            encoding
                << shape [ mName "geo", mGeo ]
                << color
                    [ mName "Percentage Change"
                    , mQuant
                    , mLegend
                        [ leTitle "Change %"
                        , leOrient loNone
                        , leX 350
                        , leY 650
                        , leDirection moHorizontal
                        ]
                    ]
                << tooltips
                    [ [ tName "label", tTitle "Local Authority Area" ]
                    , [ tName "Percentage Change", tTitle "Resident % Change" ]
                    ]

        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
                << configuration (coFacet [ facoSpacing 3 ])
    in
    toVegaLite
        [ cfg []
        , tenureData
        , trans []
        , columns 2
        , facetFlow [ fName "Type", fHeader [ hdTitle "Tenure Percentage Change Since 2011" ] ]
        , specification (asSpec [ width 400, height 300, enc [], geoshape [ maStroke "white" ] ])
        ]
```

**Visualization 2: Scatterplot showing Housing Affordability Ratio (AfR)**
The scatter plot data points show the AfR (i.e. House Price to Income). It includes a clickable interactive legend where selecting a region will highlight the data points for that region. The tooltip shows the Median Income value for each data point. The comparison between 2011 and 2021 can be observed.

```elm {l=hidden v interactive }
affordabilityRatioScatter : Spec
affordabilityRatioScatter =
    let
        data =
            dataFromUrl "https://raw.githubusercontent.com/RakshandaMehnaz/Data-Viz-datasets/main/AffordabilityDataEngWales.csv" []

        ps =
            params
                << param "mySelection"
                    [ paSelect sePoint [ seFields [ "Region name " ] ]
                    , paBindLegend ""
                    ]

        enc =
            encoding
                << position X [ pName "Year", pAxis [ axLabelAngle 0 ] ]
                << position XOffset
                    [ pName "Median House Earnings", pQuant, pScale [ scZero False ], pAxis [ axGrid False ] ]
                << position Y
                    [ pName "Median House Price", pQuant, pScale [ scZero False ], pAxis [ axGrid False ] ]
                << fillOpacity [ mCondition (prParam "mySelection") [ mNum 1 ] [ mNum 0.2 ] ]
                << color
                    [ mCondition (prParam "mySelection")
                        [ mName "Region name ", mScale [ scScheme "yelloworangered" [ 0.2, 0.9 ] ] ]
                        [ mStr "lightgrey" ]
                    ]
                << size
                    [ mName "Affordability Ratio", mScale [ scRange (raNums [ 0, 400 ]), scType scPow, scExponent (1 / 0.7) ] ]
                << tooltips
                    [ [ tName "Median House Earnings", tTitle "Median Income" ]
                    ]
    in
    toVegaLite [ width 600, height 480, data, ps [], enc [], circle [] ]
```

The maps below visualises the AfR values distributed accross LA area for 2011 and 2021. Hovering over an area will display Tooltip information about the Median House Price and Median Income for that area.

```elm {l=hidden v interactive}
affordabilityRatioMaps : Spec
affordabilityRatioMaps =
    let
        affordabilityRatioData =
            dataFromUrl "https://raw.githubusercontent.com/RakshandaMehnaz/Data-Viz-datasets/main/AffordabilityDataEngWalesGeo.csv" []

        boundaryData =
            dataFromUrl "https://gicentre.github.io/data/census21/ewLAs.json"
                [ topojsonFeature "las" ]

        trans =
            transform
                << lookup "Local authority name" boundaryData "properties.label" (luAs "geo")

        enc =
            encoding
                << shape [ mName "geo", mGeo ]
                << color
                    [ mName "Affordability Ratio"
                    , mQuant
                    , mLegend
                        [ leTitle "Affordability Ratio"
                        , leOrient loNone
                        , leX 400
                        , leY 500
                        , leDirection moHorizontal
                        ]
                    ]
                << tooltips
                    [ [ tName "Local authority name" ]
                    , [ tName "Median House Price" ]
                    , [ tName "Median House Earnings", tTitle "Median Income" ]
                    , [ tName "Affordability Ratio" ]
                    ]

        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
                << configuration (coFacet [ facoSpacing 1 ])
    in
    toVegaLite
        [ cfg []
        , affordabilityRatioData
        , trans []
        , columns 2
        , facetFlow [ fName "Year", fHeader [ hdTitle "" ] ]
        , specification (asSpec [ width 500, height 480, enc [], geoshape [ maStroke "white" ] ])
        ]
```

**Visualization 3: Grouped Histogram showing the Wellbeing measures**
The Well-being data is based on the ten regions across England and Wales. These are the average rates of the four measures of Wellbeing for the years 2021-2022 since the recent Census. The second grouped histogram shows the percentage change of each of the measures when compared with the year 2011-2012.

```elm {v}
wellbeingBar : Spec
wellbeingBar =
    let
        data =
            dataFromUrl "https://raw.githubusercontent.com/RakshandaMehnaz/Data-Viz-datasets/main/WellbeingData2011and2021v2.csv" []

        enc =
            encoding
                << position X [ pName "Region", pNominal, pAxis [ axTitle "" ] ]
                << position XOffset [ pName "Measure" ]
                << position Y [ pName "2021 Score", pQuant ]
                << color [ mName "Measure" ]
    in
    toVegaLite [ width 500, height 250, widthStep 15, data, enc [], bar [] ]
```

```elm {v}
wellbeingBar2 : Spec
wellbeingBar2 =
    let
        data =
            dataFromUrl "https://raw.githubusercontent.com/RakshandaMehnaz/Data-Viz-datasets/main/WellbeingData2011and2021v2.csv" []

        enc =
            encoding
                << position X [ pName "Region", pNominal, pAxis [ axTitle "" ] ]
                << position XOffset [ pName "Measure" ]
                << position Y [ pName "Percentage", pQuant ]
                << color [ mName "Measure" ]
    in
    toVegaLite [ width 500, height 250, widthStep 15, data, enc [], bar [] ]
```

{|visualization)}

{(insights|}

1.  Insight one

The housing tenure distribution is shown accross England and Wales by LA area. The visualisations mainly compare the changes between 2011 Census and 2021 Census, displaying the number and percentages of Residents across England and Wales for each of the Tenure type.

Considering the 2021 Census data only, Outright ownership of houses were highest in Wales, Southern England and along the coastal regions in the North East and North West. The percentage of residents in these areas are above 40%, as evident from the dark blue hue. High percentages can also be seen scattered in the Central England region and the least outright ownerships can be seen in Manchester and in London with values below 20%. Ownership by mortgage, loan and shared ownership are most common around the Central regions of England such as in Derby, in the Southern region of Hart and in the East such as King's Lynn. The regions with the greatest numbers of outright ownership have the lowest ocurrences of ownership by other means, such as in Wales and the coastal regions of England, the suburban and rural areas. Social renting and private renting is most common in Greater London (Barnet 32.8%), as well as in Manchester and Leicester with above 30% of residents. The highest percentages of social renting is in Soutwark borough and private renting in Westminster borough of London. These are the highest proportions in the whole of England and Wales which also suggests that outright ownership in these urban areas are the lowest due to the peak prices of property, greater population of immigrants and students.

Social housing has mostly remained same in England and Wales over the last decade, with some noticeable decrease in Central parts of England and London. Slight increases in Southwest England such as in Cotswold. On the other hand, Private renting has increased on most parts of England, the Central and the South. However, in some parts of Wales there have been a decrease in private renting, these are the same regions where outright ownership has increased.

2.  Insight two

The AfR is the ratio of House Price to Income for the years 2011 and 2021. The distribution is across the 10 regions of England and Wales. The Affordability Ratio (AfR) have different changing patterns between 2011 and 2021. Previously, House Prices proportionately increased with increase in annual Income. The shallow slope for 2011 suggests this change. The lower the AfR the more affordable housing is in that region. Whereas, in 2021 the AfR has changed rather exponentially. The scatter plot suggests slight increase in annual Income and more than proportionate increase in House Prices.

This is expected since the cost of living crisis that started and increased since 2019, creating a major impact in the housing market. Although some regions have not been impacted as much as others. For instance, the change in Yorkshire and Humber region is very less both in terms of income and house price. On the contrary, the anomaly in London region shows the largest AfR value, a significant difference from the rest of the data points in 2021 as well.

The choropleth map shown by LA area, reflects the AfR in 2011 and 2021. The darkest hue shows the least affordable housing regions. For instance, the least affordable area is Kensigton and Chelsea (AfR around 17 in 2011 and 25 in 2021). I was curious about how these AfR values relate to Tenure distribution. The visualisations suggest that percentage of Outright ownership in Kensington and Chelsea was only about 20% in 2021. Whereas percentage of social and private rentings ranged from around 27% to 40% in that area. So, it can be concluded that AfR ratio is correlated to changes in ownership and renting Tenure in regions where other factors such as dwellings and population density remained similar. An exception would be the South Cambridgshire region where the change in AfR has been very low, despite that outright ownerships did not increase. This could be due to low population in that area, so not particularly influenced by AfR. However, the effect of AfR on Tenure reflects some disparity on homeownership and a larger income gap resulting in social inequality.

3.  Insight three

The Personal wellbeing data is based on the Annual Population Survey which is rated on an 11-point scale and is measured using four indicators - life satisfaction, worthwile, happiness and anxiety. I have used the average (mean) ratings for the visualization.

The Personal Wellbeing data for 2021-2022 in England and Wales region suggests average ratings have improved across all four indicators. The most deterioration in these scores occured in 2021. Potentially due to the lockdown during pandemic since 2020 and then rising inflation. However, the proportion of people reporting very high levels of life satisfaction, feeling that the things done in life are worthwhile, happiness or very low levels of anxiety increased in the year ending March 2022 both in England and Wales.

Comparing to AfR data, it can be seen that the Anxiety measure in Southeast region has increased more than other regions and the areas in those region have the higher range AfR values. On the contrary, Anxiety level in London has decreased where the AfR value is one of the highest. Yorkshire and The Humber region has one of the highest measures that suggest good wellbeing and the AfR values in that region are relatively low. AfR can potentially be one of the indicators related to personal wellbeing. However, when comparing this dataset to housing affordability accross the same regions and pattern of ownership, some external factors also need to be considered to make a fair comparison. Such as the population size and structure, availability of other facilities, economic factors in the region that are likely to affect the wellbeing measures.

{|insights)}

{(designJustification|}

1.  Choice one

Following Tufte's Data-Ink Ratio principle, I focussed on maximising the use of visual elements for representing properties of the data. For instance, in the choropleth maps I used colour hues to represent percentages and percentage changes. Rather than providing multiple labels and cluttering the map, I used tooltips for the viewer to see additional information. I avoided Chart Junk by providing a simple legend to show the range of value with the change in colour hue and only including the boundary lines essential for separating LA areas. I also attempted to maintain Graphical Integrity with the use of colour hue scale for a more accurate representation of the changes in data, and it is consistent with the other sets of maps as well.

Successful interaction would have allowed the selection of year 2011 and 2021 using a dropdown on the same map and allow zooming of the map to perceive the local authority areas more clearly. I have included the codes that could not be successfully implemented in the 'experimentWithData' file.

2.  Choice two

I have used the default colur schemes in Vegalite and explored some of the available schemes in the library. It was important to maintain consistency for proper representation of the change in values. Also, to maintain a proper colour contrast and make the visualisation accessible for users who might have visual impairments I used the 'yellow-orange-red' scheme developed as part of the colour-brewer system. Slightly increased the contrast within the colour scale and reduced the opacity value. This helped in visualising more effectively the changes in distribution of AfR between adjacent regions across England and Wales.

To support Detection, Assembly and Estimation identified by Cleveland (1993) I made the following design choices:

Detection - The choropleth maps and the scatter plots for both 2011 and 2021 enables identification of trends and relationships between Tenure distribution. The color-coding for regions, size of the circles in the scatter plots and the interactive legend aid in distinguishing the AfR changes and the detection of specific regions.

Assembly - the faceted maps and the adjacent scatter plots aids in assembling the visual information. This enables the viewer to compare the changes in affordability ratios between 2011 and 2021. The interactive legend allows users to focus on specific regions, which supports assembling regional information.

Estimation - The presentation of percentage of residents rather than the large values in the choropleth maps allow for better estimation. Moreover, the change in percentages informs on changing trends, allowing the user to easily form estimations and faster comparison between subsets of values. This helps reduce the load on working memory. As mentioned by Franconeri et al. (2021) these are some of the properties of effective data visualisation.

3.  Choice three

Within the Grouped Histogram and percentage change distribution, Direct encoding is used to show percentage differences in Wellbeing measures.

As proposed by Satyanarayan et al (2017), in the grammar of interaction, I aimed to include an interaction - making a selection in the AfR scatter plot and creating a response to that selection in the choropleth map of Tenure distribution. It would have been more intuitive to compare the Tenure distribution with the AfR. To suggest for instance, how low affordability affected the rate of outright ownerships. This could not be implemented, so I have included a sketch to show my intended design.

![Sketch 1][https://github.com/cityteaching/inm402coursework2023-rakshandamehnaz/blob/80440fa78be2ce341d7d12780b49252d38feef2e/portfolio/project/interaction%20bet%20tenure%20afr%20sketch.jpg]

As a workaround, I created 2 more maps in Visualisation 2 which display the AfR information by LA area with the use of tooltips.

{|designJustification)}

{(validation|}

1.  Evaluation one

The geographic layout helped recognise patterns in Tenure distribution and using percentage of residents and percentage change data helped with comparing the patterns since the 2011 Census. However, I intended to show these on a single map with double encoding with shapes suggesting the Tenure category and colour hue showing the percentage distribution. It would have been easier to zoom in on the single map to identify the LA areas more closely. Although, this would have cluttered the map and this was a compromise to use the faceted maps to increase the Data-Ink Ratio.

The scatterplots successfully showed the AfR distribution and changes. Although, to understand how AfR has affected Tenure distribution and the resulting well-being measures in those regions would have been better visualised with the use of Grammar of Interaction by Satyanarayan et al (2017). However, the current visualisations helped make inference on the research questions.

2.  Evaluation two

To enable more intuitive comparisons I used percentage values to show proportions of residents. In the Wellbeing dataset I used Excel function to calculate the percentage change in values with respect to 2011. The map and scatterplot interactions helped in understanding the data better, although implementing map zooming would have been better.

Additionally, there are several other factors that play a role in Housing Affordability, Tenure distribution and on Personal Wellbeing as well. Due to time constraints and project scope I was not able to consider more factors such as: growing population of young professionals and immigrants, socio-economic factors like interest in loans, equitable housing access, House quality and energy efficiency.

3.  Evaluation three

As mentioned in Design Justification, interaction between the AfR scatterplot and the Tenure map could not be implemented and I have included the Sketch image in the local folder (Interaction bet Tenure AfR sketch.jpg). Additionally, interaction between the percentage change in Wellbeing grouped histogram and the AfR map would have led to better comparison. Particularly to see the LA areas that fall under the region and compare the wellbeing measures with AfR, Income and House Price.

For the future of this project I would be interested in working with data from English Housing Survey on dwellings and house standards, energy efficiency. And analyse similar surveys in Wales and Scotland to create visualisations showing these changes by regions. This kind of comprehensive visualisations can help first-time buyers to form a decision on where to buy a home and what other factors to consider.

{|validation)}

{(references|}

Cleveland, W. (1993) _The Elements of Graphing Data_, Hobart Press, ISBN 0963488411

Franconeri, S., Padilla, L., Shah, P., Zacks, J. and Hullman, J. (2021) [The science of visual data communication: What works](https://journals.sagepub.com/doi/full/10.1177/15291006211051956), _Psychological Science in the Public Interest_, 22(3) pp.110-161.

GOV.UK. (2022) _English Housing Survey_. Available at: https://www.gov.uk/government/collections/english-housing-survey (Accessed: 19 April 2023)

House of Commons Library. (2019) _Tackling the under-supply of housing_. Available at:
https://researchbriefings.files.parliament.uk/documents/CBP-7671/CBP-7671.pdf (Accessed: 5 April 2023)

Kirk, A. (2019) Chapter 7: Interactivity, pp.203-230, in _Data Visualisation: A Handbook for Data Driven Design, Sage_.

Kirk, A. (2019) Chapter 10, Composition, pp.277-293 in Data Visualisation: _A Handbook for Data Driven Design, 2nd Edition, Sage_.

Office for National Statistics. (2022) _Personal well-being in the UK: April 2021 to March 2022_. Available at: https://www.ons.gov.uk/peoplepopulationandcommunity/wellbeing/bulletins/measuringnationalwellbeing/latest (Accessed: 12 April 2023)

Office for National Statistics. (2023) _Housing, England and Wales: Census 2021_. Available at: https://www.ons.gov.uk/peoplepopulationandcommunity/housing/bulletins/housingenglandandwales/census2021 (Accessed: 6 April 2023)

Office for National Statistics. (2023) _Housing affordability in England and Wales: 2022_. Available at: https://www.ons.gov.uk/peoplepopulationandcommunity/housing/bulletins/housingaffordabilityinenglandandwales/latest (Accessed: 6 April 2023)

Rodríguez, M.T., Nunes, S. and Devezas, T., (2015) Telling stories with data visualization. _In Proceedings of the 2015 Workshop on Narrative & Hypertext (pp. 7-11)_.

Satyanarayan, A., Moritz, D., Wongsuphasawat, K. and Heer, J. (2017) [Vega-Lite: A Grammar of Interactive Graphics](https://files.osf.io/v1/resources/mqzyx/providers/osfstorage/5be5e643d354e900197998bd?action=download&version=1&direct&format=pdf). _IEEE Transactions on Visualization and Computer Graphics,_ 23(1) pp. 341-350.

{|references)}
