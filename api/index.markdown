---
layout: page
title: API Documentation
active: api
---

# API

This discribes the Arts Holland API. Arts Holland offers both a [SPARQL endpoint](#sparql) and a [REST API](#rest).

<a name="sparql"></a>
## SPARQL

All Arts Holland data can be accessed using SPARQL, a query language for RDF data. By submitting SPARQL queries to the SPARQL endpoint, you can retrieve exactly the data you need. More information about SPARQL is found in the [W3C SPARQL specification](http://www.w3.org/TR/rdf-sparql-query/).

The SPARQL endpoint is [http://api.artsholland.com/sparql](http://api.artsholland.com/sparql).

### RDF vocabulary

An [RDF vocabulary is available](http://api.artsholland.com/vocabulary/1.0/artsholland.rdf) for in-depth information about the Arts Holland data model. Also, a basic description of the Arts Holland data model can be found in the [REST API documentation](#rest) below.

### Tutorial

Besided this document, you can use the [interactive SPARQL tutorial]({{ site.baseurl }}tutorial) to guide you through the Arts Holland data, data model and the SPARQL queries you can use to access all the data.

### SPARQL requests

SPARQL queries can be submitted to the database server by doing a `GET` or `POST` request to `/sparql` and URL encoding the SPARQL query in the `query` parameter.

Make sure to set the `Content-Type` header to `application/x-www-form-urlencoded` and the `Accept` header to a media type from the table in the next section.

### Response formats

You can select one of the response data formats by setting the `Accept` header to the appropriate media type, or by appending an file extension to the request.

| Response                         | Extension | Accept header
|:---------------------------------|:----------|:---------------------------------|
| SPARQL Query Results XML Format  | `.rdf`    | `application/sparql-results+xml`
| SPARQL Query Results JSON Format | `.json`   | `application/sparql-results+json`
| SPARQL Query Results CSV         | `.csv`    | `text/csv`
| SPARQL browser                   |           | `text/html`

If you want to view the result in your browser, you can overwrite the response media type to `text/html` by specifying the request parameter `plaintext=true`.

### SPARQL browser

Arts Holland also provides a web-based SPARQL browser. You can use this browser to test SPARQL queries or just to get an impression about the data available in the Arts Holland database.

### RDF namespaces

The SPARQL browser automatically appends a set of default RDF namespace prefixes to all submitted SPARQL queries, something that you have to do yourself when doing direct `GET` or `POST` requests. Alternatively, you can rewrite your queries and use full URIs instead of namespaced ones.

For example, see the two following queries. With namespaces:

    PREFIX ah: <http://purl.org/artsholland/1.0/>
    PREFIX dc: <http://purl.org/dc/terms/>
    SELECT ?venue ?title {  
      ?venue a ah:Venue ;
        dc:title ?title .
    } LIMIT 10

Without namespaces:

    SELECT ?venue ?title {  
      ?venue a <http://purl.org/artsholland/1.0/Venue> ;
        <http://purl.org/dc/terms/title> ?title .
    } LIMIT 10

### Example request

The following query, for example, retrieves the URIs and English titles of 25 venues in Amsterdam from the Arts Holland database.

    SELECT DISTINCT ?venue ?title
    WHERE { 
      ?venue a ah:Venue .
      ?venue dc:title ?title .
      ?venue ah:locationAddress ?address .
      ?address vcard:locality "Amsterdam" . 
      FILTER(langMatches(lang(?title), "en"))
    } ORDER BY ?venue
    LIMIT 25

You can [paste this query in the Arts Holland SPARQL browser](http://api.artsholland.com/sparql?query=SELECT+DISTINCT+%3Fvenue+%3Ftitle%0D%0AWHERE+%7B+%0D%0A%09%3Fvenue+a+ah%3AVenue+.%0D%0A%09%3Fvenue+dc%3Atitle+%3Ftitle+.%0D%0A%09%3Fvenue+ah%3AlocationAddress+%3Faddress+.%0D%0A%09%3Faddress+vcard%3Alocality+%22Amsterdam%22+.+%0D%0A%09FILTER%28langMatches%28lang%28%3Ftitle%29%2C+%22en%22%29%29%0D%0A%7D+ORDER+BY+%3Fvenue%0D%0ALIMIT+25&selectoutput=browse), or directly do a `GET` or `POST` request to the SPARQL endpoint. In the latter case, don’t forget to add the following namespace prefixes:

    PREFIX ah: <http://purl.org/artsholland/1.0/>
    PREFIX dc: <http://purl.org/dc/terms/>
    PREFIX vcard: <http://www.w3.org/2006/vcard/ns#>

### Full text search

You can use the [full text search functionality](https://dev.opensahara.com/projects/useekm/wiki/IndexingSail#Full-Text-Search) of the [uSeekM library](https://dev.opensahara.com/projects/useekm) to do a full text search in the Arts Holland database. uSeekM text search currently only indexes `dc:description` and `dc:title` properties.

### Geo-spatial search

The uSeekM library is also used to add geo-spatial computations and indexing its database. This functionality can be used in all SPARQL queries submitted to the Arts Holland SPARQL endpoint. More documentation can be found in the [uSeekM wiki](https://dev.opensahara.com/projects/useekm/wiki/IndexingSail#GeoSPARQL).

<a name="rest" ></a>
##  REST API

Accessing Arts Holland using the SPARQL endpoint provides the most flexibility, but its query language and response formats might sometimes be too complex and overwhelming. Therefore, Arts Holland also provides a easy to use REST API with which you can access the most important parts of the Arts Holland semantic Open Linked Database.

The REST API endpoint is [http://api.artsholland.com/rest](http://api.artsholland.com/rest).

### Data model

The REST API only discloses the main elements in the Arts Holland database: events, productions and venues. If you want to access all the other data (such as information about hotels, restaurants and public transport stops) or if you want more control over the exact query and returned data, you can use the [SPARQL endpoint](#sparql).

The objects returned by the REST API are described below in short.

#### Main elements

| Element      | Description |
|:-------------|:------------|
| `Production` | A play, a movie, a concert, an exhibition, a lecture, etc. A production can be performed multiple times, and so can be be hosted by multiple events and venues.
| `Event`      | A instance of a production at a specific location and time.
| `Venue`      | A physical location where events take place.

Events, venues and productions are identified by a CIDN number managed by the Nederlands Uitburo.

#### Other elements

| Element               | Description |
|:----------------------|:------------|
| `Room`                | Child element of a venue, in which events can take place. A venue can have multiple rooms. Events can be held in one or more rooms of a specific venue.
| `Address`             | Child element of a venue, holds address information.
| `Attachment`          | Child element of both venues and events. An attachment holds information about images, movies or documents linked to the venue or event.
| `Offering`            | Child element of an event. Information about tickets; an event can have multiple offers.
| `PriceSpecification`  | Child element of an offer. Information about price and currency.
| `Genre`               | Productions are categorized by genre. Examples are dance, documentary, exhibition and musical.
| `VenueType`           | Venues are categorized by type. Museums, cinemas and theaters, for example, all have a different VenueType.

### URI structure

#### Object lists

| URI                                                                   | Description     |
|:----------------------------------------------------------------------|:----------------|
| [`/rest/event`](http://api.artsholland.com/rest/venue.json)           | All events
| [`/rest/venue`](http://api.artsholland.com/rest/venue.json)           | All venues
| [`/rest/production`](http://api.artsholland.com/rest/production.json) | All productions
| [`/rest/genre`](http://api.artsholland.com/rest/genre.json)           | All genres
| [`/rest/venuetype`](http://api.artsholland.com/rest/venuetype.json)   | All venue types

#### Single objects

| URI                                                                                                     | Description
|:--------------------------------------------------------------------------------------------------------|:----------------------------|
| [`/rest/event/{cidn}`](http://api.artsholland.com/rest/event/2008-a-047-0143827.json)                   | Event with CIDN `{cidn}`
| [`/rest/venue/{cidn}`](http://api.artsholland.com/rest/venue/09c50d40-279b-4e2c-bf72-f6e4a3d00ddf.json) | Venue with CIDN `{cidn}`
| [`/rest/production/{cidn}`](http://api.artsholland.com/rest/production/2011-p-003-0014068.json)         | Production with CIDN `{cidn}`

#### Child elements and relations

| URI                                                                                                     | Description
|:--------------------------------------------------------------------------------------------------------|:----------------------------|
| [`/rest/event/{cidn}/venue`](http://api.artsholland.com/rest/event/2008-a-047-0143827/venue.json)               | Venues in which event  `{cidn}` takes place
| [`/rest/event/{cidn}/production`](http://api.artsholland.com/rest/event/2008-a-047-0143827/production.json)     | Productions associated with event `{cidn}`
| [`/rest/event/{cidn}/room`](http://api.artsholland.com/rest/event/2008-a-047-0143827/room.json)                 | Rooms in which event  `{cidn}` takes place
| [`/rest/event/{cidn}/attachment`](http://api.artsholland.com/rest/event/2008-a-047-0143827/attmachent.json)     | Attachments associated with event `{cidn}`
| [`/rest/event/{cidn}/offering`](http://api.artsholland.com/rest/event/2008-a-047-0143827/offering.json)         | Offers available for event `{cidn}`
| [`/rest/event/{cidn}/offering/{name}/price`](http://api.artsholland.com/rest/event/2008-a-047-0143827/offering/normaal/price.json) | Price specification of offer `{name}`
| [`/rest/venue/{cidn}/event`](http://api.artsholland.com/rest/venue/dd7f0914-31d7-4b19-ab02-c68f82053cd1/event.json)           | Events which take place in venue `{cidn}`
| [`/rest/venue/{cidn}/production`](http://api.artsholland.com/rest/venue/dd7f0914-31d7-4b19-ab02-c68f82053cd1/production.json) | Productions which take place in venue `{cidn}`
| [`/rest/venue/{cidn}/room`](http://api.artsholland.com/rest/venue/254efcd5-66fa-4e17-b54b-62e515256a0f/room.json)             | Rooms in venue `{cidn}`
| [`/rest/venue/{cidn}/attachment`](http://api.artsholland.com/rest/venue/dd7f0914-31d7-4b19-ab02-c68f82053cd1/attachment.json) | Attachments associated with venue `{cidn}`
| [`/rest/venue/{cidn}/address`](http://api.artsholland.com/rest/venue/dd7f0914-31d7-4b19-ab02-c68f82053cd1/address.json)       | Addresses associated with venue `{cidn}`
| [`/rest/production/{cidn}/event`](http://api.artsholland.com/rest/production/63bdd2b4042e23e221b6635896240154/event.json)     | Events associated with production `{cidn}`
| [`/rest/production/{cidn}/venue`](http://api.artsholland.com/rest/production/63bdd2b4042e23e221b6635896240154/venue.json)     | Venues in which production `{cidn}` takes place
| [`/rest/production/{cidn}/attachment`](http://api.artsholland.com/rest/production/63bdd2b4042e23e221b6635896240154/attachment.json) | Attachments associated with production `{cidn}`

### Filters

| Parameter              | On element               | Description                                         
|:-----------------------|:-------------------------|:----------------------------------------------------
| [`search`](http://api.artsholland.com/rest/production.json?search=boston)               | Event, Production, Venue | Free text search on description field
| [`genre`](http://api.artsholland.com/rest/production.json?genre=GenreLiterature)                | Production               | Filters production by genre
| [`type`](http://api.artsholland.com/rest/venue.json?type=VenueTypeCinema)                 | Venue                    | Filters venues by venue type
| [`nearby`](http://api.artsholland.com/rest/venue.json?nearby=POINT%284.8931 52.3729%29&distance=2500)               | Event, Venue             | Finds events or venues nearby a specific geographic location. The filter accepts [WKT points](http://en.wikipedia.org/wiki/Well-known_text): `POINT(longitude latitude)`. A second argument specifies the distance in meters.
| [`locality`](http://api.artsholland.com/rest/venue.json?locality=utrecht)             | Event, Venue             | Filters events or venues on city or town. The locality filter is case-insensitive.
| [`before, after`](http://api.artsholland.com/rest/event.json?after=2014-01-02)        | Event                    | Filters events on start date and time, only returns events which start before or after specified date and time. The filters accept the ISO 8601 date and time format.
| [`min_price, max_price`](http://api.artsholland.com/rest/event.json?max_price=15) | Event                    | Finds events which have a ticket with a price (in any currency) of at least `min_price` or at most `max_price`.

Filters only work on first level main element API requests (e.g. `/rest/event`, `/rest/venue` and `/rest/production`).

### Response formats

You can select one of the response data formats by setting the `Accept` header to the appropriate media type, or by appending an file extension to the request.

| Response | Extension | Accept header
|:---------|:----------|:----------------------|
| RDF/XML  | `.rdf`    | `application/rdf+xml`
| JSON     | `.json`   | `application/json`
| Turtle   | `.turtle` | `text/turtle`

If you want to display API results in your browser, you can override the response media type by setting `plaintext=true`. This parameter will set the response media type to `text/html`, which your browser will nicely display instead of download to file.

JSONP is also supported: all REST API JSON results can be encapsulated in a JavaScript function call by setting `callback={callback}`.

### Localization

Text strings in the Arts Holland database are often localized, and available in more than one language (mostly Dutch and English). By specifying an IETF language tag parameter `lang={tag}` in the request URI it is possible to let the REST API only return strings in the specified language. Not all strings in the database are localized; those strings will always be returned, regardsless of the specified language tag.

The language parameter defaults to English (i.e. `lang=en`). Language filtering can be disabled by specifying `lang=any`.

### Pagination

API requests that can return more than one object are always paginated. Two URI parameters are available to control pagination: `per_page={per_page}&page={page}`.

The parameter `{per_page}` specifies the number of desired results per page, and `{page}` the desired page. The default results per page is 10.

If JSON is the desired return format of the request, paginated requests return pagination metadata in a dedicated JSON object:

    {
      "metadata": {
        "page": 3,
        "per_page": 15,
        ...
      },
      "results": [
        ...
      ]
    }

### Ordering

The REST API returns unordered results by default. This usually works fine for most applications, especially with filtered requests with small result sets. If your application, however, does require the results to be ordered, you can specify `ordered=true` in the request. The system will order the resulting objects by URI.

Please be warned that ordering has a performance impact, and will result in longer response times.

### Counting

The REST API will return the total number of available results for paged queries if `count=true` is added to the request parameters. The result will then include count metadata for that specific query, including filters:

    {
      "metadata": {
        "count": 718,
        ...
      },
      "results": [
        ...
      ]
    }

Counting will result in much longer response times as the REST API back-end has to submit the same query to the database twice; one time for counting the total number or results and one time for returning the data.

The count parameter only works for requests where JSON is the desired return format.

### Pretty-printed JSON results

The REST API will pretty-print the JSON it produces if the `pretty=true` request parameter is used.

Again, this parameter only works for JSON requests.

### Example requests

| Description                                                             | Request
|:------------------------------------------------------------------------|:----------------------------------------------------------------------------------|
| List of productions, results 16 to 20                                   | [`/rest/production.json?per_page=5&page=4`](http://api.artsholland.com/rest/production.json?per_page=5&page=4)
| Single production and all its string properties, regardless of language | [`/rest/production/0006816f-426c-4e43-8ac4-c8376f4dc3b4?lang=any`](http://api.artsholland.com/rest/production/0006816f-426c-4e43-8ac4-c8376f4dc3b4.json?lang=any)
| List of events in Amsterdam                                             | [`/rest/event?locality=amsterdam&count=true`](http://api.artsholland.com/rest/event.json?locality=amsterdam)
| All productions in the Stadsschouwburg Amsterdam, in RDF/XML format     | [`/rest/venue/04df157e-fc47-4448-83ed-d0a8c577d7dd/production.rdf`](http://api.artsholland.com/rest/venue/04df157e-fc47-4448-83ed-d0a8c577d7dd/production.rdf)
| All venues within 2.5 km. of Dam Square, Amsterdam                       | [`/rest/venue.json?nearby=POINT(4.8931 52.3729)&distance=2500`](http://api.artsholland.com/rest/venue.json?nearby=POINT%284.8931 52.3729%29&distance=2500)
| All events that take place after Januari 1st, 2014                       | [`/rest/event.json?after=2012-08-02&apiKey=9cbce178ed121b61a0797500d62cd440`](http://api.artsholland.com/rest/event.json?after=2014-01-01&apiKey=9cbce178ed121b61a0797500d62cd440)
| The address of the Nederlands Muziek Instituut in The Hague              | [`/rest/venue/010f8e45-5726-48db-b0a7-aa95abc98432/address.json`](http://api.artsholland.com/rest/venue/010f8e45-5726-48db-b0a7-aa95abc98432/address.json)