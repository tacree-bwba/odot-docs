# BTRS Microservices Version 1.00

## Table of Contents
1. [About This Document](#about-this-document)
	* [Audience](#audience)
2.  [Environments](#environments)
3.  [API Overview](#api-overview)
	* [LRS and LRM](#lrs-and-lrm)
	* [Temporality](#temporality)
	* [LRM Type](#lrm-type)
	* [Centerline Routes](#centerline-routes)
4. [Perp Year API](#perp-year-api)
	* [Perp Year Object](#perp-year-object)
	* [Get Current Perp Year](#get-current-perp-year)
	* [Get Current Perp Year by Year](#get-current-perp-year-by-year) 
	* [Get Perp Year Bounds](#get-perp-year-bounds)
5.  [Routes API](#routes-api)
	* [Get Jurisdiction Codes](#get-jurisdiction-codes)
	* [Get Trans Route Codes](#get-trans-route-codes)
	* [Get Local Suffix Codes](#get-local-suffix-codes)
	* [Get State Suffix Codes](#get-state-suffix-codes)
	* [Get Direction Codes](#get-direction-codes)
	* [County Route Search](#county-route-search)
	* [Route Bounds Search](#route-bounds-search)
	* [Route Search](#route-search)
	* [Route Roadway Inventory Events](#route-roadway-inventory-events)
	* [Get Routes RI Attributes](#get-routes-ri-attributes)
6. [Streets API](#streets-api)
	* [Street Name Attributes](#street-name-attributes)
	* [Street Event Search](#street-event-search)
	* [Street Name Search](#street-name-search)
7. [Boundary API](#boundary-api)
	* [EndPoints](#endpoints)

## About This Document

The purpose of this document is to provide a quick overview of the REST services provided by the BTRS Services API.  One should note that all services are still being developed at this stage and so urls and request parameters are still in flux.  The desire is to give developers wishing to use the API something so that they can begin to test and develop with them.

### Audience
This document is intended for developers wishing to begin testing the BTRS Microservices API.

## Environments
Currently there is only one environment and it is considered temporary until the Docker environment is established.  For the services, the following base URL shall be used:

**http://dotibtrsd01:8080**

All endpoints provided hereafter shall be assumed that the base URL is prepended to it.  In the future, a DNS entry shall be made to direct traffic via a well known domain name to whatever server matches the environment, so that the server can be swapped without affecting calling environments.

## API Overview

The API is currently divided up into a couple different base endpoints which are covered in the subsequent sections:

- *Routes* – Finding information about a route.
- *Route Events* – Reading information about point and linear events.
- *Streets* – API for retrieving information about street events including their relationships to municipalities and townships.
- *Boundary* – Areas that represent a boundary attribute such as municipalities, counties, townships and districts.
- *Conflation* – Transforming events from one perp year to another.

### LRS and LRM

LRS stands for Linear Referencing System and is a collection of LRMs that can be related and conflated.

LRM stands for Linear Referencing Method and represents a type of linear referencing supported by RIMS.  For purposes of this document, 2 LRMs are the focus, County and State.  Further, each of the LRMs are broken down by perpYear, so each year a new version of these LRMs are published.

### Temporality

Unlike traditional BTRS based data, all data provided by the API has a temporal component to it identified by the data&#39;s Perp Year.  Perp Year identifies the year from which the data is being published, and is usually 1 year less than the current calendar year.  For example, the perp year 2018 represents the version of the roadway inventory that was published and to which all data as of 1/1/2019 is collected against.  A concrete example of this would be 2019 Pavement Condition Data (PCR) which is collected and published against the 2018 Perp Year LRS.

### LRM Type

Many of the requests have an optional lrmType parameter.  The LRM Type identifies when the request should use the State traversal of a route, or the County traversal.  If one isn&#39;t provided, **the county traversal is used by default**.  There may be cases in the future that some endpoints may also support 2-decimal versions of these end points to support legacy BTRS applications, but as of now, everything is going to be assumed that measures shall use 3-decimal accuracy.

The LRM Types are identified using one of the following 2 codes:

- CTL\_3DECIMAL – This is the county traversal using 3-decimal measure accuracy.
- STL\_3DECIMAL – This is the state traversal using 3-decimal measure accuracy.

#### LRM Type Traversal Fields

The county traversal LRM represents routes as they traverse through a county.  Routes and measures are identified using the following keys that shall be included in most event request responses:

- **perpYear –** As all data in the system includes historical information, a perpYear is needed to identify the version of the LRM being queried.  The perpYear identifies the year desired; however, it is optional, as when not specified, the current one is always used.
- **routeId** – The identifier of a route based on the lrmType.  When the route type is county, this shall be the route&#39;s nlfId e.g. SFRAIR00070\*\*C.  If the lrmType is state, the routeId has the following form: SIR00070\*\*CPRE.  In this case, the county ID is moved to the end of the identifier and represents the county within the state where the route originates.
- **nlfId** – This field identifies the county routeId.
- **nlfIdSt** – This field identifies the state system routeId.
- **beginMeasure** – The start measure of a linear event based on the lrmType.  For county measures, this is the begin measure of a route within a county.  For state, this is the state offset for the entire route traversal.
- **endMeasure** – The end measure of the linear event based on the lrmType.  For county measures, this is the end measure of a route within a county.  For state, this is the state offset for the entire route traversal.
- **measure** – The measure value for a point event.  This can be either state or county depending on the lrmType.
- **leaveInd** – Identifies that a geometric discontinuity is associated with the endMeasure.  The type of discontinuity is identified by the **leaveReenterTypeCd.  ** This field has a Boolean type.
- **reenterInd** – Identifies that a geometric discontinuity is associated with the beginMeasure.  The type of discontinuity is identified by the **leaveReenterTypeCd**. This field has a Boolean type.
- **leaveReenterTypeCd** – Identifies the type of leave/reenter when applicable.  Possible values include the following:
  * COUNTY\_LINE – Route is leaving or reentering a county.  In the case of a leave, this is only set if the route is going to eventually reenter the county.
  * PHYSICAL\_GAP – Identifies a physical discontinuity of the route.
  * LOCAL\_ROUTE – Identifies specifically a local road where the route generally is transferred to a different route, then continues on to eventually transfer back to the original.
  * ALPHA\_ROUTE – False discontinuities that are placed in the data due to ESRI limitations related to the geometry definition of a route.
  * JOURNAL\_GAP – False discontinuities that occur when the digitizing of the road jumps from the cardinal side of the road to the non-cardinal.  This is done to appease cases where a route has a cardinal overlap on the non-cardinal side of a route.
  * OTHER – Unreconciled discontinuities.

### Centerline Routes

The API is currently making a distinction between "centerline" routes and non-centerline.  Centerline routes only include cardinal directed routes, and the last character of the routeId shall be a 'C' character.

This character is called the Bi-Directional Code, and can carry the following values:

- *C – Cardinal* – indicating that measures are in the digitized direction.
- *N – Non-Cardinal* – indicating the opposite side of a route where the road is divided.  Its measures flow in the same direction as the cardinal side even though the flow of the road is in the opposite direction.

For purposes of most applications, the non-cardinal routes should be excluded from storing events as these routes don't carry attribution in the roadway inventory system.  Additionally, the measure strategy for these routes is subject to change while this portion of the system is still being defined and developed.

Where applicable in route searches, a **centerline** request attribute is defined, and when not provided, is defaulted to true to indicate that the non-cardinal routes are to be excluded.

**WARNING – Use non-cardinal routes at your own risk when storing data.  Minimal support shall be provided at this point and time, and may involve extensive correction of your data at some point and time.**

## Perp Year API
The perp year API provides details related to getting, validating and identifying a perp year.  The perp year is what relates a version of the roadway inventory to various event data collections.

### Perp Year Object
```json
{
    "year": 2017,
    "beginDt": "2017-01-01T00:00:00.000-0500",
    "endDt": "2017-12-31T00:00:00.000-0500"
}
```
#### Attributes
- year – The perp year being identified.
- beginDt – The date this perp year started.
- endDt – The end date of this perp year.  If the value is null, that implies this is the current perp year.

### Get Current Perp Year

Retrieves the current perp year.

&#160;&#160;&#160;&#160;&#160;**End Point:** GET /perpyear

### Get Current Perp Year by Year

Retrieves the current perp year.

&#160;&#160;&#160;&#160;&#160;**End Point:** GET /perpyear/:year

#### Parameters

- year – The desired year to be retrieved.

### Get Perp Year Bounds

Retrieves the range of available perp years.  The result is an array with the min and max perp year supported.  Note that prior to 2008, only state system routes are supported.

&#160;&#160;&#160;&#160;&#160;**End Point:** GET /perpyear/bounds

#### Response
```json
[
    1984,
    2018
]
```
## Routes API

The routes API provides information about finding and searching routes as well as event information about routes.

### Get Jurisdiction Codes

Retrieves the available Jurisdiction Codes component of the NlfId.  This is the first character of the NlfId.

&#160;&#160;&#160;&#160;&#160;**End Point:** GET /routes/jurisdictions

#### Response
```json
{
    "total": 6,
    "fieldErrors": {},
    "success": true,
    "errorMessages": [],
    "items": [
        {
            "id": "S",
            "code": "S",
            "active": true,
            "shortName": "S",
            "sequence": 83,
            "longName": "State"
        },
        {
            "id": "C",
            "code": "C",
            "active": true,
            "shortName": "C",
            "sequence": 67,
            "longName": "County"
        },
. . . 
     ]
}
```
### Get Trans Route Codes

Retrieves the available Trans Route Codes component of the NlfId. This starts at the 5<sup>th</sup> position of the NlfId and is 2 characters long.

&#160;&#160;&#160;&#160;&#160;**End Point:** GET /routes/route\_codes

#### Response
```json
{
    "total": 15,
    "fieldErrors": {},
    "success": true,
    "errorMessages": [],
    "items": [
        {
            "id": "IR",
            "shortName": "IR",
            "code": "IR",
            "active": true,
            "longName": "Interstate Route",
            "sequence": 10
        },
        {
            "id": "US",
            "shortName": "US",
            "code": "US",
            "active": true,
            "longName": "United State Route",
            "sequence": 20
        },

. . . 
     ]
}
```

### Get Local Suffix Codes

Retrieves the available Local Suffix Codes component of the NlfId.  This is the 12<sup>th</sup> character of the NlfId.

&#160;&#160;&#160;&#160;&#160;**End Point:** GET /routes/local\_suffix

#### Response
```json
{
    "total": 15,
    "fieldErrors": {},
    "success": true,
    "errorMessages": [],
    "items": [
        {
            "id": "*",
            "active": true,
            "code": "*",
            "longName": "Regular Route",
            "shortName": "*",
            "sequence": 42
        },
        {
            "id": "A",
            "active": true,
            "code": "A",
            "longName": "Alternate Route",
            "shortName": "A",
            "sequence": 65
        }
. . . 
     ]
}
```

### Get State Suffix Codes

Retrieves the available State Suffix Codes component of the NlfId.  This is the 13<sup>th</sup> character of the NlfId.

&#160;&#160;&#160;&#160;&#160;**End Point:** GET /routes/state\_suffix

#### Response
```json
{
    "total": 15,
    "fieldErrors": {},
    "success": true,
    "errorMessages": [],
    "items": [
        {
            "id": "*",
            "shortName": "*",
            "code": "*",
            "active": true,
            "longName": "Regular Route",
            "sequence": 42
        },
        {
            "id": "A",
            "shortName": "A",
            "code": "A",
            "active": true,
            "longName": "Alternate Route",
            "sequence": 65
        },
. . . 
     ]
}
```

### Get Direction Codes

Retrieves the available Direction Codes component of the NlfId.  This is the 14<sup>th</sup> character of the NlfId. Note that this service is provided for completeness, though currently it only carries two values: ['N', 'C'].  For purposes of BTRS, only 'C' is used as the 'N' designation indicates a non-cardinal route which is not yet ready for public consumption.

&#160;&#160;&#160;&#160;&#160;**End Point:** GET /routes/direction\_codes

#### Response
```json
{
    "total": 1,
    "items": [
        "C"
    ],
    "success": true,
    "errorMessages": [],
    "fieldErrors": {}
}
```

### County Route Search

This end point is provided to perform fast lookups of a NlfId (county route) based on search criteria being entered for example by a user in an autocomplete component.  The goal is for this end point to return results quickly so that the user doesn't experience a relevant delay in the response while typing.

&#160;&#160;&#160;&#160;&#160;**End Point:** GET /routes/county\_route\_search

#### Request Parameters

- **route\_criteria** – {String, required} – This field must contain a minimum of 3 characters; otherwise, an empty result is returned.  The value is case-insensitive and shall be parsed based on varying assumptions to best determine the user&#39;s intention for which nlfId(s) they are searching.  The goal is to provide the option for shorthanding route names.  For example, FRAIR70 is treated as shorthand for obtaining a result of &#39;SFRAIR00070\*\*C&#39;.
- **year** – {Integer, optional} – The perp year on which to search for the routes.  This field is optional, and if not provided, shall use the current active perp year.
- **max\_results** – {Integer, optional but recommended} – This parameter indicates the maximum number of results to return from the query.  While it is optional, it is highly recommended to reduce the cost of serializing large payloads down to the user and keep the interface responsive.  A good recommended value for this would be 100 or less.  It is rare that a user would scroll through these results before providing another character that would further narrow the search.
- **centerline** – {Boolean, optional, default=true} – This field indicates that only Cardinal routes are provided in the result.  It is optional as the default value is set to true.  Non-cardinal routes should only be included in the result with review by the Office of Technical Services to make sure that non-cardinal routes are sustainable for the user&#39;s usage.
- **counties** – {List<String&#62;, optional} – An optional list of counties which indicates that they are the only counties included in the search.

#### Response

**Example Request:**

GET /routes/county\_route\_search?route\_criteria=fra70&amp;max\_results=100
```json
{
    "total": 31,
    "fieldErrors": {},
    "success": true,
    "items": [
        "CFRACR00070**C",
        "CFRACR00700**C",
        "CFRACR00701**C",
        "CFRACR00702**C",
        "CFRACR00703**C",
        "CFRACR00704**C",
        "CFRACR00705**C",
        ...
        "CFRACR07008**C",
        "MFRAMR00070**C",
        "MFRAMR00700**C",
        "MFRAMR00701**C",
        "MFRAMR00702**C",
        "MFRAMR00703**C",
        "MFRAMR00705**C",
        "MFRAMR00706**C",
        "MFRAMR00707**C",
        "MFRAMR00708**C",
        "MFRAMR00709**C",
        "MFRAMR00709A*C",
        "SFRAIR00070**C"
    ],
    "errorMessages": []
}
```
### Route Bounds Search

This end point retrieves route measure bounds information based on a provided route id.  The request supports both the County Route and State Route Lrm Types.

&#160;&#160;&#160;&#160;&#160;**End Point:** GET /routes/bounds/:routeId

#### Path Variables

- **routeId –** {String, required} – This identifies a routeId from which to request boundary details.  LrmTypes of both CTL\_3DECIMAL and STL\_3DECIMAL are supported.

#### Request Parameters

- **year** – {Integer, optional} – The perp year on which to search for the route.  This field is optional, and if not provided, shall use the current active perp year.
- **lrmType** – {String, optional} – This accepts an enumerated type of the following values: [CTL\_3DECIMAL, STL\_3DECIMAL].  If not provided, it defaults to CTL\_3DECIMAL

#### Response

The response returns the routeId for the LrmType.  The nlfIdSt is always included in the response.  If the lrmType is STL\_3DECIMAL, then the nlfId is NOT included.  If the lrmType is CTL\_3DECIMAL, then the nlfId is included along with the corresponding measures for the county LRM.

***5.7.3.1***

- nlfId – The routeId for the county LRM.
- nlfIdSt – The routeId for the state LRM.
- ctlBeginNbr – The county LRM begin boundary.
- ctlEndNbr – The county LRM end boundary.
- stlBeginNbr – The state LRM begin boundary.  If the LRM type is for the county route, then the boundaries are matched to those of the county measures.
- stlEndNbr – The state LRM end boundary. If the LRM type is for the county route, then the boundaries are matched to those of the county measures.

**Example Request:** Get a county route boundary
GET /routes/bounds/SFRAIR00070\*\*C?year=2018&amp;lrmType=CTL\_3DECIMAL

```json
{
    "nlfId": "SFRAIR00070**C",
    "nlfIdSt": "SIR00070**CPRE",
    "ctlBeginNbr": 0.000,
    "ctlEndNbr": 24.940,
    "stlBeginNbr": 86.043,
    "stlEndNbr": 110.983
}
```
In this example, the state measure 86.043 corresponds to the county measure 0.000.  They likewise match for the end measures.

**Example Request:** Get a state route boundary

GET /routes/bounds/SIR00070\*\*CPRE?year=2018&amp;lrmType=STL\_3DECIMAL
```json
{
    "nlfIdSt": "SIR00070**CPRE",
    "stlBeginNbr": 0.000,
    "stlEndNbr": 225.587
}
```
Note that in this example, only attributes related to the state measures are provided, and the full boundaries of the route are provided

### Route Search

This end point retrieves routes on both LrmTypes along with optional min/max boundaries.  This is a post request where the user passes a json object that defines the search configuration.  The choices made determine the attributes in the returned response.

&#160;&#160;&#160;&#160;&#160;**End Point:** POST /routes/search/

#### Request Body
```json
{
    "perpYear": 2018,
    "includeBoundaries": true,
    "stateLrm": true,
    "countyLrm": true,
	  "jurisdictions": [],
	  "counties": ["FRA"],
    "routeCodes" : ["IR"],
    "routeNbrs": ["70"],
    "extensionCodes": ["*"],
    "descriptionCodes": ["*"],
    "directionalCodes": ["N", "C"]
}
```
***Attribute Description***

- **perpYear** – {Integer, optional} – The perp year on which to search for the route.  This field is optional, and if not provided, shall use the current active perp year.
- **includeBoundaries** – {Boolean, optional, default: true} – This indicates that route boundaries for the selected LRMs should be included in the result
- **stateLrm** – {Boolean, default: false} – This indicates that the state LRM attributes should be included in the query and result, in this case meaning the nlfIdSt, stlBeginNbr, and stlEndNbr
- **countyLrm** – {Boolean, default: true} – This indicates that the county LRM attributes should be included in the query and result, in this case meaning the nlfId, ctlBeginNbr, and ctlEndNbr
- **jurisdictions** – {List<String&#62;, optional} – List of route jurisdiction codes on which to filter.
- **counties** – {List<String&#62;, optional} – List of county codes on which to filter.
- **routeCodes** – {List\&lt;String\&gt;, optional} – List of route trans route codes on which to filter.
- **routeNbrs** – {List\&lt;String\&gt;, optional} – List of route numbers on which to filter.
- **extensionCodes** – {List\&lt;String\&gt;, optional} – List of route extension codes on which to filter.
- **descriptionCodes** – {List\&lt;String\&gt;, optional} – List of route description codes on which to filter.
- **directionalCodes** – {List\&lt;String\&gt;, optional} – List of route direction codes on which to filter. By default, all queries are filtered with the [&quot;C&quot;] unless the non-cardinal direction is specified in the request.

#### Response

The response object minimally includes a list of objects that can contain only a routeId unless includeBoundaries is set to true.  In this case, it will include either state, county or both LRMs depending on the stateLrm/countyLrm request settings.

***Response Object***

- nlfId – The routeId for the county LRM.
- nlfIdSt – The routeId for the state LRM.
- ctlBeginNbr – The county LRM begin boundary.
- ctlEndNbr – The county LRM end boundary.
- stlBeginNbr – The state LRM begin boundary.  If the LRM type is for the county route, then the boundaries are matched to those of the county measures.
- stlEndNbr – The state LRM end boundary. If the LRM type is for the county route, then the boundaries are matched to those of the county measures.

#### Example Request:

POST /routes/ri/events

**Request Body**
```json
{
    "perpYear": 2018,
    "includeBoundaries": true,
    "stateLrm": true,
    "countyLrm": true,
	  "jurisdictions": [],
	  "counties": ["FRA"],
    "routeCodes" : ["IR"],
    "routeNbrs": ["70"],
    "extensionCodes": ["*"],
    "descriptionCodes": ["*"],
    "directionalCodes": ["N", "C"]
}
```
**Response**
```json
{
    "total": 2,
    "success": true,
    "fieldErrors": {},
    "errorMessages": [],
    "items": [
        {
            "nlfId": "SFRAIR00070**C",
            "nlfIdSt": "SIR00070**CPRE",
            "ctlBeginNbr": 0.000,
            "ctlEndNbr": 24.940,
            "stlBeginNbr": 86.043,
            "stlEndNbr": 110.983
        },
        {
            "nlfId": "SFRAIR00070**N",
            "nlfIdSt": "SIR00070**NFRA",
            "ctlBeginNbr": 0.000,
            "ctlEndNbr": 24.940,
            "stlBeginNbr": 0.000,
            "stlEndNbr": 24.940
        }
    ]
}
```
### Route Roadway Inventory Events

This end point retrieves route linear and point events based on the configuration of the request body object.

&#160;&#160;&#160;&#160;&#160;**End Point:** POST /routes/ri/events/

#### Request Body
```json
{
  "lrmType": "CTL_3DECIMAL",
  "allAttributes": true,
  "segments": [
    {
      "id": "a",
      "routeId": "SFRASR00003**C"
    },
    {
      "perpYear": 2018,
      "routeId": "SFRAIR00071**C",
      "beginMeasure": 10,
      "endMeasure": 18
    }
  ]
}
```
***Attribute Description***

- **lrmType** – {String, optional} – This accepts an enumerated type of the following values: [CTL\_3DECIMAL, STL\_3DECIMAL].  If not provided, it defaults to CTL\_3DECIMAL
- **allAttributes** – {Boolean, optional, default: false} – This indicates that all available attributes related to an event from the Roadway Inventory should be included in the response.
- **attributes** – {List<String&#62;, optional} – This is an alternative to the allAttributes option, where the caller may specify which attributes are to be included in the result. Reducing the number of attributes usually reduces the number of segments that will be returned and lightens the response payload.
- **continuous** – {Boolean, default: true} – Relevant when the County Route LRM Type has been selected.  This indicator implies that when routes that leave the county (County leave/reenters) and return are part of the result, that the segments related to the sections that left the county are included in the result.
- **segments** – {List<Object&#62;, optional} – The list of segments to return as part of the result.
  - **id** – {String, optional} – An id that is added to the response to help track this requested segment.  The id becomes an attribute in the response containing the array of segments that were retrieved and associated with this request.  If an id is NOT provided, then the index of the item from the request is used.  Note that ids can be numbers or strings.
  - **perpYear** – {Integer, optional} – The perp year on which to search for the event.  This field is optional, and if not provided, shall use the current active perp year.
  - **routeId** – {String, required} – The route identifier for the event to be retrieved.  The LrmType shall dictate whether this is a state route id or a county route id.
  - **beginMeasure** – {Decimal, optional, defaults to beginning of route} – The begin measure of the events to retrieve.
  - **endMeasure** – {Decimal, optional, defaults to end of route} – The end measure of the events to retrieve.

#### Response

The response object minimally includes a list of objects that can contain only a routeId unless includeBoundaries is set to true.  In this case, it will include either state, county or both LRMs depending on the stateLrm/countyLrm request settings.

***Response Object***

The response object is a json map object where the key is the original id of the requested data, and the value is an array of the linear events that satisfy the request from the posted data.

&#160;&#160;&#160;&#160;*Event Object Common Fields*

- Id – A unique id assigned to this event.
- perpYear – The perpYear to which the data matches.
- nlfId – The routeId for the county LRM.
- nlfIdSt – The routeId for the state LRM.
- beginMeasure – The beginMeasure for this event based on the lrmType.
- endMeasure – The endMeasure for this event based on the lrmType.
- primary – Boolean value indicating the section is primary.
- primarySegment.nlfId – Provided when the current segment is secondary.  This is the nlfId of the primary county route over this section.
- primarySegment.beginMeasure – The primary route&#39;s beginMeasure correlating to the segment&#39;s beginMeasure.
- primarySegment.endMeasure – The primary route&#39;s endMeasure correlating to the segment&#39;s endMeasure.
- leave – Boolean value indicating that the end point of this event is adjacent to a discontinuity.
- reenter – Boolean value indicating that the begin point of this event is adjacent to a discontinuity.
- overlapInverseInd – Boolean value that indicates that the secondary event is digitized in the opposite direction of its corresponding primary.
- stl3dBeginNbr – The state LRM begin boundary. This is included when the county LRM is requested for reference.
- stl3dEndNbr – The state LRM end boundary. This is included when the county LRM is requested for reference.

*Event Object Optional Attributes*

In addition to the common fields, the requester can pick and choose other supported fields within the roadway inventory data.  To view the complete list of available fields, see the **Routes RI Attributes** end point.

#### Example Request:

POST /routes/ri/events

**Request Body**
```json
{
  "lrmType": "CTL_3DECIMAL",
  "attributes": ["lanes", "leftMuniName", "leftMuniPopulation", "rightMuniName", "rightMuniPopulation" ],

  "segments": [
  	
    {
      "perpYear": 2018,
      "routeId": "SFRAIR00071**C",
      "beginMeasure": 15.30,
      "endMeasure": 16.00
    },
    {
      "id": "A",
      "perpYear": 2017,
      "routeId": "SFRAUS00023**C",
      "beginMeasure": 0.6,
      "endMeasure": 2.1
    }
  ]
}
```
**Response**
```json
{
    "0": [
        {
            "primarySegment.beginMeasure": 13.250,
            "nlfId": "SFRAIR00071**C",
            "perpYear": 2018,
            "stl3dBeginNbr": 106.366,
            "rightMuniPopulation": 787033,
            "reenter": false,
            "rightMuniName": "COLUMBUS",
            "leftMuniName": "COLUMBUS",
            "beginMeasure": 15.300,
            "primarySegment.nlfId": "SFRAIR00070**C",
            "leave": false,
            "overlapInverseInd": false,
            "stl3dEndNbr": 107.066,
            "lanes": 6,
            "primarySegment.endMeasure": 13.950,
            "segmentLength": 0.700,
            "id": "8af3f9b7705eeb8c017063461ccc010a",
            "nlfIdSt": "SIR00071**CHAM",
            "leftMuniPopulation": 787033,
            "endMeasure": 16.000,
            "primary": false
        }
    ],
    "A": [
        {
            "nlfId": "SFRAUS00023**C",
            "perpYear": 2017,
            "rightMuniPopulation": 787033,
            "reenter": false,
            "rightMuniName": "COLUMBUS",
            "leftMuniName": "COLUMBUS",
            "beginMeasure": 0.600,
            "leave": false,
            "overlapInverseInd": false,
            "lanes": 4,
            "segmentLength": 0.900,
            "id": "8af3f9b7705eeb8c017063461d03010b",
            "nlfIdSt": "SUS00023**CSCI",
            "leftMuniPopulation": 787033,
            "endMeasure": 1.500,
            "primary": true
        },
        {
            "nlfId": "SFRAUS00023**C",
            "perpYear": 2017,
            "beginMeasure": 1.500,
            ...
        }
    ]
}
```
### Get Routes RI Attributes

Retrieves the available attributes/metadata for the Roadway Inventory (RI) dataset.  The attributes are available for use with the **Route Roadway Inventory Events** end point.

&#160;&#160;&#160;&#160;&#160;**End Point:** GET /routes/ri/attributes

#### Response
```json
{
    "attributes": [
        "accessControl",
        "laneWidth",
        "lanes",
        "leftBaseTypeCd",
        "leftCounty",
        "leftInShoulderWidth",
        "leftMuniName",
        "leftTownshipName",
        "medianType",
        "medianWidth",
        "mileageClass",
        "nhsCd",
        "roadwayClass",
        "roadwayWidth",
      ...
    ]
}
```
## Streets API

The streets API provides information specifically relating street names to routes.  This linear event dataset does not fully cover all routes in the roadway inventory, but only the ones that actually have a street name.

The data also relates the street name to the governing body of the route, specifically the county, township and municipality.  There are cases where a street can be in 2 different municipalities (jurisdictional split) and/or a split between townships and municipalities.  In these cases there will be a record for each governing body.

### Street Name Attributes

Retrieves the available attributes/metadata for the Street Event dataset.  The attributes are available for use with the **Street Search** end point.

&#160;&#160;&#160;&#160;&#160;**End Point:** GET /streets/attributes

#### Response
```json
{
    "attributes": [
        "countyCode",
        "ctl3dBeginNbr",
        "ctl3dEndNbr",
        "id",
        "leave",
        "leaveReenterTypeCode",
        "muniFipsCode",
        "muniName",
        "nlfId",
        "nlfIdSt",
        "perpYear",
        "primary",
        "reenter",
        "segmentLength3D",
        "stl3dBeginNbr",
        "stl3dEndNbr",
        "streetDirectionPrefix",
        "streetDirectionSuffix",
        "streetName",
        "streetSuffix",
        "townshipFipsCode",
        "townshipName"
    ]
}
```
### Street Event Search

This end point retrieves street name events based on the posted search criteria

&#160;&#160;&#160;&#160;&#160;**End Point:** POST /streets/search

#### Request Body
```json
{
    "perpYear": 2018,
    "lrmType": "CTL_3DECIMAL",
    "attributes": [ "countyCode",  "muniName", "muniFipsCode", "streetName"],

    "routeCodes" : ["US"],
    "routeNbrs": ["23"],
    "streetNames": ["INDIANOLA", "HIGH"],
    "muniFipsCodes": ["18000"]
   } 
```
***Attribute Description***

- **perpYear** – {Integer, optional} – The perp year on which to search for the route.  This field is optional, and if not provided, shall use the current active perp year.
- **lrmType** {String, optional} – This accepts an enumerated type of the following values: [CTL\_3DECIMAL, STL\_3DECIMAL].  If not provided, it defaults to CTL\_3DECIMAL
- **allAttributes** – {Boolean, optional, default: false} – This indicates that all available attributes related to an event from the Street Names should be included in the response.
- **attributes** – {List<String&#62;, optional} – This is an alternative to the allAttributes option, where the caller may specify which attributes are to be included in the result. Reducing the number of attributes usually reduces the number of segments that will be returned and lightens the response payload.
- **jurisdictions** – {List<String&#62;, optional} – List of route jurisdiction codes on which to filter.
- **counties** – {List<String&#62;, optional} – List of county codes on which to filter.
- **routeCodes** – {List<String&#62;, optional} – List of route trans route codes on which to filter.
- **routeNbrs** – {List<String&#62;, optional} – List of route numbers on which to filter.
- **extensionCodes** – {List<String&#62;, optional} – List of route extension codes on which to filter.
- **descriptionCodes** – {List<String&#62;, optional} List of route description codes on which to filter.
- **directionalCodes** – {List<String&#62;, optional} – List of route direction codes on which to filter. By default, all queries are filtered with the [&quot;C&quot;] unless the non-cardinal direction is specified in the request.
- **muniFipsCodes** – {List<String&#62;, optional} – List of municipal FIPS codes on which to filter.
- **townshipFipsCodes** – {List<String&#62;, optional} – List of township FIPS codes on which to filter.
- **streetNames** – {List<String&#62;, optional} – List of street names on which to filter.

#### Response

Based on the example request body shown above.
```json
{
  "total": 7,
  "success": true,
  "fieldErrors": {},
  "errorMessages": [],
  "items": [
    {
      "nlfId": "SFRAUS00023**C",
      "perpYear": 2018,
      "stl3dBeginNbr": 83.730,
      "reenter": false,
      "muniFipsCode": "18000",
      "countyCode": "FRA",
      "streetName": "HIGH",
      "muniName": "COLUMBUS",
      "beginMeasure": 5.410,
      "leave": false,
      "stl3dEndNbr": 88.830,
      "segmentLength": 5.100,
      "nlfIdSt": "SUS00023**CSCI",
      "endMeasure": 10.510
    },
    {
      "nlfId": "SFRAUS00023**C",
      "perpYear": 2018,
      "stl3dBeginNbr": 93.630,
      "reenter": false,
      "muniFipsCode": "18000",
      "countyCode": "FRA",
      "streetName": "INDIANOLA",
      "muniName": "COLUMBUS",
      "beginMeasure": 15.310,
      "leave": false,
      "stl3dEndNbr": 96.850,
      "segmentLength": 3.220,
      "nlfIdSt": "SUS00023**CSCI",
      "endMeasure": 18.530
    },
    ...
  ]
}        
```
### Street Name Search

Retrieves a unique list of street names based on the provided criteria provided in the request body.

&#160;&#160;&#160;&#160;&#160;**End Point:** GET /streets/names

#### Request Body

The request body structure is identical to **Street Event Search** end point, with the additions/changes of the following attributes:

- **maxResults** – {number, optional} – The maximum number of results to return.  This field is highly recommended for broad searches, especially where this might be used in an autocomplete interface scenario.
- **lrmType** – This field is ignored as it doesn't impact the result.

**Example Request Body**

```json
    "perpYear": 2018,
    "counties": ["FRA"],
    "routeCodes" : ["sr", "cr"],
    "muniFipsCodes" : ["06278"],
    "maxResults": 3
}
```
#### Response

Based on the request example above, the following is an example response.
```json
{
    "total": 3,
    "success": true,
    "fieldErrors": {},
    "errorMessages": [],
    "items": [
        "BROAD",
        "CASSADY",
        "COLLEGE"
    ]
}
```
## Boundary API

This API provides end points for retrieving boundary datasets such as County, Municipality, and Townships.

### EndPoints

- *GET /boundaries/counties/* – Retrieves county information along with name and district
- *GET /boundaries/counties/{districtNbr}* – Retrieves county information along with name and district for a specific district.
- *GET /boundaries/counties/ids* – Retrieves just the list of county 3-character id codes.
- *GET /boundaries/municipalities }* – Retrieve all municipality records for the current perp year.
- *GET /boundaries/municipalities?county={countyCode}* – Retrieve the municipality list by county code for the current perp year.
- *GET /boundaries/townships }* – Retrieve all township records for the current perp year.
- *GET /boundaries/townships?county={countyCode}* – Retrieve the township list by county code for the current perp year.
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTYzNzA1MzE5XX0=
-->