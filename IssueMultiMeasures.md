## Background ##

This concerns the representation of datasets that contain multiple measures.
In particular how the data cube vocabulary can be applied to non-statistical
datasets in which each individual "observation" would normally be thought of as
having multiple measured facets  (e.g. the weight and cost of a single shipment as opposed to aggregate statistical measures
of shipment weights and costs). Such situations arise with OLAP cubes,
sensor measurements, pollution sampling measurements etc.

The background is given in [Issue 40](https://code.google.com/p/publishing-statistical-data/issues/detail?id=40).

## Principles ##

  * Any generalizations of the Data Cube vocabulary should not prevent its specialization to SDMX.

  * The set of dimensions in the DSD should continue to uniquely identify a single `Observation`.

  * The measures should (continue to) be represented as specializations of `qb:ComponentProperty` and so be RDF properties. We should be able to use these properties directly on observations.

  * It should be possible, indeed straight forward, to model statistical sets with multiple measures using the SDMX approach of having a dimension which selects the measure and a single measure value per observation. We might call this the **measure dimension** (MD) modelling approach.

  * It should also be possible, for non-statistical sets, to permit multiple measures to be attached to a single observation if that is the appropriate modelling for that dataset. This generality would not be available within the SDMX specalization. We might call this the **multi-measure observations** (MMO) modelling approach.

  * The DSD structure should match the RDF structure. So if we declare a measure on the DSD it should be usable on observations within datasets which use that DSD.

  * We want to avoid redundant duplication with the DSD. In particular in the RDF representation of the MD approach declaring the measures in the DSD should be sufficient and it should not be **necessary** (though permitted) to define a code list for the measure dimension (since in the RDF case those declarations in fact make up the code list). On the other hand, in SDMX-ML DSDs this duplication is already present, so it's no major concern for the RDF representation of SDMX DSDs.

## Design ##

### Basics ###

Measures are instances of `qb:MeasureProperty` which in turn is a sub-class of `qb:ComponentProperty`.

If a measure is declared within the DSD then each observation in the corresponding data sets is able to use that property to denote the measured value.
Thus defining a measure `eg:lifeExpectancy` means that an observation would be able to directly state facts like:
```
  dataset:obs1 a qb:Observation;
      # .. dimensions and attributes 
      eg:lifeExpectancy 71.5 ;
      .
```

### Measure dimension data sets ###

For a statistical data set, especially ones already available as SDMX, or for other similar data sets, one can use the "measure dimension" representation.

In the DSD the publisher should declare a dimension to act as the measure dimension. This additional dimension is needed to ensure that we can address separate observations for each measure uniquely within the cube.

The measure dimension will be an instance of `qb:MeasureDimensionProperty` (a sub class of `qb:DimensionProperty`). This measure dimension may have a concept and code list associated with it in the usual way.

For convenience we provide a single pre-defined qb:MeasureDimensionProperty `qb:measureType`. For data sets not already in SDMX there may not be an existing code list corresponding to the enumeration of measures. In the special case of using `qb:measureType` as the measure dimension, the set of allowed measures is assumed to be those measures declared within the DSD. There is no need to define a separate code list or enumerated class to duplicate this information. Thus, `qb:measureType` is a “magic” dimension property with an implicit code list.

In this form of data set the selected measure on each observation remains that of the corresponding RDF property.

For example, given a DSD like:
```
eg:dsd1 a qb:DataStructureDefinition;
    rdfs:comment "shippments by time"@en;
    qb:component 
        [ qb:componentProperty  sdmx-dimension:refTime; ],
        [ qb:componentProperty  eg-measure:quantity; ],
        [ qb:componentProperty  eg-measure:weight; ],
        [ qb:componentProperty  qb:measureType; ];
    .
```

Then a typical set of observations might look like:
```
eg:obs1  a qb:Observation;
    qb:dataSet eg:dataset1;
    sdmx-dimension:refTime "30-07-2010"^^xsd:date;
    qb:measureType eg-measure:quantity;
    eg-measure:quantity 42 .

eg:obs2  a qb:Observation;
    qb:dataSet eg:dataset1;
    sdmx-dimension:refTime "30-07-2010"^^xsd:date;
    qb:measureType eg-measure:weight;
    eg-measure:weight 1.3 .

eg:dataset1 a qb:DataSet;
    qb:structure eg:dsd1 .
```

Note the duplication of having the measure property show up both as the property that carries the measured value, and as the value of the measure dimension. We accept this duplication as necessary to ensure the uniform cube/dimension mechanism **and** a uniform way of declaring and using measure properties on all kinds of datasets.

Note that in the RDF representation there is no need for a separate "primary measure" which subsumes each of the individual measures, those individual measures are used directly. We'll cover how round tripping of the primary measure is handled shortly.

### Multiple measure observations ###

If a data set publisher wishes to allow multiple measures on a single observation (e.g. for raw sensor data or individual shipment data) then they can do so. You simply declare all the measures but provide no measure dimension (that is no, instance of  `qb:MeasureDimensionProperty`).

[Aside. Note that interpretation of a DSD will require closed world reasoning.]

In that case the above example would look like:

```
eg:dsd2 a qb:DataStructureDefinition;
    rdfs:comment "shippments by time with multiple measures"@en;
    qb:component 
        [ qb:componentProperty  sdmx-dimension:refTime; ],
        [ qb:componentProperty  eg-measure:quantity; ],
        [ qb:componentProperty  eg-measure:weight; ];
    .

eg:obs2a  a qb:Observation;
    qb:dataSet eg:dataset2;
    sdmx-dimension:refTime "30-07-2010"^^xsd:date;
    eg-measure:weight 1.3 ;
    eg-measure:quantity 42 ;
    .

eg:dataset2 a qb:DataSet;
    qb:structure eg:dsd2 .
```

In documenting the vocabulary we would be clear on the pros- and cons- of using MD v. MMO type modelling. Including issues of SDMX compatibility, the fundamental nature of the data (record structured v. independent measures) and the ability to directly address and annotate individual measured values as opposed to just observations.

### SDMX Specialization ###

In the SDMX vocabulary we will define `sdmx:DataStructureDefinition` as a subclass of `qb:DataStructureDefinition`.

A data set corresponding to an `sdmx:DataStructureDefinition` has to meet the additional constraint that each observation only has a single measured value - either because the set only declares a single measure or because it declares a measure dimension.

Furthermore, if multiple measures are declared, then one of the declared dimension properties must be a qb:MeasureDimensionProperty, and the value of this dimension on each observation must match the measure property present on the observation.

There may be other constraints on `sdmx:DataStructureDefinition` to ensure SDMX compatibility:

  * presence of certain slices
  * no sparse cubes
  * certain constraints on attachment levels for attributes, dimensions and properties

In the case of a data set with N measures an SDMX DSD will typically declare N + 1 measures, the N individual measures and 1 additional Primary Measure, typically OBS\_VALUE.
In the RDF representation the primary measure is not needed on the individual observations but needs to be recorded in the DSD to enable round tripping. We will enable this by providing `sdmx:primaryMeasure` which links a `sdmx:DataStructureDefinition` to a `qb:MeasureProperty`.

### Attributes ###

In the measure dimension approach (or where there is only a single measure anyway) then attributes such as `sdmx-attribute:unitMeasure` and `sdmx-attribute:unitMultiplier` can be attached to individual Observations or to higher attachment levels  - Slice, DataSet or the MeasureProperty itself.

In the multiple measures (MMO) approach then those attributes can only be usefully attached to the MeasureProperty.

If a dataset requires attribute attachment to an individual measure on an individual observation (e.g. to indicate a estimated instead of a measured value) then the SDMX-compatible "measure dimension" representation must be used.

### Slicing ###

Using the measure dimension approach it is possible to define a slice of observations corresponding to a single measure (i.e. fixing the value of the measure dimension).

We contemplated machinery to support this even within an MMO model (through the use of a magic dimension in slice keys and slice instances) but rejected this for now. If you need to slice along measures then use the "measure dimension" representation. We will document this as a possible future extension point depending on experience with the vocabulary.

### DSD aliases ###

As an aside we noted that the current DSD structure does not make the nature of the different components immediately clear (they are defined by the `rdf:type` of the components. This is especially of concern where one re-uses components declared elsewhere, as their `rdf:type` may not be apparent from reading just the DSD.

We may want to introduce sub-properties of `qb:componentProperty` (`qb:measure`, `qb:dimension`, `qb:attribute` and `qb:measureDimension`) with restricted ranges. This would make DSDs more easily readable at a small cost of redundancy.

We haven't decided on this and it is not critical path.