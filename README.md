# Deneb-Templates
Vega-Lite Power BI visual templates for use in Deneb
Written in Vega or Vega-Lite, for use with the [Deneb](https://deneb-viz.github.io/) custom visual in Power BI.

## Choropleth QFES Regions
Map visual written in Vega-Lite. Geographic boundaries are hard-coded, to support usage by the Microsoft Certified version of Deneb available via AppSource. 

![image](https://private-user-images.githubusercontent.com/106286328/269844719-6c41569f-4d88-40a9-ac0b-11c95f0cd6bb.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTEiLCJleHAiOjE2OTUzNjMxOTMsIm5iZiI6MTY5NTM2Mjg5MywicGF0aCI6Ii8xMDYyODYzMjgvMjY5ODQ0NzE5LTZjNDE1NjlmLTRkODgtNDBhOS1hYzBiLTExYzk1ZjBjZDZiYi5wbmc_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBSVdOSllBWDRDU1ZFSDUzQSUyRjIwMjMwOTIyJTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDIzMDkyMlQwNjA4MTNaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT03N2Q2ZDc2NTY4YzBkMzUzY2MzMzBmOWY2MDI3NDMzOWNlMDVlMDYxOGIwM2U3MjRmNDgzMTUzNjExN2EwMWQwJlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCZhY3Rvcl9pZD0wJmtleV9pZD0wJnJlcG9faWQ9MCJ9.i8Pwrrh_DxB17ZIlqOkFJJVUDuOy9FZwZ6jcEy9DU8Y)

#### Usage
Regions are highlighted by some measure defined in Power BI. The simplest implementation of this is to create a table in Power BI, via DAX or Power Query, with a single row for each Region, and a column of data to be presented. If one is familiar with Vega-Lite, the code can be easily adjusted to support additional data layers (eg, labels), colours & formatting, or to change the spacial join field (Region Name, Region Number, and Firecom Name are currently supported). 

Note, Regions have been simplified by exclusion of all islands and by algorithm in R `{rmapshaper}` to 20%. Regions presented here may therefore differ to those publicly available on Queensland Open Data Portal. 

#### Geography properties: Region Name - Region Number - Firecom Name
- Northern - 1 - Townsville
- Central - 2 - Rockhampton
- South Western - 3 - Toowoomba
- North Coast - 4 - Kawana
- Brisbane - 5 - Brisbane
- South Eastern - 6 - Southport
- Far Northern - 7 - Cairns

# Deneb Template - Histogram with Verts
A Deneb template using Vega-Lite. Histogram with vertical lines for member values. For use in Power BI. 

For a given metric and group, plot the histogram of the metric with vertical lines for specified member values. Allows an individual value to be compared against the overall distribution of a metric, potentially useful for identifying outliers or assessing overall utility of a metric.

![plot](https://user-images.githubusercontent.com/106286328/264239646-a95161f0-8727-4139-b970-7e8c3792e56e.png)
*Note, the data to generate this image has been randomised and does not reflect the true distrubtion of the shown metric.*

This assumes the following features of a Power BI model:
- Table with data
- Table of unique group members, taken from the data table above. Append a value not found in the list to "turn off" this layer in Deneb. This is then used as a slicer for the vertical lines only, preserving the data in the original table. Example using DAX (note that ALLEXCEPT can be used instead of ALL to include any slicers on the page):
```DAX
Deneb_Member_List = 
UNION(
    // Get list of all LGAs for use in slicer
    DISTINCT(
        ALL( Incidents[Member_name] )
    ),
    // Add option to turn off this layer in Deneb
    ROW( "Member_name", "_Turn off_" )
)
```
- For a given metric, three measures are created.
1. One to determine the value for all available rows in the data table.
```DAX
Num_incidents = DISTINCTCOUNT( Incidents[IncidentID] )
```
2. Another to determine the value for only the rows available in the unique group member table.
```DAX
Deneb_vert = 
CALCULATE(
    [Num_incidents],
    FILTER(
        Incidents,
        Incidents[Member_name] IN VALUES( Deneb_Member_List[Member_name] )
    )
)
```
3. A measure for the label displayed with the vertical line.
```DAX
Deneb_vert_label = 
FIRSTNONBLANK(
    VALUES( Deneb_Member_List[Member_name] )
    , 1 ) & 
    " (" & 
    FORMAT( [Deneb_vert] , "#,##0" )
    & ")"

```
