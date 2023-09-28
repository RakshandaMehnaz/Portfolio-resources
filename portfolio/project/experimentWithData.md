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

For housing affordability data by local authority area in England and Wales including the annual income, annual house price and the ratio of house price to income - considering representing it in geomap distributed by local authority area for further detail. The map will be colour coded to categorise these four Tenure types:

- Owns outright
- Owns with mortgage/loan/shared ownership
- Socially rented
- Private rented/Rent free

Affordability ratio (House price to Income) data can be visualised by region (Total 10 regions in England and Wales). Interaction with map data can show the local authority that falls under the highlighted region.

The tenure data representation of 2011 and 2021 have some inconsistencies - the grouping of Tenure type, changes in local authority area...

The 2011 Tenure data has the following breakdown of ownership:

- Owned (sum of owned outright and with mortgage/loan)
- Owned outright
- Owned with a mortgage or loan
- Owned with shared ownership

Here I've added the shared ownership values to the Owned with a mortgage or loan category. This is because the 2021 data was already grouped this way and separate value of shared ownership was very small. Also, I've tidied the data manually in Excel spreadsheet and converted to CSV before visualising it. (Tenure types in one column and Values in one column)

Other potential factors affecting housing and affordability: To analyse factors that contributed to the changes in housing affordability, I consulted a combination of datasets and studies - The English Housing Survey, Welsh Housing Data.

Another consideration was to investigate the housing quality by Decent Home Standard, Energy Efficiency Rating (EER), dwelling condition. These information were found in the English Housing Survey (EHS) data on dwelling condition and safety for England and Welsh Housing Quality Standard (WHQS) for Wales. Both these websites contained data over the years and updated till 2021 which is the year I was looking to compare. The idea was to analyse the changes over the last ten years.

I also looked into the Indices of Multiple Deprivation (IMD) which is a measure of relative deprivation for small areas in England. It includes a domain on barriers to housing and services to help evaluate changes in equitable access to housing. However, the latest release of this data was on 2018. Whereas, I was interested to analyse data for 2021.

EHS 2011-2012 Key findings:

- growth in private rented sector and around 65% of households were owner occupiers.
- avg weekly rent in private rented sector continued to be well above the social rented sector (GBP 164 vs GBP 83)
- about 64% of social renters received Housing Benefit, whereas about 26% for private renters.
- energy efficiency of housing stock continued to improve, avg SAP rating. the social rented sector had the highest EER bands A to C. (34% housing association and 26% of local authority dwellings)
- about 24% of dwellings were non-decent. Lowest rate in social renting 17% and highest rate in private renting 35%.
- damp problems in dwellings were about 5%. Private rentals were more likely than those in other tenures to have damp problems as they were more likely to be older stock.

EHS 2021-2022 Key findings:

- about 64% owner occupied households. So has been similar since the last decade.
- about 35% of households had outright owners and 30% were mortgagers.
- private rented sector consists of about 19% of households.
- social rented sector accounts for 17% of households in England. over the last decade, social housing provision has increasingly been supplied by Housing Associations.
- private rented sector continues to hold the highest proportion of non-decent dwellings, about 14% failed to meet Decent Homes Standard. About 11% private rented households reported to have damp problems.
- SAP scores increased overall. Highest for social housing, then for owner occupied dwellings and the least for private rentals.

To explore if housing affordability across regions and the wellbeing measures have any relationship or pattern. A bar chart to visualise the four measures of wellbeing - life satisfaction, worthwile, happiness, anxiety - by region for the year 2011 to 2012 and 2021 to 2022. It would also be useful to create interaction between the wellbeing mean score and the scatterplot with AfR data points.

#Visualisation 1

```elm {l=hidden v interactive}
tenureMaps2021 : Spec
tenureMaps2021 =
    let
        tenureData =
            dataFromUrl "https://raw.githubusercontent.com/RakshandaMehnaz/Data-Viz-datasets/main/Tenure%202021%20Local%20Auth.csv" []

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
                    [ mName "Value"
                    , mQuant
                    , mLegend
                        [ leTitle "Residents"
                        , leOrient loNone
                        , leX 350
                        , leY 650
                        , leDirection moHorizontal
                        ]
                    ]
                << tooltips
                    [ [ tName "label", tTitle "Local Authority Area" ]
                    , [ tName "Value", tTitle "Number of Residents" ]
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
        , facetFlow [ fName "Type", fHeader [ hdTitle "2021" ] ]
        , specification (asSpec [ width 400, height 300, enc [], geoshape [ maStroke "white" ] ])
        ]
```

#Visualisation 2

```elm {l=hidden v interactive}
tenureMaps2011 : Spec
tenureMaps2011 =
    let
        tenureData =
            dataFromUrl "https://raw.githubusercontent.com/RakshandaMehnaz/Data-Viz-datasets/main/Tenure2011LocalAuth.csv" []

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
                    [ mName "Value"
                    , mQuant
                    , mLegend
                        [ leTitle "Residents"
                        , leOrient loNone
                        , leX 350
                        , leY 650
                        , leDirection moHorizontal
                        ]
                    ]
                << tooltips
                    [ [ tName "label", tTitle "Local Authority Area" ]
                    , [ tName "Value", tTitle "Number of Residents" ]
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
        , facetFlow [ fName "Type", fHeader [ hdTitle "2011" ] ]
        , specification (asSpec [ width 400, height 300, enc [], geoshape [ maStroke "white" ] ])
        ]
```

#Visualisation 3

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
                        [ mName "Region name ", mScale [ scScheme "magma" [ 0.2, 0.6 ] ] ]
                        [ mStr "lightgrey" ]
                    ]
                << size
                    [ mName "Affordability Ratio", mScale [ scRange (raNums [ 0, 400 ]), scType scPow, scExponent (1 / 0.7) ] ]
    in
    toVegaLite [ width 650, height 480, data, ps [], enc [], circle [] ]
```

#Visualisation 4

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
                    , [ tName "Median House Earnings" ]
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

#Visualisation 5

```elm {v}
wellbeingBar : Spec
wellbeingBar =
    let
        data =
            dataFromUrl "https://raw.githubusercontent.com/RakshandaMehnaz/Data-Viz-datasets/main/WellbeingData2011and2021v2.csv" []

        enc =
            encoding
                << position X [ pName "Region", pNominal ]
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
                << position X [ pName "Region", pNominal ]
                << position XOffset [ pName "Measure" ]
                << position Y [ pName "Percentage", pQuant ]
                << color [ mName "Measure" ]
    in
    toVegaLite [ width 500, height 250, widthStep 15, data, enc [], bar [] ]
```

#Interaction between Tenure Dist and Affordability Ratio

Affordability data without the Geo coordinates

```elm {l v interactive}
associative : Spec
associative =
    let
        data =
            dataFromUrl "https://raw.githubusercontent.com/RakshandaMehnaz/Data-Viz-datasets/main/AffordabilityDataEngWales.csv" []

        enc =
            encoding
                << position X [ pName "Year", pAxis [ axLabelAngle 0 ] ]
                << position XOffset [ pName "Median House Earnings" ]
                << position Y [ pName "Median House Price", pQuant ]
                << color [ mName "Region name " ]
    in
    toVegaLite [ width 500, height 350, data, enc [], circle [] ]
```

```elm {l v interactive}
tenureMaps : Spec
tenureMaps =
    let
        tenureData =
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
                        , leX 720
                        , leY 600
                        , leDirection moHorizontal
                        ]
                    ]
                << tooltips
                    [ [ tName "Local authority name" ]
                    , [ tName "Median House Price" ]
                    , [ tName "Median House Earnings" ]
                    , [ tName "Affordability Ratio" ]
                    ]

        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
                << configuration (coFacet [ facoSpacing 2 ])
    in
    toVegaLite
        [ cfg []
        , tenureData
        , trans []
        , columns 2
        , facetFlow [ fName "Year", fHeader [ hdTitle "2011 vs 2021" ] ]
        , specification (asSpec [ width 550, height 600, enc [], geoshape [] ])
        ]
```

Attempt at creating field interactions for Tenure Data;

2021

```elm {l v interactive}
tenureMaps2 : Spec
tenureMaps2 =
    let
        tenureData =
            dataFromUrl "https://raw.githubusercontent.com/RakshandaMehnaz/Data-Viz-datasets/main/CensusMapSiteData.csv" []

        boundaryData =
            dataFromUrl "https://gicentre.github.io/data/census21/ewLAs.json"
                [ topojsonFeature "las" ]

        ps =
            params
                << param "measure"
                    [ paValue (str "Year")
                    , paBind
                        (ipSelect
                            [ inName "Year"
                            , inOptions [ "2011", "2021" ]
                            ]
                        )
                    ]

        trans =
            transform
                << calculateAs "datum[measure]" "y"
                << lookup "label" boundaryData "properties.label" (luAs "geo")

        enc =
            encoding
                << shape [ mName "geo", mGeo ]
                << color
                    [ mName "Value"
                    , mQuant
                    , mLegend
                        [ leTitle "Residents"
                        , leOrient loNone
                        , leX 520
                        , leY 650
                        , leDirection moHorizontal
                        ]
                    ]
                << tooltips
                    [ [ tName "label" ]
                    , [ tName "Value" ]
                    ]

        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
                << configuration (coFacet [ facoSpacing 2 ])
    in
    toVegaLite
        [ cfg []
        , tenureData
        , trans []
        , ps []
        , columns 2
        , facetFlow [ fName "Type", fHeader [ hdTitle "" ] ]
        , specification (asSpec [ width 400, height 300, enc [], geoshape [] ])
        ]
```
