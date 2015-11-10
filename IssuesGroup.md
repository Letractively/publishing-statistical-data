## Groups ##

The SDMX Information Model supports a notion of Groups which group TimeSeries or Sections within an overall DataSet container.

The notion of groups is slightly different in the two cases. In the TimeSeries case the Group is indicated by a GroupKey which serves to collect timeseries matching that partial key for the purposes of attaching attributes.

In the Section Case (SDMX IM, p.75) then Sections don't have full keys and the Group provides the rest of the Key as well as providing a place to attach attributes.

The IM model has a fixed containment hierarchy for Cross Sections:
```
XSDataSet > Group > Section > XSObservation
```
whereas with TimeSeries the GroupKey level is optional.

## Vocabulary extensions ##

We need to:
  * represent the containment structure
  * attach dimensions (in the case of Groups of Sections) and attributes to Groups
  * decide how attachment levels work and whether the Group attachments are replicated downwards

### Containment structure ###

We already have `sdmx:Group` as a grouping structure but no property for representing the containment hierarchy.

**Suggestion:** generalize the domain of `sdmx:slice` to include both `sdmx:DataSet` and `sdmx:Group`.

An alternative is introduce more specific relations `sdmx:group` (`sdmx:DataSet` -> `sdmx:Group`) and `sdmx:part` (`sdmx:Group` -> `sdmx:Section` or `sdmx:TimeSeries`).

### Attaching dimensions and attributes ###

Nothing more needed. The domain of instances of `sdmx:ComponentProperty` is already open so we simply document that they can be attached to Groups to denote attributes and dimensions which are shared across all the TimeSeries/Sections (and enclosed observations) within the Group.

### Attachment levels ###

Our resolution to [issue 12](https://code.google.com/p/publishing-statistical-data/issues/detail?id=12) already says that we publish attributes at the attachment level called for in the DSD but that tool chains may replicate the information down to the observations for consumption purposes.

Currently [Issue 12](https://code.google.com/p/publishing-statistical-data/issues/detail?id=12) proposes `sdmx:FlatDataSet` and `sdmx:TimeSeriesDataSet` to distinguish the flat and succinct cases. However, with Groups we have DataSets than contain Groups of either Sections or TimeSeries so something needs to change.

**Suggestion:** Simply rename `sdmx:TimeSeriesDataSet` to `sdmx:HierarchicalDataSet` and say a `sdmx:HierarchicalDataSet` can contain optional nested `sdmx:Group`s and attributes and dimensions will be attached at the level called for in the DSD.

An alternative would be to add a pair classes `smdx:Flat` and `sdmx:Hierarchical` that can be used separately from `sdmx:DataSet`. This would have the advantage that it could be attached to Groups and Sections/TimeSeries as well so that you can tell locally that you have to look at the containing context to determine the full set of dimensions and attributes.

For declaring the attachment level in the DSD we do have to resolve [Issue 30](https://code.google.com/p/publishing-statistical-data/issues/detail?id=30). Note that the proposal for [Issue 36](https://code.google.com/p/publishing-statistical-data/issues/detail?id=36) (IssueMetadata) provides for structured entries (`sdmx:ComponentSpecification`) within the component list of a DSD (`sdmx:componentOrder`). So if we adopt that approach then we would just need an `sdmx:attachmentLevel` property which linked to the class denoting the level at which to attach.

## Worked example ##

Looking at the `CrossSectionalSample.xml` from SDMX 2.0 Section 3b we have source data (in compact form):

```
<biscs:DataSet>
  <biscs:Group  TIME="2000" BIS_UNIT="USD" UNIT_MULT="5" DECIMALS="2" AVAILABILITY="A" FREQ="A" >
      <biscs:Section COLLECTION="B" TIME_FORMAT="P1Y">
      <biscs:STOCKS  JD_CATEGORY="A" value="3.14" OBS_STATUS="A" VIS_CTY="MX"/>
      <biscs:FLOWS  JD_CATEGORY="A" value="1.00" OBS_STATUS="A" VIS_CTY="MX"/>
      <biscs:STOCKS  JD_CATEGORY="B" value="6.39" OBS_STATUS="A" VIS_CTY="MX"/>
      <biscs:FLOWS  JD_CATEGORY="B" value="2.27" OBS_STATUS="A" VIS_CTY="MX"/>
      <biscs:STOCKS  JD_CATEGORY="C" value="2.34" OBS_STATUS="A" VIS_CTY="MX"/>
      <biscs:FLOWS  JD_CATEGORY="C" value="-1.00" OBS_STATUS="A" VIS_CTY="MX"/>
      <biscs:STOCKS  JD_CATEGORY="D" value="3.19" OBS_STATUS="A" VIS_CTY="MX"/>
      <biscs:FLOWS  JD_CATEGORY="D" value="-1.06" OBS_STATUS="A" VIS_CTY="MX"/>
    </biscs:Section>
  </biscs:Group>
</biscs:DataSet>
```

Under the simplest version of this proposal this would become:

```
bis:data1 a sdmx:HierarchicalDataSet, sdmx:DataSet;
    sdmx:slice  bis:group1 .

bis:group1 a sdmx:Group;   
    bis:dimensionTime 2000;
    bis:attributeUnit "USD";
    bis:attributeMult 5;
    bis:attributeDecimals 2;
    bis:attributeAvailability "A";
    sdmx:slice bis:section1 .

bis:section1 a sdmx:Section;
    bis:dimensionCollection "B";
    bis:attributeTImeFormat "P1Y";
    sdmx:observation bis:data1obs1, ... .

bis:data1obs1 a sdmx:Observation;
    bis:dimensionStockOrFlow bis:STOCKS;
    bis:dimensionsJDCategory "A";
    bis:dimensionsVisCity "MX";
    bis:attributeObsStatus "A" .
```

Note that several of those components (`dimensionTime`, `attributeUnit`, `attributeMult`, `attributeDecimals`) are covered by COG common concepts and might be better represented in the RDF using the corresponding sdmx-attribute and sdmx-dimension component properties.