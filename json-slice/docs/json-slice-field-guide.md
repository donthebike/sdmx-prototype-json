# A draft field guide to json-slice objects

**json-slice format** **v0.3.0**

- [Introduction](#Introduction)
- [Message](#Message)
- [Header](#Header)
- [Structure](#Structure)
- [DataSets](#DataSets)
- [Tutorial: Handling component values](#handling_values)

----

## <a name="Introduction"></a>Introduction
Let's first start with a brief introduction of the SDMX information model.

In order to make sense of some statistical data, we need to know the concepts
associated with them. For example, on its own the figure 1.2953 is pretty meaningless,
but if we know that this is an exchange rate for the US dollar against the euro on
23 November 2006, it starts making more sense.

There are two types of concepts: dimensions and attributes. Dimensions, when combined,
allow to uniquely identify statistical data. Attributes on the other hand do not help
identifying statistical data, but they add useful information (like the unit of measure
or the number of decimals). Dimensions and attributes are known as "components".

The measurement of some phenomenon (e.g. the figure 1.2953 mentioned above) is known as an
"observation" in SDMX. Observations are grouped together into a "data set". However, there
can also be an intermediate grouping. For example, all exchange rates for the US dollar
against the euro can be measured on a daily basis and these measures can then be
grouped together, in a so-called "time series". Similarly, you can group a collection of
observations made at the same point in time, in a "cross-section" (for example,
the values of the US dollar, the Japanese yen and the Swiss franc against the euro at a
particular date). Of course, these intermediate groupings are entirely optional and you
may simply decide to have a flat list of observations in your data set.

The SDMX information model is much richer than this limited introduction,
however the above should be sufficient to understand the json-slice format. For
additional information, please refer to the [SDMX documentation](http://sdmx.org/?page_id=10).

Before we start, let's clarify a few more things about this guide:
* New fields may be introduced in later versions. Therefore
consuming applications should tolerate the addition of new fields with ease.
* The ordering of fields in objects is undefined. The fields may appear in any order
and consuming applications should not rely on any specific ordering. It is safe to consider a
nulled field and the absence of a field as the same thing.
* Not all fields appear in all contexts. For example response with error messages
may not contain fields for data, dimensions and attributes.

----

## <a name="Message"></a>Message

Message is the top level object and it contains the data as well as the metadata needed to interpret those data.
Example:

    {
      "header": {
        "id": "b1804c51-1ee3-45a9-bb75-795cd4e06489",
        "prepared": "2012-05-04T03:30:00"
      },
      "structure": {
        # structure objects #
      },
      "dataSets": [
        # data set objects #
      ],
      "errors": null
    }

### header

*Object* *nullable*. *Header* contains basic information about the message, such as when it was prepared and
how has sent it. Example:

    "header": {
      "id": "b1804c51-1ee3-45a9-bb75-795cd4e06489",
      "prepared": "2012-05-04T03:30:00"
    }

### structure

*Object* *nullable*. *Structure* contains the information needed to interpret the data available in the message,
such as the list of concepts used. Example:

    "structure": {
        "id": "ECB_EXR_WEB",
        "href": "http://sdw-ws.ecb.europa.eu/dataflow/ECB/EXR/1.0",
        "dimensions": {
            # dimensions object #
        },
        "attributes": {
            # attributes object #
        }
    }

### dataSets

*Array* *nullable*. *DataSets* field is an array of *[DataSet](#DataSet)* objects. That's where the data (i.e.: the observations)
will be. Example:

    "dataSets": [
      {
        "action": "Informational",
        "extracted": "2012-05-04T03:30:00",
        "series": [
          # data object #
        ]
      }
    ]

In typical cases, the file will contain only one data set. However, in some cases, such as when retrieving,
from an SDMX 2.1 web service, what has changed in the data source since in particular point in time, the web
service might return more than one data set.

### errors

*Array* *nullable*. RESTful web services indicates errors using the HTTP status
codes. In addition, whenever appropriate, the error can also be returned using
this error field. Error is an array of error messages. Example:

    "errors": [
      "Invalid number of dimensions in parameter key"
    ]

----


## <a name="Header"></a>header

Header contains basic information about the message, such as when it was prepared and
how has sent it. Example:

    "header": {
      "id": "b1804c51-1ee3-45a9-bb75-795cd4e06489",
      "prepared": "2013-01-03T12:54:12",
      "sender: {
        "id": "SDMX"
      }
    }

### id

*String*. Unique string that identifies the message for further references.
Example:

    "id": "TEC00034"

### test

*Boolean* *nullable*. Indicates whether the message is for test purposes or not. False for normal messages. Example:

    "test": false

### prepared

*String*. A timestamp, formatted according to the ISO-8601 standard, indicating when the message was prepared. Example:

    "prepared": "2012-05-04T03:30:00Z"

### sender

*Object*. Information about the party that is transmitting the message. Sender contains the following fields:

* id - *String*. A unique identifier of the party.
* name - *String* *nullable*. A human-readable name of the party.
* contact - *Array* *nullable*. A collection of contact details.

Example:

    "sender": {
      "id": "ECB",
      "name": "European Central Bank"
      "contact": [
        # contact details #
      ]
    }

#### contact

*Array* *nullable*. Information on how the party can be contacted.

Each object in the collection may contain the following field:
* name - *String*. The contact's name.
* department - *String* *nullable*. The organisational structure within which the contact person works.
* role - *String* *nullable*. The responsibility of the contact person.
* telephone - *String* *nullable*. The telephone number for the contact person.
* fax - *String* *nullable*. The fax number for the contact person.
* x400 - *String* *nullable*. The X.400 address for the contact person.
* uri - *String* *nullable*. URI holds an information URL for the contact person.
* email - *String* *nullable*. The email address for the contact person.

Example:

    "contact": [
        {
            "name": "Statistics hotline",
            "email": "statistics@xyz.org"
        }
    ]

### receiver

*Object* *nullable*. Information about the party that is receiving the message. This can be useful is a scenario of
bilateral data exchanges. Receiver contains the same fields as sender (see above):

* id - *String*. A unique identifier of the receiver.
* name - *String* *nullable*. A human-readable name of the party.
* contact - *Array* *nullable*. A collection of contact details.

Example:

    "receiver": {
      "id": "SDMX"
    }

### extracted

*String* *nullable*. A timestamp, formatted according to the ISO-8601 standard, indicating when the data have been extracted from the data source.
Example:

    "extracted": "2012-05-04T03:30:00Z"

### embargoDate

*String* *nullable*. EmbargoDate holds an ISO-8601 time period before which the data included in this message are not available. Example:

    "embargoDate": "2012-05-04"

### source

*Array* *nullable*. Human-readable information about the source of the data. Example:

    "source": [
      "European Central Bank and European Commission"
    ]

----

## <a name="Structure"></a>structure

*Object* *nullable*. Provides the structural metadata necessary to interpret the data contained in the message.
It tells you which are the components (dimensions and attributes) used in the message and also describes to which
level in the hierarchy (data set, series, observations) these components are attached.

Example:

    "structure": {
        "id": "ECB_EXR_WEB",
        "href": "http://sdw-ws.ecb.europa.eu/dataflow/ECB/EXR/1.0",
        "dimensions": {
            # dimensions object #
        },
        "attributes": {
            # attributes object #
        }
    }

### id

*String* *nullable*. An identifier for the structure. Example:

    "id": "ECB_EXR_WEB"

### href

*String* *nullable*. A link to an SDMX 2.1 web service resource where additional information regarding the structure is
available. Example:

    "href": "http://sdw-ws.ecb.europa.eu/dataflow/ECB/EXR/1.0"

### ref

*Object* *nullable*. Provides the 4 elements necessary to uniquely identify structural metadata that offers additional
information about the data contained in the message. Example:

    "ref": {
        "type": "dataStructure",
        "agencyID": "ECB",
        "id": "EXR",
        "version": 1.0
    }

The ref element may contain the following elements:

#### type

*String*. The type of structural metadata. The following values are allowed: dataStructure, dataflow, provisionAgreement.
For additional information about these, please refer to the [SDMX documentation](http://sdmx.org/?page_id=10). Example:

    "type": "dataStructure"

#### agencyID

*String*. The ID of the agency maintaining the structural metadata. Example:

    "agencyID": "ECB"

#### id

*String*. The ID of the structural metadata. Example:

    "id": "EXR"

#### version

*String* *nullable*. The version of the structural metadata. If not supplied, defaults to 1.0. Example:

    "version": 1.0

### dimensions

*Object*. Describes the dimensions used in the message as well as the levels in the hierarchy (data set, series,
observations) to which these dimensions are attached. Example:

    "dimensions": {
        "dataSet": [
            {
                # Component object #
            },
        ],
        "series": [
            {
                # Component object #
            }
        ],
        "observation": [
            {
                # Component object #
            }
        ]
    }

### attributes

*Object*. Describes the attributes used in the message as well as the levels in the hierarchy (data set, series,
observations) to which these attributes are attached. Example:

    "attributes": {
        "dataSet": [
            {
                # Component object #
            },
        ],
        "series": [
            {
                # Component object #
            }
        ],
        "observation": [
            {
                # Component object #
            }
        ]
    }

### <a name="Component"></a>Component

A component represents a dimension or an attribute used in the message. It contains basic information about the component
(such as its name and id) as well as the list of values used in the message for this particular component. Example:

    {
      "id": "FREQ",
      "name": "Frequency",
      "keyPosition": 1,
      "values": [
        {
          # value object #
        }
      ]
    }

Each of the components may contain the following fields

#### id

*String*. Identifier for the component.
Example:

    "id": "FREQ"

#### name

*String*. Name provides a human-readable name for the component.
Example:

    "name": "Frequency"

#### keyPosition

*Number*. Indicates the position of the dimension in the key, starting at 1. This field should not be supplied
for attributes.Example:

    "keyPosition": 1

#### description

*String* *nullable*. Provides a description for the component. Example:

    "description": "The time interval at which observations occur over a given time period."

#### role

*String* *nullable*. Defines the component role(s). For normal components the value
is null. Components can play various roles, such as, for example:

- **time**. Time dimension is a special dimension which designates the period in
time in which the data identified by the full series key applies.
- **measure**. Measure dimension is a special type of dimension which defines
multiple measures.

Example:

    "role": "time"

#### default

*String* or *Number* *nullable*. Defines a default value for the component (valid for attributes only!). If
no value is provided in the data part of the message then this value applies. Example:

    "default": "A"

#### values

Array of [values](#component_values) for the component. Example:

    "values": [
      {
        "id": "M",
        "name": "Monthly"
      }
    ]

#### <a name="component_values"></a>Component value

*Object* *nullable*. A particular value for a component in a message. Example:

    {
        "id": "M",
        "name": "Monthly"
    }

##### id

*String*. Unique identifier for a value. Example:

    "id": "A"

##### name

*String*. Human-readable name for a value. Example:

    "name": "Missing value; data cannot exist"

##### description

*String* *nullable*. Description provides a human-readable description of the value. The description is typically longer
than the text provided for the name field. Example:

    "description": "Provisional value"

##### parent

*String* *nullable*. Parent value (for code hierarchies). If parent is null then
the value does not belong to any hierarchy. Hierarchy root values have special
value "ROOT" for the parent. There may be multiple roots. Each value has only one
parent. Example:

    "parent": "U2"

##### start and end fields

*String* *nullable*. Start and end are instants of time that define an interval.
They mark the beginning and the end of the reporting period represented by the component value.
The length of the interval is the duration (i.e. duration = end - start).
These fields should be used only when the component value represents one of the values for the time dimension!

Values are considered as inclusive for the start field and as exclusive for the end field
(so-called half-open interval). Values must follow the ISO 8601 syntax for combined dates and times,
including time zone.

Example:

    {
        "id": "2010",
        "name": "2010",
        "start": "2010-01-01T00:00:00.000Z",
        "end": "2011-01-01T00:00:00.000Z"
    }

Duration can easily be calculated. Taking the example above, and using the times in milliseconds since Jan 1st 1970:
- start = 2010-01-01T00:00:00.000Z = 1262304060000
- end = 2011-01-01T00:00:00.000Z = 1293840060000
- duration = 1293840060000 - 1262304060000 = 31536000000 ms = 31536000 s = 525600 m = 8760 h = 365 days

These fields are useful for visualisation tools, when selecting the appropriate point in time for the time axis.
Statistical data, can be collected, for example, at the beginning, the middle or the end of the period, or can
represent the average of observations through the period. Based on this information and using the start and end
fields, it is easy to get or calculate the desired point in time to be used for the time axis.

----

## <a name="DataSets"></a>DataSets

An array of data set objects. Example:

    "dataSets": [
      {
        "action": "Informational",
        "extracted": "2012-05-04T03:30:00",
        "series": [
          # series object #
        ]
      }
    ]

There are between 2 and 3 levels in a data set object, depending on the way the data in the message is organized.

A data set may contain a flat list of observations. In this scenario, we have 2 levels in the data part of the message:
the data set level and the observation level.

A data set may also organize observations in logical groups called series. These groups can represent time series or
cross-sections. In this scenario, we have 3 levels in the data part of the message: the data set level, the
series level and the observation level.

Dimensions and attributes may be attached to any of these 3 levels.

In case the data set is a flat list of observations, observations will be found directly under a data set object. In
case the data set represents time series or cross sections, the observations will be found under the series elements.

### id

*String* *nullable*. Id provides an identifier for the data set. Example:

    "id": "ECB_EXR_2011-06-17"

### name

*String* *nullable*. A human-friendly name for the dataset. Example:

    "name": "Bilateral exchange rates"

### description

*String* *nullable*. Description provides a plain text, human-readable
description of the dataset. Example:

    "description": "The nominal effective exchange rate (EER) index is a summary measure of the external value of
    a currency vis-á-vis the currencies of the most trading partners, while the real EER - obtained by deflating
    the nominal rate with appropriate price or cost indices - is the most commonly used indicator of international
    price and cost competitiveness. Daily spot exchange rates provided by the Front Office Division."

### action

*String* *nullable*. Action provides a list of actions, describing the intention of the data transmission
from the sender's side. ```Default value is Informational```

* Append - this is an incremental update for an existing data set or the provision of new data or documentation
(attribute values) formerly absent. If any of the supplied data or metadata are already present, it will not replace
these data.

* Replace - data are to be replaced, and may also include additional data to be appended.

* Delete - data are to be deleted.

* Informational - data are being exchanged for informational purposes only, and not meant to update a system.

Example:

    "action": "Informational"

### provider

*Object* *nullable*. Provider is information about the party that provides the data contained in the data set.
Provider contains the following fields:

* id - *String*. A unique identifier of the party.
* name - *String* *nullable*. A human-readable name of the party.

Example:

    "provider": {
      "id": "EUROSTAT"
    }

### reportingBegin

*String* *nullable*. The start of the time period covered by the message. Example:

    "reportingBegin": "2012-05-04"

### reportingEnd

*String* *nullable*. The end of the time period covered by the message. Example:

    "reportingEnd": "2012-06-01"

### validFrom

*String* *nullable*. The validFrom indicates the inclusive start time indicating the validity of the information in the data.

    "validFrom": "2012-01-01T10:00:00Z"

### validTo

*String* *nullable*. The validTo indicates the inclusive end time indicating the validity of the information in the data.

    "validTo": "2013-01-01T10:00:00Z"

### publicationYear

*String* *nullable*. The publicationYear holds the ISO 8601 four-digit year.

    "publicationYear": "2005"

### publicationPeriod

*String* *nullable*. The publicationPeriod specifies the period of publication of the data in terms of whatever
provisioning agreements might be in force (i.e., "2005-Q1" if that is the time of publication for a data set
published on a quarterly basis).

    "publicationPeriod": "2005-Q1"

### dimensions

*Array* *nullable*. Collection of [dimensions values](#dimension) attached to the data set level. This is typically the case when a
particular dimension always has the same value for the data available in the data message. In order to avoid repetition,
that value can simply be attached at the data set level.

### attributes

*Array* *nullable*. Collection of [attributes values](#attributes) attached to the data set level. This is typically the case when a
particular attribute always has the same value for the data available in the data message. In order to avoid repetition,
that value can simply be attached at the data set level.

### observations

*Array* *nullable*. Collection of [observations](#observations) directly attached to a data set. This is the case when
a data set represents a flat collection of observations. In case the observations are organised into logical groups
(time series or cross-sections), use the [series element](#series) instead.

### <a name="series"></a> series

*Array* *nullable*. A collection of series. Each series object contains the observation values and associated metadata (dimensions and attributes), when
the observations contained in the data set are used into logical groups (time series or cross-sections). This element must
**not** be used in case the data set represents a flat list of observations. Example:

    {
      "dimensions": [ 0, 0, 0, 0, 0, 0 ],
      "attributes": [ 0, 1 ],
      "observations": [
        [ 0, 105.6, null, null ],
        [ 1, 105.9 ],
        [ 2, 106.9 ],
        [ 3, 107.3, 0 ]
      ]
    }

#### <a name="dimensions"></a>dimensions

*Array* *nullable*. An array of dimension values. Each value is an index to the
*values* array in the respective *Dimension* object.

    // Dimensions for series
    "dimensions": [
      0,
      93,
      12,
      1
    ]

For information on how to handle the dimension values, see the section dedicated to [handling component values](#handling_values).

#### <a name="attributes"></a>attributes

*Array* *nullable*. Collection of attributes values. Each value is an index to the
*values* array in the respective *Attribute* object. Example:

    "attributes": [ 0, 1 ]

For information on how to handle the attribute values, see the section dedicated to [handling component values](#handling_values).

#### <a name="observations"></a>observations

*Array* *nullable*. An array of observation values. Each observation value is an
array of two of more values.

    "observations": [
        [ 0, 105.6, null, null ],
        [ 1, 105.9 ],
        [ 2, 106.9 ],
        [ 3, 107.3, 0 ]
    ]

The first elements in an observation value array are the index values of the observation level dimensions. It's one
for time series and cross-sections, but there will be more than one when the data set represents a flat list of
observations.

Then comes the observation value. The data type for observation value is *Number*. Data type for a reported
missing observation value is a *null*.

Elements after the observation value are values for the observation level attributes.

----

## <a name="handling_values"></a>Handling component values

Let's say that the following message needs to be processed:

    {
        "header": {
            "id": "62b5f19d-f1c9-495d-8446-a3661ed24753",
            "prepared": "2012-11-29T08:40:26Z",
            "sender": {
                "id": "ECB",
                "name": "European Central Bank"
            }
        },
        "structure": {
            "id": "ECB_EXR_WEB",
            "href": "http://sdw-ws.ecb.europa.eu/dataflow/ECB/EXR/1.0",
            "dimensions": {
                "dataSet": [
                    {
                        "id": "FREQ",
                        "name": "Frequency",
                        "keyPosition": 1,
                        "values": [
                            {
                                "id": "D",
                                "name": "Daily"
                            }
                        ]
                    },
                    {
                        "id": "CURRENCY_DENOM",
                        "name": "Currency denominator",
                        "keyPosition": 3,
                        "values": [
                            {
                                "id": "EUR",
                                "name": "Euro"
                            }
                        ]
                    },
                    {
                        "id": "EXR_TYPE",
                        "name": "Exchange rate type",
                        "keyPosition": 4,
                        "values": [
                            {
                                "id": "SP00",
                                "name": "Spot rate"
                            }
                        ]
                    },
                    {
                        "id": "EXR_SUFFIX",
                        "name": "Series variation - EXR context",
                        "keyPosition": 5,
                        "values": [
                            {
                                "id": "A",
                                "name": "Average or standardised measure for given frequency"
                            }
                        ]
                    }
                ],
                "series": [
                    {
                        "id": "CURRENCY",
                        "name": "Currency",
                        "keyPosition": 2,
                        "values": [
                            {
                                "id": "NZD",
                                "name": "New Zealand dollar"
                            }, {
                                "id": "RUB",
                                "name": "Russian rouble"
                            }
                        ]
                    }
                ],
                "observation": [
                    {
                        "id": "TIME_PERIOD",
                        "name": "Time period or range",
                        "keyPosition": 6,
                        "values": [
                            {
                                "id": "2013-01-18",
                                "name": "2013-01-18",
                                "start": "2013-01-18T00:00:00.000Z",
                                "end": "2013-01-18T23:59:59.000Z"
                            }, {
                                "id": "2013-01-21",
                                "name": "2013-01-21",
                                "start": "2013-01-21T00:00:00.000Z",
                                "end": "2013-01-21T23:59:59.000Z"
                            }
                        ]
                    }
                ]},
            "attributes": {
                "dataSet": [],
                "series": [
                    {
                        "id": "TITLE",
                        "name": "Series title",
                        "values": [
                            {
                                "name": "New zealand dollar (NZD)"
                            }, {
                                "name": "Russian rouble (RUB)"
                            }
                        ]
                    }
                ],
                "observation": [
                    {
                        "id": "OBS_STATUS",
                        "name": "Observation status",
                        "values": [
                            {
                                "id": "A",
                                "name": "Normal value"
                            }
                        ]
                    }
                ]
            }
        },
        "dataSets": [
            {
                "extracted": "2013-01-21T15:20:00.000Z",
                "action": "Informational",
                "dimensions": [0, 0, 0, 0],
                "series": [
                    {
                        "dimensions": [0],
                        "attributes": [0],
                        "observations": [
                            [0, 1.5931, 0],
                            [1, 1.5925, 0]
                        ]
                    }, {
                        "dimensions": [1],
                        "attributes": [1],
                        "observations": [
                            [0, 40.3426, 0],
                            [1, 40.3000, 0]
                        ]
                    }
                ]
            }
        ]
    }

There is one data set in the message, and it contains two series.

    {
        "dimensions": [0],
        "attributes": [0],
        "observations": [
            [0, 1.5931, 0],
            [1, 1.5925, 0]
        ]
    },
    {
        "dimensions": [1],
        "attributes": [1],
        "observations": [
            [0, 40.3426, 0],
            [1, 40.3000, 0]
        ]
    }

The structure.dimensions field tells us that, out of the 6 dimensions, 4 have the same value for the 2 series and are therefore
attached to the data set level.

We see that, for the first series, we get the value 0:

    "dimensions": [0]

From the structure information, we know that CURRENCY is the series dimension.

    "seriesDimensions": [
        {
            "id": "CURRENCY",
            "name": "Currency",
            "keyPosition": 2,
            "values": [
                {
                    "id": "NZD",
                    "name": "New Zealand dollar"
                }, {
                    "id": "RUB",
                    "name": "Russian rouble"
                }
            ]
        }
    ]

The value 0 identified previously is the index of the item in the collection of values for this component. In this case,
the dimension value is therefore "New Zealand dollar".

The same logic applies when mapping attributes.
