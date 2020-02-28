---


---

<h1 id="btrs-microservices-version-1.00">BTRS Microservices Version 1.00</h1>
<h2 id="table-of-contents">Table of Contents</h2>
<ol>
<li><a href="#about-this-document">About This Document</a>
<ul>
<li><a href="#audience">Audience</a></li>
</ul>
</li>
<li><a href="#environments">Environments</a></li>
<li><a href="#api-overview">API Overview</a>
<ul>
<li>LRS and LRM</li>
<li>Temporality</li>
<li>LRM Type</li>
<li>Centerline Routes</li>
</ul>
</li>
<li><a href="#perp-year-api">Perp Year API</a>
<ul>
<li>Perp Year Object<br>
4.2 Get Current Perp Year<br>
4.3 Get Current Perp Year by Year<br>
4.4 Get Perp Year Bounds</li>
</ul>
</li>
<li>Routes API<br>
5.1 Get Jurisdiction Codes<br>
5.2 Get Trans Route Codes<br>
5.3 Get Local Suffix Codes<br>
5.4 Get State Suffix Codes<br>
5.5 Get Direction Codes<br>
5.6 County Route Search<br>
5.7 Route Bounds Search<br>
5.8 Route Search<br>
5.9 Route Roadway Inventory Events<br>
5.10 Get Routes RI Attributes</li>
</ol>
<h4 id="streets-api">Streets API</h4>
<p>6.1 Street Name Attributes<br>
6.2 Street Event Search<br>
6.3 Street Name Search</p>
<h4 id="boundary-api">Boundary API</h4>
<p>7.1 EndPoints</p>
<h2 id="about-this-document">About This Document</h2>
<p>The purpose of this document is to provide a quick overview of the REST services provided by the BTRS Services API.  One should note that all services are still being developed at this stage and so urls and request parameters are still in flux.  The desire is to give developers wishing to use the API something so that they can begin to test and develop with them.</p>
<h3 id="audience">Audience</h3>
<p>This document is intended for developers wishing to begin testing the BTRS Microservices API.</p>
<h2 id="environments">Environments</h2>
<p>Currently there is only one environment and it is considered temporary until the Docker environment is established.  For the services, the following base URL shall be used:</p>
<p><strong><a href="http://dotibtrsd01:8080">http://dotibtrsd01:8080</a></strong></p>
<p>All endpoints provided hereafter shall be assumed that the base URL is prepended to it.  In the future, a DNS entry shall be made to direct traffic via a well known domain name to whatever server matches the environment, so that the server can be swapped without affecting calling environments.</p>
<h2 id="api-overview">API Overview</h2>
<p>The API is currently divided up into a couple different base endpoints which are covered in the subsequent sections:</p>
<ul>
<li>Routes – Finding information about a route.</li>
<li>Route Events – Reading information about point and linear events.</li>
<li>Streets – API for retrieving information about street events including their relationships to municipalities and townships.</li>
<li>Boundary – areas that represent a boundary attribute such as municipalities, counties, townships and districts.</li>
<li>Conflation – transforming events from one perp year to another</li>
</ul>
<ol>
<li></li>
<li>3.1LRS and LRM</li>
</ol>
<p>LRS stands for Linear Referencing System and is a collection of LRMs that can be related and conflated.</p>
<p>LRM stands for Linear Referencing Method and represents a type of linear referencing supported by RIMS.  For purposes of this document, 2 LRMs are the focus, County and State.  Further, each of the LRMs are broken down by perpYear, so each year a new version of these LRMs are published.</p>
<ol>
<li></li>
<li>3.2Temporality</li>
</ol>
<p>Unlike traditional BTRS based data, all data provided by the API has a temporal component to it identified by the data's Perp Year.  Perp Year identifies the year from which the data is being published, and is usually 1 year less than the current calendar year.  For example, the perp year 2018 represents the version of the roadway inventory that was published and to which all data as of 1/12019 is collected against.  A concrete example of this would be 2019 Pavement Condition Data (PCR) which is collected and published against the 2018 perp year LRS.</p>
<ol>
<li></li>
<li>3.3LRM Type</li>
</ol>
<p>Many of the requests have an optional lrmType parameter.  The LRM Type identifies when the request should use the State traversal of a route, or the County traversal.  If one isn't provided, <strong>the county traversal is used by default</strong>.  There may be cases in the future that some endpoints may also support 2-decimal versions of these end points to support legacy BTRS applications, but as of now, everything is going to be assumed that measures shall use 3-decimal accuracy.</p>
<p>The LRM Types are identified using one of the following 2 codes:</p>
<ul>
<li>CTL_3DECIMAL – this is the county traversal using 3-decimal measure accuracy.</li>
<li>STL_3DECIMAL – This is the state traversal using 3-decimal measure accuracy.</li>
</ul>
<ol>
<li></li>
<li></li>
<li>3.3.1LRM Type Traversal Fields</li>
</ol>
<p>The county traversal LRM represents routes as they traverse through a county.  Routes and measures are identified using the following keys that shall be included in most event request responses:</p>
<ul>
<li><strong>perpYear –</strong> As all data in the system includes historical information, a perpYear is needed to identify the version of the LRM being queried.  The perpYear identifies the year desired; however, it is optional, as when not specified, the current one is always used.</li>
<li><strong>routeId</strong> – The identifier of a route based on the lrmType.  When the route type is county, this shall be the route's nlfId e.g. SFRAIR00070**C.  If the lrmType is state, the routeId has the following form: SIR00070**CPRE.  In this case, the county ID is moved to the end of the identifier and represents the county within the state where the route originates.</li>
<li><strong>nlfId</strong> – This field identifies the county routeId.</li>
<li><strong>nlfIdSt</strong> – This field identifies the state system routeId.</li>
<li><strong>beginMeasure</strong> – The start measure of a linear event based on the lrmType.  For county measures, this is the begin measure of a route within a county.  For state, this is the state offset for the entire route traversal.</li>
<li><strong>endMeasure</strong> = The end measure of the linear event based on the lrmType.  For county measures, this is the end measure of a route within a county.  For state, this is the state offset for the entire route traversal.</li>
<li><strong>measure</strong> – The measure value for a point event.  This can be either state or county depending on the lrmType</li>
<li><strong>leaveInd</strong> – Identifies that a geometric discontinuity is associated with the endMeasure.  The type of discontinuity is identified by the **leaveReenterTypeCd.  ** This field has a Boolean type.</li>
<li><strong>reenterInd –</strong> Identifies that a geometric discontinuity is associated with the beginMeasure.  The type of discontinuity is identified by the **leaveReenterTypeCd.  ** This field has a Boolean type.</li>
<li><strong>leaveReenterTypeCd –</strong> Identifies the type of leave/reenter when applicable.  Possible values include the following:
<ul>
<li>COUNTY_LINE – Route is leaving or reentering a county.  In the case of a leave, this is only set if the route is going to eventually reenter the county.</li>
<li>PHYSICAL_GAP – Identifies a physical discontinuity of the route.</li>
<li>LOCAL_ROUTE – Identifies specifically a local road where the route generally is transferred to a different route, then continues on to eventually transfer back to the original.</li>
<li>ALPHA_ROUTE – False discontinuities that are placed in the data due to ESRI limitations related to the geometry definition of a route.</li>
<li>JOURNAL_GAP – False discontinuities that occur when the digitizing of the road jumps from the cardinal side of the road to the non-cardinal.  This is done to appease cases where a route has a cardinal overlap on the non-cardinal side of a route.</li>
<li>OTHER – Unreconciled discontinuities.</li>
</ul>
</li>
</ul>
<ol>
<li></li>
<li>3.4Centerline Routes</li>
</ol>
<p>The API is currently making a distinction between "centerline" routes and non-centerline.  Centerline routes only include cardinal directed routes, and the last character of the routeId shall be a 'C' character.</p>
<p>This character is called the Bi-Directional Code, and can carry the following values:</p>
<ul>
<li>C – Cardinal, indicating that measures are in the digitized direction.</li>
<li>N – Non-Cardinal – indicating the opposite side of a route where the road is divided.  Its measures flow in the same direction as the cardinal side even though the flow of the road is in the opposite direction.</li>
</ul>
<p>For purposes of most applications, the non-cardinal routes should be excluded from storing events as these routes don't carry attribution in the roadway inventory system.  Additionally, the measure strategy for these routes is subject to change while this portion of the system is still being defined and developed.</p>
<p>Where applicable in route searches, a <strong>centerline</strong> request attribute is defined, and when not provided, is defaulted to true to indicate that the non-cardinal routes are to be excluded.</p>
<p><strong>WARNING – Use non-cardinal routes at your own risk when storing data.  Minimal support shall be provided at this point and time, and may involve extensive correction of your data at some point and time.</strong></p>
<h2 id="perp-year-api">Perp Year API</h2>
<p>The perp year API provides details related to getting, validating and identifying a perp year.  The perp year is what relates a version of the roadway inventory to various event data collections.</p>
<h3 id="perp-year-object">Perp Year Object</h3>
<pre class=" language-json"><code class="prism  language-json"><span class="token punctuation">{</span>
    <span class="token string">"year"</span><span class="token punctuation">:</span> <span class="token number">2017</span><span class="token punctuation">,</span>
    <span class="token string">"beginDt"</span><span class="token punctuation">:</span> <span class="token string">"2017-01-01T00:00:00.000-0500"</span><span class="token punctuation">,</span>
    <span class="token string">"endDt"</span><span class="token punctuation">:</span> <span class="token string">"2017-12-31T00:00:00.000-0500"</span>
<span class="token punctuation">}</span>
</code></pre>
<h4 id="attributes">Attributes</h4>
<ul>
<li>year – The perp Year being idenfied</li>
<li>beginDt – The date this perp year started</li>
<li>endDt – The end date of this perp year.  If the value is null, that implies this is the current perp year.</li>
</ul>
<ol>
<li></li>
<li>4.2Get Current Perp Year</li>
</ol>
<p>Retrieves the current perp year.</p>
<p><strong>End Point:</strong> GET /perpyear</p>
<ol>
<li></li>
<li>4.3Get Current Perp Year by Year</li>
</ol>
<p>Retrieves the current perp year.</p>
<p><strong>End Point:</strong> GET /perpyear/:year</p>
<ol>
<li></li>
<li></li>
<li>4.3.1Parameters</li>
</ol>
<ul>
<li>year – The desired year to be retrieved</li>
</ul>
<ol>
<li></li>
<li>4.4Get Perp Year Bounds</li>
</ol>
<p>Retrieves the range of available perp years.  The result is an array with the min and max perp year supported.  Note that prior to 2008, only state system routes are supported.</p>
<p><strong>End Point:</strong> GET /perpyear/bounds</p>
<ol>
<li></li>
<li></li>
<li>
<p>4.4.1Response</p>
</li>
<li>
<p>5Routes API</p>
</li>
</ol>
<p>The routes API provides information about finding and searching routes as well as event information about routes.</p>
<ol>
<li></li>
<li>5.1Get Jurisdiction Codes</li>
</ol>
<p>Retrieves the available Jurisdiction Codes component of the NlfId.  This is the first character of the NlfId</p>
<p><strong>End Point:</strong> GET /routes/jurisdictions</p>
<ol>
<li></li>
<li></li>
<li>
<p>5.1.1Response</p>
</li>
<li></li>
<li>
<p>5.2Get Trans Route Codes</p>
</li>
</ol>
<p>Retrieves the available Trans Route Codes component of the NlfId. This starts at the 5</p>
<h1 id="th">th</h1>
<p>position of the NlfId and is 2 characters long.</p>
<p><strong>End Point:</strong> GET /routes/route_codes</p>
<ol>
<li></li>
<li></li>
<li>
<p>5.2.1Response</p>
</li>
<li></li>
<li>
<p>5.3Get Local Suffix Codes</p>
</li>
</ol>
<p>Retrieves the available Local Suffix Codes component of the NlfId.  This is the 12</p>
<h1 id="th-1">th</h1>
<p>character of the NlfId.</p>
<p><strong>End Point:</strong> GET /routes/local_suffix</p>
<ol>
<li></li>
<li></li>
<li>
<p>5.3.1Response</p>
</li>
<li></li>
<li>
<p>5.4Get State Suffix Codes</p>
</li>
</ol>
<p>Retrieves the available State Suffix Codes component of the NlfId.  This is the 13</p>
<h1 id="th-2">th</h1>
<p>character of the NlfId.</p>
<p><strong>End Point:</strong> GET /routes/state_suffix</p>
<ol>
<li></li>
<li></li>
<li>
<p>5.4.1Response</p>
</li>
<li></li>
<li>
<p>5.5Get Direction Codes</p>
</li>
</ol>
<p>Retrieves the available Direction Codes component of the NlfId.  This is the 14</p>
<h1 id="th-3">th</h1>
<p>character of the NlfId. Note that this service is provided for completeness, though currently it only carries two values: ['N', 'C'].  For purposes of BTRS, only 'C' is used as the 'N' designation indicates a non-cardinal route which is not yet ready for public consumption.</p>
<p><strong>End Point:</strong> GET /routes/direction_codes</p>
<ol>
<li></li>
<li></li>
<li>
<p>5.5.1Response</p>
</li>
<li></li>
<li>
<p>5.6County Route Search</p>
</li>
</ol>
<p>This end point is provided to perform fast lookups of a NlfId (county route) based on search criteria being entered for example by a user in an autocomplete component.  The goal is for this end point to return results quickly so that the user doesn't experience a relevant delay in the response while typing.</p>
<p><strong>End Point:</strong> GET /routes/county_route_search</p>
<ol>
<li></li>
<li></li>
<li>5.6.1Request Parameters</li>
</ol>
<ul>
<li><strong>route_criteria</strong> – {String, required} This field must contain a minimum of 3 characters; otherwise, an empty result is returned.  The value is case-insensitive and shall be parsed based on varying assumptions to best determine the user's intention for which nlfId(s) they are searching.  The goal is to provide the option for shorthanding route names.  For example, FRAIR70 is treated as shorthand for obtaining a result of 'SFRAIR00070**C'.</li>
<li><strong>year –</strong> {Integer, optional} The perp year on which to search for the routes.  This field is optional, and if not provided, shall use the current active perp year.</li>
<li><strong>max_results –</strong> {Integer, optional but recommended} This parameter indicates the maximum number of results to return from the query.  While it is optional, it is highly recommended to reduce the cost of serializing large payloads down to the user and keep the interface responsive.  A good recommended value for this would be 100 or less.  It is rare that a user would scroll through these results before providing another character that would further narrow the search.</li>
<li><strong>centerline</strong> – {Boolean, optional, default=true} – This field indicates that only Cardinal routes are provided in the result.  It is optional as the default value is set to true.  Non-cardinal routes should only be included in the result with review by the Office of Technical Services to make sure that non-cardinal routes are sustainable for the user's usage.</li>
<li><strong>counties</strong> – {List&amp;lt;String&amp;gt;, optional} an optional list of counties which indicates that they are the only counties included in the search.</li>
</ul>
<ol>
<li></li>
<li></li>
<li>5.6.2Response</li>
</ol>
<p><strong>Example Request:</strong></p>
<p>GET /routes/county_route_search?route_criteria=fra70&amp;max_results=100</p>
<ol>
<li></li>
<li>5.7Route Bounds Search</li>
</ol>
<p>This end point retrieves route measure bounds information based on a provided route id.  The request supports both the County Route and State Route Lrm Types.</p>
<p><strong>End Point:</strong> GET /routes/bounds/:routeId</p>
<ol>
<li></li>
<li></li>
<li>5.7.1Path Variables</li>
</ol>
<ul>
<li><strong>routeId –</strong> {String, required} This identifies a routeId from which to request boundary details.  LrmTypes of both CTL_3DECIMAL and STL_3DECIMAL are supported.</li>
</ul>
<ol>
<li></li>
<li></li>
<li>5.7.2Request Parameters</li>
</ol>
<ul>
<li><strong>year –</strong> {Integer, optional} The perp year on which to search for the route.  This field is optional, and if not provided, shall use the current active perp year.</li>
<li><strong>lrmType–</strong> {String, optional} This accepts an enumerated type of the following values: [CTL_3DECIMAL, STL_3DECIMAL].  If not provided, it defaults to CTL_3DECIMAL</li>
</ul>
<ol>
<li></li>
<li></li>
<li>5.7.3Response</li>
</ol>
<p>The response returns the routeId for the LrmType.  The nlfIdSt is always included in the response.  If the lrmType is STL_3DECIMAL, then the nlfId is NOT included.  If the lrmType is CTL_3DECIMAL, then the nlfId is included along with the corresponding measures for the county LRM.</p>
<ol>
<li></li>
<li></li>
<li></li>
<li>5.7.3.1</li>
</ol>
<ul>
<li>nlfId – The routeId for the county LRM</li>
<li>nlfIdSt – The routeId for the state LRM</li>
<li>ctlBeginNbr – The county LRM begin boundary</li>
<li>ctlEndNbr – The county LRM end boundary</li>
<li>stlBeginNbr – The state LRM begin boundary.  If the LRM type is for the county route, then the boundaries are matched to those of the county measures</li>
<li>stlEndNbr – The state LRM end boundary. If the LRM type is for the county route, then the boundaries are matched to those of the county measures</li>
<li></li>
</ul>
<p><strong>Example Request:</strong> Get a county route boundary</p>
<p>GET /routes/bounds/SFRAIR00070**C?year=2018&amp;lrmType=CTL_3DECIMAL</p>
<p>In this example, the state measure 86.043 corresponds to the county measure 0.000.  They likewise match for the end measures.</p>
<p><strong>Example Request:</strong> Get a state route boundary</p>
<p>GET /routes/bounds/SIR00070**CPRE?year=2018&amp;lrmType=STL_3DECIMAL</p>
<p>Note that in this example, only attributes related to the state measures are provided, and the full boundaries of the route are provided</p>
<ol>
<li></li>
<li>5.8Route Search</li>
</ol>
<p>This end point retrieves routes on both LrmTypes along with optional min/max boundaries.  This is a post request where the user passes a json object that defines the search configuration.  The choices made determine the attributes in the returned response.</p>
<p><strong>End Point:</strong> POST /routes/search/</p>
<ol>
<li></li>
<li></li>
<li>
<p>5.8.1Request Body</p>
</li>
<li></li>
<li></li>
<li></li>
<li>
<p>5.8.1.1Attribute Description</p>
</li>
</ol>
<ul>
<li><strong>perpYear–</strong> {Integer, optional} The perp year on which to search for the route.  This field is optional, and if not provided, shall use the current active perp year.</li>
<li><strong>includeBoundaries –</strong> {Boolean, optional, default: true} this indicates that route boundaries for the selected LRMs should be included in the result</li>
<li><strong>stateLrm–</strong> {Boolean, default: false} Indicates that the state LRM attributes should be included in the query and result, in this case meaning the nlfIdSt, stlBeginNbr, and stlEndNbr</li>
<li><strong>countyLrm–</strong> {Boolean, default: true} Indicates that the county LRM attributes should be included in the query and result, in this case meaning the nlfId, ctlBeginNbr, and ctlEndNbr</li>
<li><strong>jurisdictions –</strong> {List&amp;lt;String&amp;gt;, optional} List of route jurisdiction codes on which to filter.</li>
<li><strong>counties –</strong> {List&amp;lt;String&amp;gt;, optional}} List of county codes on which to filter.</li>
<li><strong>routeCodes –</strong> {List&amp;lt;String&amp;gt;, optional} List of route trans route codes on which to filter.</li>
<li><strong>routeNbrs –</strong> {List&amp;lt;String&amp;gt;, optional} List of route numbers on which to filter.</li>
<li><strong>extensionCodes –</strong> {List&amp;lt;String&amp;gt;, optional} List of route extension codes on which to filter.</li>
<li><strong>descriptionCodes –</strong> {List&amp;lt;String&amp;gt;, optional} List of route description codes on which to filter.</li>
<li><strong>directionalCodes –</strong> {List&amp;lt;String&amp;gt;, optional} List of route direction codes on which to filter. By default, all queries are filtered with the ["C"] unless the non-cardinal direction is specified in the request.</li>
</ul>
<ol>
<li></li>
<li></li>
<li>5.8.2Response</li>
</ol>
<p>The response object minimally includes a list of objects that can contain only a routeId unless includeBoundaries is set to true.  In this case, it will include either state, county or both LRMs depending on the stateLrm/countyLrm request settings.</p>
<ol>
<li></li>
<li></li>
<li></li>
<li>5.8.2.1 Response Object</li>
</ol>
<ul>
<li>nlfId – The routeId for the county LRM</li>
<li>nlfIdSt – The routeId for the state LRM</li>
<li>ctlBeginNbr – The county LRM begin boundary</li>
<li>ctlEndNbr – The county LRM end boundary</li>
<li>stlBeginNbr – The state LRM begin boundary.  If the LRM type is for the county route, then the boundaries are matched to those of the county measures</li>
<li>stlEndNbr – The state LRM end boundary. If the LRM type is for the county route, then the boundaries are matched to those of the county measures</li>
</ul>
<ol>
<li></li>
<li></li>
<li>5.8.3Example Request:</li>
</ol>
<p>POST /routes/ri/events</p>
<p><strong>Request Body</strong></p>
<p><strong>Response</strong></p>
<ol>
<li></li>
<li>5.9Route Roadway Inventory Events</li>
</ol>
<p>This end point retrieves route linear and point events based on the configuration of the request body object.</p>
<p><strong>End Point:</strong> POST /routes/ri/events/</p>
<ol>
<li></li>
<li></li>
<li>
<p>5.9.1Request Body</p>
</li>
<li></li>
<li></li>
<li></li>
<li>
<p>5.9.1.1Attribute Description</p>
</li>
</ol>
<ul>
<li><strong>lrmType–</strong> {String, optional} This accepts an enumerated type of the following values: [CTL_3DECIMAL, STL_3DECIMAL].  If not provided, it defaults to CTL_3DECIMAL</li>
<li><strong>allAttributes –</strong> {Boolean, optional, default: false} this indicates that all available attributes related to an event from the Roadway Inventory should be included in the response.</li>
<li><strong>attributes –</strong> {List&amp;lt;String&amp;gt;, optional} This is an alternative to the allAttributes option, where the caller may specify which attributes are to be included in the result. Reducing the number of attributes usually reduces the number of segments that will be returned and lightens the response payload.</li>
<li><strong>continuous–</strong> {Boolean, default: true} relavent when the County Route LRM Type has been selected.  This indicator implies that when routes that leave the county (County leave/reenters) and return are part of the result, that the segments related to the sections that left the county are included in the result.</li>
<li><strong>segments–</strong> {List&amp;lt;Object&amp;gt;, optional} The list of segments to return as part of the result.
<ul>
<li><strong>oo</strong>** id –** {String, optional}} An id that is added to the response to help track this requested segment.  The id becomes an attribute in the response containing the array of segments that were retrieved and associated with this request.  If an id is NOT provided, then the index of the item from the request is used.  Note that ids can be numbers or strings.</li>
<li><strong>oo</strong>** perpYear–** {Integer, optional} The perp year on which to search for the event.  This field is optional, and if not provided, shall use the current active perp year.</li>
<li><strong>oo</strong>** routeId–** {String, required} the route identifier for the event to be retrieved.  The LrmType shall dictate whether this is a state route id or a county route id.</li>
<li><strong>oo</strong>** beginMeasure –** {Decimal, optional, defaults to beginning of route} The begin measure of the events to retrieve.</li>
<li><strong>oo</strong>** endMeasure –** {Decimal, optional, defaults to end of route} The end measure of the events to retrieve.</li>
</ul>
</li>
</ul>
<ol>
<li></li>
<li></li>
<li>5.9.2Response</li>
</ol>
<p>The response object minimally includes a list of objects that can contain only a routeId unless includeBoundaries is set to true.  In this case, it will include either state, county or both LRMs depending on the stateLrm/countyLrm request settings.</p>
<ol>
<li></li>
<li></li>
<li></li>
<li>5.9.2.1 Response Object</li>
</ol>
<p>The response object is a json map object where the key is the original id of the requested data, and the value is an array of the linear events that satisfy the request from the posted data.</p>
<p><strong>Event Object Common Fields</strong></p>
<ul>
<li>Id – a unique id assigned to this event</li>
<li>perpYear – The perpYear to which the data matches</li>
<li>nlfId – The routeId for the county LRM</li>
<li>nlfIdSt – The routeId for the state LRM</li>
<li>beginMeasure – The beginMeasure for this event based on the lrmType</li>
<li>endMeasure – The endMeasure for this event based on the lrmType</li>
<li>primary – Boolean value indicating the section is primary</li>
<li>primarySegment.nlfId – provided when the current segment is secondary.  This is the nlfId of the primary county route over this section</li>
<li>primarySegment.beginMeasure - The primary route's beginMeasure correlating to the segment's beginMeasure</li>
<li>primarySegment.endMeasure - The primary route's endMeasure correlating to the segment's endMeasure</li>
<li>leave – Boolean value indicating that the end point of this event is adjacent to a discontinuity</li>
<li>reenter – Boolean value indicating that the begin point of this event is adjacent to a discontinuity</li>
<li>overlapInverseInd – Boolean value that indicates that the secondary event is digitized in the opposite direction of its corresponding primary</li>
<li>stl3dBeginNbr– The state LRM begin boundary. This is included when the county LRM is requested for reference.</li>
<li>stl3dEndNbr– The state LRM end boundary. This is included when the county LRM is requested for reference.</li>
</ul>
<p><strong>Event Object Optional Attributes</strong></p>
<p>In addition to the common fields, the requester can pick and choose other supported fields within the roadway inventory data.  To view the complete list of available fields, see the <strong>Routes RI Attributes</strong> end point.</p>
<ol>
<li></li>
<li></li>
<li>5.9.3Example Request:</li>
</ol>
<p>POST /routes/ri/events</p>
<p><strong>Request Body</strong></p>
<p><strong>Response</strong></p>
<ol>
<li></li>
<li>5.10Get Routes RI Attributes</li>
</ol>
<p>Retrieves the available attributes/metadata for the Roadway Inventory (RI) dataset.  The attributes are available for use with the <strong>Route Roadway Inventory Events</strong> end point.</p>
<p><strong>End Point:</strong> GET /routes/ri/attributes</p>
<ol>
<li></li>
<li></li>
<li>
<p>5.10.1Response</p>
</li>
<li>
<p>6Streets API</p>
</li>
</ol>
<p>The streets API provides information specifically relating street names to routes.  This linear event dataset does not fully cover all routes in the roadway inventory, but only the ones that actually have a street name.</p>
<p>The data also relates the street name to the governing body of the route, specifically the county, township and municipality.  There are cases where a street can be in 2 different municipalities (jurisdictional split) and/or a split between townships and municipalities.  In these cases there will be a record for each governing body.</p>
<ol>
<li></li>
<li>6.1Street Name Attributes</li>
</ol>
<p>Retrieves the available attributes/metadata for the Street Event dataset.  The attributes are available for use with the <strong>Street Search</strong> end point.</p>
<p><strong>End Point:</strong> GET /streets/attributes</p>
<ol>
<li></li>
<li></li>
<li>
<p>6.1.1Response</p>
</li>
<li></li>
<li>
<p>6.2Street Event Search</p>
</li>
</ol>
<p>This end point retrieves street name events based on the posted search criteria</p>
<p><strong>End Point:</strong> POST /streets/search</p>
<ol>
<li></li>
<li></li>
<li>
<p>6.2.1Request Body</p>
</li>
<li></li>
<li></li>
<li></li>
<li>
<p>6.2.1.1Attribute Description</p>
</li>
</ol>
<ul>
<li><strong>perpYear–</strong> {Integer, optional} The perp year on which to search for the route.  This field is optional, and if not provided, shall use the current active perp year.</li>
<li><strong>lrmType–</strong> {String, optional} This accepts an enumerated type of the following values: [CTL_3DECIMAL, STL_3DECIMAL].  If not provided, it defaults to CTL_3DECIMAL</li>
<li><strong>allAttributes –</strong> {Boolean, optional, default: false} this indicates that all available attributes related to an event from the Street Names should be included in the response.</li>
<li><strong>attributes –</strong> {List&amp;lt;String&amp;gt;, optional} This is an alternative to the allAttributes option, where the caller may specify which attributes are to be included in the result. Reducing the number of attributes usually reduces the number of segments that will be returned and lightens the response payload.</li>
<li><strong>jurisdictions –</strong> {List&amp;lt;String&amp;gt;, optional} List of route jurisdiction codes on which to filter.</li>
<li><strong>counties –</strong> {List&amp;lt;String&amp;gt;, optional}} List of county codes on which to filter.</li>
<li><strong>routeCodes –</strong> {List&amp;lt;String&amp;gt;, optional} List of route trans route codes on which to filter.</li>
<li><strong>routeNbrs –</strong> {List&amp;lt;String&amp;gt;, optional} List of route numbers on which to filter.</li>
<li><strong>extensionCodes –</strong> {List&amp;lt;String&amp;gt;, optional} List of route extension codes on which to filter.</li>
<li><strong>descriptionCodes –</strong> {List&amp;lt;String&amp;gt;, optional} List of route description codes on which to filter.</li>
<li><strong>directionalCodes –</strong> {List&amp;lt;String&amp;gt;, optional} List of route direction codes on which to filter. By default, all queries are filtered with the ["C"] unless the non-cardinal direction is specified in the request.</li>
<li><strong>muniFipsCodes –</strong> {List&amp;lt;String&amp;gt;, optional} List of municipal FIPS codes on which to filter.</li>
<li><strong>townshipFipsCodes –</strong> {List&amp;lt;String&amp;gt;, optional} List of township FIPS codes on which to filter.</li>
<li><strong>streetNames –</strong> {List&amp;lt;String&amp;gt;, optional} List of street names on which to filter.</li>
</ul>
<ol>
<li></li>
<li></li>
<li>6.2.2Response</li>
</ol>
<p>Based on the example request body shown above</p>
<ol>
<li></li>
<li>6.3Street Name Search</li>
</ol>
<p>Retrieves a unique list of street names based on the provided criteria provided in the request body</p>
<p><strong>End Point:</strong> GET /streets/names</p>
<ol>
<li></li>
<li></li>
<li>6.3.1Request Body</li>
</ol>
<p>The request body structure is identical to <strong>Street Event Search</strong> end point, with the additions/changes of the following attributes:</p>
<ul>
<li><strong>maxResults –</strong> {number, optional} The maximum number of results to return.  This field is highly recommended for broad searches, especially where this might be used in an autocomplete interface scenario.</li>
<li><strong>lrmType</strong> – This field is ignored as it doesn't impact the result</li>
</ul>
<p><strong>Example Request Body</strong></p>
<ol>
<li></li>
<li></li>
<li>6.3.2Response</li>
</ol>
<p>Based on the request example above, the following is an example response.</p>
<ol>
<li>7Boundary API</li>
</ol>
<p>This API provides end points for retrieving boundary datasets such as County, Municipality, and Townships.</p>
<ol>
<li></li>
<li>7.1EndPoints</li>
</ol>
<ul>
<li>GET /boundaries/counties/ - Retrieves county information along with name and district</li>
<li>GET /boundaries/counties/{districtNbr} - Retrieves county information along with name and district for a specific district.</li>
<li>GET /boundaries/counties/ids - Retrieves just the list of county 3-character id codes.</li>
<li>GET /boundaries/municipalities } – retrieve all municipality records for the current perp year.</li>
<li>GET /boundaries/municipalities?county={countyCode} – retrieve the municipality list by county code for the current perp year.</li>
<li>GET /boundaries/townships } – retrieve all township records for the current perp year.</li>
<li>GET /boundaries/townships?county={countyCode} – retrieve the township list by county code for the current perp year.</li>
</ul>

