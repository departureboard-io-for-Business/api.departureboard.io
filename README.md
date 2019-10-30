# departureboard.io API

Welcome, this GitHub Repository is used for storing a copy of the departureboard.io API documentation which is used over at https://api.departureboard.io.

It is also used as a place for developers to raise issues and feature requests.


---

# Introduction

Welcome to the departureboard.io API. This project is provided by [departureboard.io for Business](https://businessdepartureboard.io). 

The departureboard.io API is a high performance API, written in Golang, that provides two main functions:

1. **A REST API interface to the legacy National Rail SOAP API:** Gives developers the ability to pull live information on departures, arrivals, and services from National Rail, without having to use the legacy SOAP API. Free to use, and information is still pulled directly from National Rail, providing the same level of real-time data without the additional complexity of having to interact with SOAP.

2. **A REST API interface for additional National Rail information:** Gives developers the ability to pull a range of information about the Rail Network, via a REST API interface. This is not an offering that National Rail currently provide, and is custom developed. Data is sourced from periodically updated XML documents, parsed, and provided for consumption via the departureboard.io API.

# Intelligent Caching

The departureboard.io API also implements custom designed "Intelligent Caching" which benefits all users of the API. It provides significantly improved performance for common API calls, and also reduces the amount of calls that are made to National Rail in the background.

Intelligent Caching works by storing the results for API calls for an extremely short period of time (20 seconds). This allows that response to be re-used for other users who are also making the same API call through the departureboard.io API. The benefits of this are twofold: 

* If someone has made the same API request as you within the past 20 seconds, your response will be served from the Intelligent Cache which on average is 5x faster than an uncached response.
* If your response is served from the Intelligent Cache, your request will not be sent to National Rail. This reduces the amount of National Rail API calls that are made using your API Key. If you are a high volume user this can save you significant money.

A response is only served from Intelligent Cache if someone has requested the **exact** same data as you within the past 20 seconds.

You can determine if your request was served by the Intelligent Cache by looking for the `DBAPI-Served-By-Intelligent-Cache` HTTP header. This will be either `TRUE` or `FALSE`. 

If the value is `TRUE` you can determine the age of the response via the `Age` HTTP header.

# FAQ

This section contains some answers to frequently asked questions. Please feel free to [get in touch](mailto:hello@departureboard.io) if you have any additional queries.

### Where can I keep up-to-date with departureboard.io API developments?

All news, information, and updates for the departureboard.io API will be announced via [the departureboard.io for Business Medium Page](https://medium.com/@departureboard.io_for_business).

There is also a detailed [Changelog](https://github.com/departureboard-io-for-Business/api.departureboard.io/blob/master/CHANGELOG.md) available on GitHub.

Typically, when a major new version of the API is released, the previous version will stay running for 6 months (180 days) before being retired. It is recommended you keep up to date with the [changelog](https://github.com/departureboard-io-for-Business/api.departureboard.io/blob/master/CHANGELOG.md) so you are aware of such changes. An issue will be raised on GitHub for the retirement of legacy API versions. It is recommended you "Watch" the repository on GitHub, so you will be alerted to upcoming version retirements.

The aim is to avoid making any major changes, but occasionally (such as when National Rail make upstream API changes) they are unavoidable.

Versioning is controlled via the URI path, for example version 2.0 of the API is accessible via `https://api.departureboard.io/api/v2.0/`.


### How up to date is the information provided by this API?
The departureboard.io API is powered by the official [National Rail Darwin API](https://www.nationalrail.co.uk/100296.aspx), providing a RESTful interface on top of the legacy SOAP API. Consequently, the information provided by the departureboard.io API is as up to date as the official National Rail feeds are. The National Rail API is used by the in-station departure boards, so typically the information provided is very accurate.

For API Endpoints that are custom developed by departureboard.io (such as `getStationBasicInfo` and `getStationDetailsByCRS`), information is still sourced via National Rail. This information is made available by National Rail in the form of lengthy XML documents containing hundreds of thousands of lines. departureboard.io downloads, and parses these documents each night at 00:30 UK time, transforming and storing them in a high-performance, redundant database backend. Consequently this information is also always up to date.

### Is it slower to use this API vs. the National Rail SOAP API?

For a query that **is not served by the Intelligent Cache**: In short, often yes. But only marginally so.

The departureboard.io API is incredibly high performance, written in Golang, typically only adding a few milliseconds to a request. There are certain scenarios where you may find it quicker to consume the departureboard.io API than the National Rail API, for example:

* The departureboard.io API may be quicker than your application at crafting the SOAP XML payload, and parsing the SOAP XML response, so you may find it quicker than using the National Rail API directly.
* The departureboard.io API is hosted in Google Cloud Platform, across 3 Availability Zones. It utilises Premium Networking, taking advantage of Cold Potato Routing. Consequently, depending on the origin of the request, you may see quicker response times vs. going to the National Rail API directly due to network optimisation.

Each API request that interfaces with National Rail returns a `nationalRailResponseTimeMilliseconds` property, detailing how long the request to National Rail took in milliseconds. This can be used to determine how much time the departureboard.io API added to the request. 

For queries that **are served by the Intelligent Cache**: You can expect responses to be up to 5x faster than going to National Rail directly.


### How do I get a National Rail API Key?

It is quick and easy to get a National Rail OpenLDBWS API Key. Just visit the [registration page](http://realtime.nationalrail.co.uk/OpenLDBWSRegistration/), and sign up.

### What can I make with this API?

The possibilities are almost endless, but some initial ideas:

* A Live Train departure site, such as [departureboard.io](https://departureboard.io)
* A physical departure screen for your house to show live departures from your home station
* An alerting application to alert you when your regular train is delayed
* A web-based tool to look up detailed Train Station information

### How much does it cost?

Nothing, the departureboard.io API is free to use. You may be invoiced directly by National Rail if you exceed their free quota of 5 million API calls per 28-day railway period. See the [National Rail Charging Document](https://www.nationalrail.co.uk/static/documents/Usage_Charging_Document.pdf) for more information.

If your response is served by the [Intelligent Cache](#intelligent-caching), then a request is not made to National Rail so it will not count as usage against your API Key. For example, if you make 10 million API calls to the departureboard.io API, and 5 million of those calls are returned from the Intelligent Cache. Then National Rail will only see 5 million calls against your API Key.

Custom developed endpoints (such as `getStationBasicInfo` and `getStationDetailsByCRS`) **do not** require a National Rail API key and are also free of charge.

### Can I make high-volume usage of this API?

Reasonably so. The aim of this API is to make it simple for developers to consume National Rail information.

In order to provide a reliable and speedy service for everyone, at a reasonable cost, the following rate limits are in place:

Type                              | Included API Endpoints  | Rate Limit     
--------------------------------- | ---------------------   | --------------
National Rail Backed API Calls    | getDeparturesByCRS<br>getArrivalsByCRS<br>getArrivalsAndDeparturesByCRS<br>getServiceDetailsByID<br>getNextDeparturesByCRS<br>getFastestDeparturesByCRS | 1000/req Minute
Tier 1 departureboard.io API Calls | getStationBasicInfo     | No Limit
Tier 2 departureboard.io API Calls | getStationDetailsByCRS  | No Limit


After exceeding these limits, you may start to see a 429 HTTP Response. For higher volume usage please [get in touch](mailto:hello@departureboard.io) to arrange a higher limit.

National Rail also have limits on how many API requests can be made against an API Key. You should see the [Usage Charging Document](https://www.nationalrail.co.uk/static/documents/Usage_Charging_Document.pdf) for moree information.


### Is Cross-Origin Resource Sharing (CORS) enabled?

Yes, the departureboard.io allows CORS. However, be careful about exposing your National Rail API Key if calling this API client side. For example, via AJAX or JavaScript. It is possible to have a custom API FQDN configured that does not require your API Key to be specified as a Query Parameter, with a custom CORS configuration. [Get in touch](mailto:hello@departureboard.io) to discuss setting this up.

### Is this a reliable service?

The departureboard.io API is provided without any official SLA.

However, the core application components run in Google Cloud Platform on Kubernetes across 3 Availability Zones in Europe-West2. The backend database used to support the custom developed API Endpoints is replicated across 6 regions.

The service will automatically scale in response to demand, and a strict route-to-live process is followed to minimise bugs in production. You can confidently make use of this service.

# Departures and Arrivals

The following set of API Endpoints allow you to get departure and arrival information for train stations across the UK.

## Get Departures by CRS

```shell
curl -X GET "https://api.departureboard.io/api/v2.0/getDeparturesByCRS/KGX/?apiKey=0000a0a0-00aa-00dc-0000-0a000a00a0aa&numServices=1" -H "accept: */*"
```


> The above command returns JSON structured like this:

```json
{
  "generatedAt": "2019-09-10T13:49:03.1769126+01:00",
  "nationalRailResponseTimeMilliseconds": 127,
  "locationName": "London Kings Cross",
  "filterLocationName": null,
  "filtercrs": null,
  "filterType": null,
  "crs": "KGX",
  "nrccMessages": [
    {
      "message": "Additional trains to Great Northern destinations also operate from London St Pancras International (STP) which is a short walk from London Kings Cross."
    }
  ],
  "platformAvailable": true,
  "areServicesAvailable": null,
  "trainServices": [
    {
      "serviceID": "FPhH5Dlqy19zPg8uHxCT1Q==",
      "serviceIDUrlSafe": "FPhH5Dlqy19zPg8uHxCT1Q--",
      "serviceType": "train",
      "isCancelled": null,
      "filterLocationCancelled": null,
      "cancelReason": null,
      "delayReason": null,
      "adhocAlert": null,
      "std": "13:48",
      "etd": "On time",
      "platform": "6",
      "operator": "Hull Trains",
      "operatorCode": "HT",
      "isCircularRoute": null,
      "length": null,
      "detachFront": null,
      "isReverseFormation": null,
      "rsid": null,
      "origin": [
        {
          "locationName": "London Kings Cross",
          "crs": "KGX",
          "via": null,
          "futureChangeTo": null,
          "assocIsCancelled": null
        }
      ],
      "destination": [
        {
          "locationName": "Hull",
          "crs": "HUL",
          "via": null,
          "futureChangeTo": null,
          "assocIsCancelled": null
        }
      ],
      "currentOrigins": null,
      "currentDestinations": null,
      "subsequentCallingPointsList": [
        {
          "subsequentCallingPoints": [
            {
              "locationName": "Grantham",
              "crs": "GRA",
              "st": "14:49",
              "et": "On time",
              "at": null,
              "length": null
            },
            {
              "locationName": "Hull",
              "crs": "HUL",
              "st": "16:18",
              "et": "On time",
              "at": null,
              "length": null
            }
          ]
        }
      ]
    }
  ],
  "busServices": null,
  "ferryServices": null
}
```
getDeparturesByCRS is used to get a list of services departing from a UK train station by the CRS (Computer Reservation System) code. This will typically return a list of train services, but will also return any replacement bus or ferry services that are in place.

### HTTP Request

`GET https://api.departureboard.io/api/v2.0/getDeparturesByCRS/{CRS}/`

### URL Parameters

Parameter | Required | Default     | Description
--------- | -------  | ----------- | -----------
CRS       | **true**     | none        | The CRS (Computer Reservation System) for the station you wish to get departure information for, e.g. KGX for London Kings Cross.

### Query Parameters

Parameter | Required | Default     | Description
--------- | -------  | ----------- | -----------
apiKey    | **true**     | none        | The National Rail OpenLDBWS API Key to use for looking up service information. You must [register with National Rail](http://realtime.nationalrail.co.uk/OpenLDBWSRegistration/.) to obtain this key.
numServices | false     | 10        | The number of departing services to return. This is a maximum value, less may be returned if there is not a sufficient amount of services running from the selected station within the time window specified.
timeOffset | false     | 0          | The time window in minutes to offset the departure information by. For example, a value of 20 will not show services departing within the next 20 minutes.
timeWindow | false     | 120        | The time window to show services for in minutes. For example, a value of 60 will show services departing the station in the next 60 minutes.
serviceDetails | false     | true   | Should the response contain information on the calling points for each service? If set to false, calling points will not be returned.
filterStation | false     | none    | The CRS (Computer Reservation System) code to filter the results by. For example, performing a lookup for PAD (London Paddington) and setting `filterStation` to RED (Reading), will only show services departing from Paddington that stop at Reading.

### Example Queries

* Lookup departures from London Kings Cross, with default query parameters:

`curl -X GET "https://api.departureboard.io/api/v2.0/getDeparturesByCRS/KGX/?apiKey=0000a0a0-00aa-00dc-0000-0a000a00a0aa" -H "accept: */*"`

* Lookup departures from London Kings Cross, showing 5 results:

`curl -X GET "https://api.departureboard.io/api/v2.0/getDeparturesByCRS/KGX/?apiKey=0000a0a0-00aa-00dc-0000-0a000a00a0aa&numServices=5" -H "accept: */*"`

* Lookup departures from London Kings Cross, showing 5 results, including services from up to 20 minutes in the past:

`curl -X GET "https://api.departureboard.io/api/v2.0/getDeparturesByCRS/KGX/?apiKey=0000a0a0-00aa-00dc-0000-0a000a00a0aa&numServices=5&timeOffset=-20" -H "accept: */*"`

### Notes

When the `serviceDetails` query parameter is set to `true` for `getDeparturesByCRS`, only **subsequent calling points** will be returned. This is by design from National Rail.

To get subsequent **and** previous calling points in the same response, use the `getArrivalsAndDeparturesByCRS` or `getServiceDetailsByID` endpoint.

It is possible to have two sets of `subsequentCallingPoints` returned for a service (when looking up train services that split). Consequently, `subsequentCallingPoints` are returned in the `subsequentCallingPointsList` array. **However**, in 99% of circumstances there are only 1 set of `subsequentCallingPoints` returned. For services that do not split, you can use `subsequentCallingPointsList[0].subsequentCallingPoints` to access the previous calling points. The [Python Code Samples](https://github.com/benjamin-maynard/departureboard-io-api-code-examples) have examples of how this works.

## Get Arrivals by CRS

```shell
curl -X GET "https://api.departureboard.io/api/v2.0/getArrivalsByCRS/KGX/?apiKey=0000a0a0-00aa-00dc-0000-0a000a00a0aa&numServices=1" -H "accept: */*"
```


> The above command returns JSON structured like this:

```json
 {
  "generatedAt": "2019-09-10T13:47:43.2393326+01:00",
  "nationalRailResponseTimeMilliseconds": 113,
  "locationName": "London Kings Cross",
  "filterLocationName": null,
  "filtercrs": null,
  "filterType": null,
  "crs": "KGX",
  "nrccMessages": [
    {
      "message": "Additional trains to Great Northern destinations also operate from London St Pancras International (STP) which is a short walk from London Kings Cross."
    }
  ],
  "platformAvailable": true,
  "areServicesAvailable": null,
  "trainServices": [
    {
      "serviceID": "kIPdqWwazsxnPprAbvdDsQ==",
      "serviceIDUrlSafe": "kIPdqWwazsxnPprAbvdDsQ--",
      "serviceType": "train",
      "isCancelled": null,
      "filterLocationCancelled": null,
      "cancelReason": null,
      "delayReason": null,
      "adhocAlert": null,
      "sta": "13:44",
      "eta": "13:53",
      "platform": null,
      "operator": "Grand Central",
      "operatorCode": "GC",
      "isCircularRoute": null,
      "length": null,
      "detachFront": null,
      "isReverseFormation": null,
      "rsid": null,
      "origin": [
        {
          "locationName": "Bradford Interchange",
          "crs": "BDI",
          "via": null,
          "futureChangeTo": null,
          "assocIsCancelled": null
        }
      ],
      "destination": [
        {
          "locationName": "London Kings Cross",
          "crs": "KGX",
          "via": null,
          "futureChangeTo": null,
          "assocIsCancelled": null
        }
      ],
      "currentOrigins": null,
      "currentDestinations": null,
      "previousCallingPointsList": [
        {
          "previousCallingPoints": [
            {
              "locationName": "Bradford Interchange",
              "crs": "BDI",
              "st": "10:22",
              "et": null,
              "at": "On time",
              "length": null
            },
            {
              "locationName": "Doncaster",
              "crs": "DON",
              "st": "12:04",
              "et": null,
              "at": "12:09",
              "length": null
            }
          ]
        }
      ]
    }
  ],
  "busServices": null,
  "ferryServices": null
}
```

getArrivalsByCRS is used to get a list of services arriving to a UK train station by the CRS (Computer Reservation System) code. This will typically return a list of train services, but will also return any replacement bus or ferry services that are in place.

### HTTP Request

`GET https://api.departureboard.io/api/v2.0/getArrivalsByCRS/{CRS}/`

### URL Parameters

Parameter | Required | Default     | Description
--------- | -------  | ----------- | -----------
CRS       | **true**     | none        | The CRS (Computer Reservation System) for the Station you wish to get arrival information for, e.g. KGX for London Kings Cross.


### Query Parameters

Parameter | Required | Default     | Description
--------- | -------  | ----------- | -----------
apiKey    | **true**     | none        | The National Rail OpenLDBWS API Key to use for looking up service information. You must [register with National Rail](http://realtime.nationalrail.co.uk/OpenLDBWSRegistration/.) to obtain this key.
numServices | false     | 10        | The number of arriving train services to return. This is a maximum value, less may be returned if there is not a sufficient amount of services running to this station within the time window specified.
timeOffset | false     | 0          | The time window in minutes to offset the arrival information by. For example, a value of 20 will not show services arriving within the next 20 minutes.
timeWindow | false     | 120        | The time window to show train services for in minutes. For example, a value of 60 will show services arriving to the station in the next 60 minutes.
serviceDetails | false     | true   | Should the response contain information on the calling points for each service? If set to false, calling points will not be returned.
filterStation | false     | none    | The CRS (Computer Reservation System) code to filter the results by. For example, performing a lookup for PAD (London Paddington) and setting `filterStation` to RED (Reading), will only show services arriving to Paddington that stopped at Reading.


### Example Queries

* Lookup arrivals to London Kings Cross, with default query parameters:

`curl -X GET "https://api.departureboard.io/api/v2.0/getArrivalsByCRS/KGX/?apiKey=0000a0a0-00aa-00dc-0000-0a000a00a0aa" -H "accept: */*"`

* Lookup arrivals to London Kings Cross, showing 5 results:

`curl -X GET "https://api.departureboard.io/api/v2.0/getArrivalsByCRS/KGX/?apiKey=0000a0a0-00aa-00dc-0000-0a000a00a0aa&numServices=5" -H "accept: */*"`

* Lookup arrivals to London Kings Cross, showing 5 results, including services from up to 20 minutes in the past:

`curl -X GET "https://api.departureboard.io/api/v2.0/getArrivalsByCRS/KGX/?apiKey=0000a0a0-00aa-00dc-0000-0a000a00a0aa&numServices=5&timeOffset=-20" -H "accept: */*"`

### Notes

When the `serviceDetails` query parameter is set to `true` for getArrivalsByCRS, only **previous calling points** will be returned. This is by design from National Rail.

To get previous **and** subsequent Calling Points in the same response, use the `getArrivalsAndDeparturesByCRS` or `getServiceDetailsByID` endpoint.

It is possible to have two sets of `previousCallingPoints` returned for a service (when looking up train services that split). Consequently, `previousCallingPoints` are returned in the `previousCallingPointsList` array. **However**, in 99% of circumstances there are only 1 set of `previousCallingPoints` returned. For services that do not split, you can use `previousCallingPointsList[0].previousCallingPoints` to access the previous calling points. The [Python Code Samples](https://github.com/benjamin-maynard/departureboard-io-api-code-examples) have examples of how this works.


## Get Arrivals and Departures by CRS

```shell
curl -X GET "https://api.departureboard.io/api/v2.0/getArrivalsAndDeparturesByCRS/HUN/?apiKey=0000a0a0-00aa-00dc-0000-0a000a00a0aa&numServices=1" -H "accept: */*"
```


> The above command returns JSON structured like this:

```json
{
  "generatedAt": "2019-09-10T13:45:31.0349152+01:00",
  "nationalRailResponseTimeMilliseconds": 117,
  "locationName": "Huntingdon",
  "filterLocationName": null,
  "filtercrs": null,
  "filterType": null,
  "crs": "HUN",
  "nrccMessages": null,
  "platformAvailable": true,
  "areServicesAvailable": null,
  "trainServices": [
    {
      "serviceID": "A/A9aZIIFKIkHdE9sMyrng==",
      "serviceIDUrlSafe": "A_A9aZIIFKIkHdE9sMyrng--",
      "serviceType": "train",
      "isCancelled": null,
      "filterLocationCancelled": null,
      "cancelReason": null,
      "delayReason": null,
      "adhocAlert": null,
      "std": "13:49",
      "etd": "On time",
      "sta": "13:48",
      "eta": "On time",
      "platform": "3",
      "operator": "Thameslink",
      "operatorCode": "TL",
      "isCircularRoute": null,
      "length": 12,
      "detachFront": null,
      "isReverseFormation": null,
      "rsid": null,
      "origin": [
        {
          "locationName": "Horsham",
          "crs": "HRH",
          "via": null,
          "futureChangeTo": null,
          "assocIsCancelled": null
        }
      ],
      "destination": [
        {
          "locationName": "Peterborough",
          "crs": "PBO",
          "via": null,
          "futureChangeTo": null,
          "assocIsCancelled": null
        }
      ],
      "currentOrigins": null,
      "currentDestinations": null,
      "subsequentCallingPointsList": [
        {
          "subsequentCallingPoints": [
            {
              "locationName": "Peterborough",
              "crs": "PBO",
              "st": "14:05",
              "et": "On time",
              "at": null,
              "length": null
            }
          ]
        }
      ],
      "previousCallingPointsList": [
        {
          "previousCallingPoints": [
            {
              "locationName": "Horsham",
              "crs": "HRH",
              "st": "11:25",
              "et": null,
              "at": "On time",
              "length": null
            },
            {
              "locationName": "St Neots",
              "crs": "SNO",
              "st": "13:41",
              "et": null,
              "at": "On time",
              "length": null
            }
          ]
        }
      ]
    }
  ],
  "busServices": null,
  "ferryServices": null
}
```

getArrivalsAndDeparturesByCRS is used to get a list of services arriving to and departing from a UK train station by the CRS (Computer Reservation System) code. This will typically return a list of train services, but will also return any replacement bus or ferry services that are in place.

### HTTP Request

`GET https://api.departureboard.io/api/v2.0/getArrivalsAndDeparturesByCRS/{CRS}/`

### URL Parameters

Parameter | Required | Default     | Description
--------- | -------  | ----------- | -----------
CRS       | **true**     | none        | The CRS (Computer Reservation System) for the Station you wish to get departure and arrival information for, e.g. KGX for London Kings Cross.


### Query Parameters

Parameter | Required | Default     | Description
--------- | -------  | ----------- | -----------
apiKey    | **true**     | none        | The National Rail OpenLDBWS API Key to use for looking up service information. You must [register with National Rail](http://realtime.nationalrail.co.uk/OpenLDBWSRegistration/.) to obtain this key.
numServices | false     | 10        | The number of arriving and departing services to return. This is a maximum value, less may be returned if there is not a sufficient amount of services arriving to or departing from this station within the time window specified.
timeOffset | false     | 0          | The time window in minutes to offset the arrival and departure information by. For example, a value of 20 will not show services arriving to or departing from the station within the next 20 minutes.
timeWindow | false     | 120        | The time window in minutes to offset the arrival and departure information by. For example, a value of 20 will not show services arriving to or departing from the selected station within the next 20 minutes.
serviceDetails | false     | true   | Should the response contain information on the calling points for each service? If set to false, calling points will not be returned.
filterStation | false     | none    | The CRS (Computer Reservation System) code to filter the results by. <br><br> **When setting this you must also set the `filterType` parameter**. <br><br>For example, performing a lookup for PAD (London Paddington) and setting `filterStation` to RED (Reading) and `filterType` to `from`, will only show services arriving to London Paddington that stopped at Reading. <br><br>Setting a filter for `getArrivalsAndDeparturesByCRS` is similar to performing a `getArrivalsByCRS` or `getDeparturesByCRS` lookup, with the appropriate `filterStation` parameter. However using the `getArrivalsAndDeparturesByCRS` endpoint shows more details for each of the returned services.
filterType | Only if filterStation is set     | none    | Determines if the `filterStation` parameter should be applied for services arriving to, or leaving from the selected station.

### Example Queries

* Lookup arrivals to, and departures from London Kings Cross, with default query parameters:

`curl -X GET "https://api.departureboard.io/api/v2.0/getArrivalsAndDeparturesByCRS/KGX/?apiKey=0000a0a0-00aa-00dc-0000-0a000a00a0aa" -H "accept: */*"`

* Lookup arrivals to, and departures from London Kings Cross, showing 5 results:

`curl -X GET "https://api.departureboard.io/api/v2.0/getArrivalsAndDeparturesByCRS/KGX/?apiKey=0000a0a0-00aa-00dc-0000-0a000a00a0aa&numServices=5" -H "accept: */*"`

* Lookup services for arriving to Reading, that stopped at Hayes & Harlington on the way:

`curl -X GET "https://api.departureboard.io/api/v2.0/getArrivalsAndDeparturesByCRS/RED/?apiKey=0000a0a0-00aa-00dc-0000-0a000a00a0aa&filterStation=HAY&filterType=from" -H "accept: */*"`

### Notes

It is possible to have two sets of `previousCallingPoints` or `subsequentCallingPoints` returned for a service (when looking up train services that split). Consequently, `previousCallingPoints` and `subsequentCallingPoints` are returned in the `previousCallingPointsList` and `subsequentCallingPointsList` arrays. **However**, in 99% of circumstances there are only 1 set of `previousCallingPoints` and `subsequentCallingPoints` returned. For services that do not split, you can use `previousCallingPointsList[0].previousCallingPoints` or `subsequentCallingPointsList[0].subsequentCallingPoints` to access the calling points. The [Python Code Samples](https://github.com/benjamin-maynard/departureboard-io-api-code-examples) have examples of how this works.

# Fastest and Next Departures

The following set of API Endpoints allow you to look up the fastest and next departures from a UK Train Station, to multiple destinations.

## Get Next Departures by CRS

```shell
curl -X GET "https://api.departureboard.io/api/v2.0/getNextDeparturesByCRS/EUS/?apiKey=000a0a0-00aa-00dc-0000-0a000a00a0aa&filterList=MKC,MAN" -H "accept: */*"
```


> The above command returns JSON structured like this:

```json
{
  "GeneratedAt": "2019-09-10T13:50:25.7418604+01:00",
  "nationalRailResponseTimeMilliseconds": 130,
  "locationName": "London Euston",
  "crs": "EUS",
  "filterLocationName": null,
  "filtercrs": null,
  "filterType": null,
  "nrccMessages": null,
  "platformAvailable": true,
  "areServicesAvailable": null,
  "destination": [
    {
      "crs": "MKC",
      "service": {
        "serviceID": "09uXKDYQdDRjLNRxgajOhQ==",
        "serviceIDUrlSafe": "09uXKDYQdDRjLNRxgajOhQ--",
        "serviceType": "train",
        "isCancelled": null,
        "filterLocationCancelled": null,
        "cancelReason": null,
        "delayReason": null,
        "adhocAlert": null,
        "std": "13:49",
        "etd": "On time",
        "sta": null,
        "eta": null,
        "platform": null,
        "operator": "West Midlands Trains",
        "operatorCode": "LM",
        "isCircularRoute": null,
        "length": 4,
        "detachFront": null,
        "isReverseFormation": null,
        "rsid": null,
        "origin": [
          {
            "locationName": "London Euston",
            "crs": "EUS",
            "via": null,
            "futureChangeTo": null,
            "assocIsCancelled": null
          }
        ],
        "destination": [
          {
            "locationName": "Liverpool Lime Street",
            "crs": "LIV",
            "via": null,
            "futureChangeTo": null,
            "assocIsCancelled": null
          }
        ],
        "currentOrigins": null,
        "currentDestinations": null,
        "subsequentCallingPointsList": [
          {
            "subsequentCallingPoints": [
              {
                "locationName": "Watford Junction",
                "crs": "WFJ",
                "st": "14:03",
                "et": "On time",
                "at": null,
                "length": null
              },
              {
                "locationName": "Liverpool Lime Street",
                "crs": "LIV",
                "st": "17:42",
                "et": "On time",
                "at": null,
                "length": null
              }
            ]
          }
        ]
      }
    },
    {
      "crs": "MAN",
      "service": {
        "serviceID": "j+meN1h8C66roUaXvm+iXg==",
        "serviceIDUrlSafe": "j!meN1h8C66roUaXvm!iXg--",
        "serviceType": "train",
        "isCancelled": null,
        "filterLocationCancelled": null,
        "cancelReason": null,
        "delayReason": null,
        "adhocAlert": null,
        "std": "14:00",
        "etd": "On time",
        "sta": null,
        "eta": null,
        "platform": "1",
        "operator": "Virgin Trains",
        "operatorCode": "VT",
        "isCircularRoute": null,
        "length": null,
        "detachFront": null,
        "isReverseFormation": null,
        "rsid": null,
        "origin": [
          {
            "locationName": "London Euston",
            "crs": "EUS",
            "via": null,
            "futureChangeTo": null,
            "assocIsCancelled": null
          }
        ],
        "destination": [
          {
            "locationName": "Manchester Piccadilly",
            "crs": "MAN",
            "via": null,
            "futureChangeTo": null,
            "assocIsCancelled": null
          }
        ],
        "currentOrigins": null,
        "currentDestinations": null,
        "subsequentCallingPointsList": [
          {
            "subsequentCallingPoints": [
              {
                "locationName": "Stoke-on-Trent",
                "crs": "SOT",
                "st": "15:24",
                "et": "On time",
                "at": null,
                "length": null
              },
              {
                "locationName": "Manchester Piccadilly",
                "crs": "MAN",
                "st": "16:05",
                "et": "On time",
                "at": null,
                "length": null
              }
            ]
          }
        ]
      }
    }
  ]
}
```
getNextDeparturesByCRS is used to get the next service running between two stations. Multiple destinations can be specified. This will typically return a single train service, but will also return a replacement bus or ferry service if in place.

**This will return the next departures for each of the `filterList` stations specified. It may not return the fastest next service. To get the fastest next service use the `getFastestDeparturesByCRS` endpoint.**

### HTTP Request

`GET https://api.departureboard.io/api/v2.0/getNextDeparturesByCRS/{CRS}/`

### URL Parameters

Parameter | Required | Default     | Description
--------- | -------  | ----------- | -----------
CRS       | **true**     | none        | The CRS (Computer Reservation System) for the station you wish to get departure information for, e.g. KGX for London Kings Cross.

### Query Parameters

Parameter | Required | Default     | Description
--------- | -------  | ----------- | -----------
apiKey    | **true**     | none        | The National Rail OpenLDBWS API Key to use for looking up service information. You must [register with National Rail](http://realtime.nationalrail.co.uk/OpenLDBWSRegistration/.) to obtain this key.
filterList | **true**     | none    | The CRS (Computer Reservation System) codes to show departing services to. Up to 20 destination stations can be specified. These should be split by a comma, for example `HAY,EAL,PAD`.
timeOffset | false     | 0          | The time window in minutes to offset the departure information by. For example, a value of 20 will  show the next services departing after 20 minutes.
timeWindow | false     | 120        | The time window to show services for in minutes. For example, a value of 60 will show services departing the station in the next 60 minutes.
serviceDetails | false     | true   | Should the response contain information on the calling points for each service? If set to false, calling points will not be returned.

### Example Queries

* Lookup next departure from London Kings Cross to: Huntingdon, Peterborough and St. Neots:

`curl -X GET "https://api.departureboard.io/api/v2.0/getNextDeparturesByCRS/KGX/?apiKey=0000a0a0-00aa-00dc-0000-0a000a00a0aa&filterList=HUN,PBO,SNO" -H "accept: */*"`

* Lookup next departure from London Kings Cross to: Huntingdon, Peterborough and St. Neots, departing within the next 5 minutes:

`curl -X GET "https://api.departureboard.io/api/v2.0/getNextDeparturesByCRS/KGX/?apiKey=0000a0a0-00aa-00dc-0000-0a000a00a0aa&filterList=HUN,PBO,SNO&timeWindow=5" -H "accept: */*"`

* Lookup next departure from London Kings Cross to: Huntingdon, Peterborough and St. Neots, don't return Calling Points:

`curl -X GET "https://api.departureboard.io/api/v2.0/getNextDeparturesByCRS/KGX/?apiKey=0000a0a0-00aa-00dc-0000-0a000a00a0aa&filterList=HUN,PBO,SNO&serviceDetails=false -H "accept: */*"`

### Notes

When the `serviceDetails` query parameter is set to `true` for `getNextDeparturesByCRS`, only **subsequent calling points** will be returned. This is by design from National Rail.

To get subsequent **and** previous Calling Points for a service, take the `serviceIDUrlSafe` value is provided in the response and use the `getServiceDetailsByID` endpoint to lookup more details.

It is possible to have two sets of `subsequentCallingPoints` returned for a service (when looking up train services that split). Consequently, `subsequentCallingPoints` are returned in the `subsequentCallingPointsList` array. **However**, in 99% of circumstances there are only 1 set of `subsequentCallingPoints` returned. For services that do not split, you can use `subsequentCallingPointsList[0].subsequentCallingPoints` to access the previous calling points. The [Python Code Samples](https://github.com/benjamin-maynard/departureboard-io-api-code-examples) have examples of how this works.


## Get Fastest Departures by CRS

```shell
curl -X GET "https://api.departureboard.io/api/v2.0/getFastestDeparturesByCRS/EUS/?apiKey=000a0a0-00aa-00dc-0000-0a000a00a0aa&filterList=MKC,MAN" -H "accept: */*"
```


> The above command returns JSON structured like this:

```json
{
  "GeneratedAt": "2019-09-10T13:51:58.8967576+01:00",
  "nationalRailResponseTimeMilliseconds": 135,
  "locationName": "London Euston",
  "crs": "EUS",
  "filterLocationName": null,
  "filtercrs": null,
  "filterType": null,
  "nrccMessages": null,
  "platformAvailable": true,
  "areServicesAvailable": null,
  "destination": [
    {
      "crs": "MKC",
      "service": {
        "serviceID": "Zl3D4b/c5lYGfUNhF3Ylrg==",
        "serviceIDUrlSafe": "Zl3D4b_c5lYGfUNhF3Ylrg--",
        "serviceType": "train",
        "isCancelled": null,
        "filterLocationCancelled": null,
        "cancelReason": null,
        "delayReason": null,
        "adhocAlert": null,
        "std": "14:10",
        "etd": "On time",
        "sta": null,
        "eta": null,
        "platform": null,
        "operator": "Virgin Trains",
        "operatorCode": "VT",
        "isCircularRoute": null,
        "length": null,
        "detachFront": null,
        "isReverseFormation": null,
        "rsid": null,
        "origin": [
          {
            "locationName": "London Euston",
            "crs": "EUS",
            "via": null,
            "futureChangeTo": null,
            "assocIsCancelled": null
          }
        ],
        "destination": [
          {
            "locationName": "Chester",
            "crs": "CTR",
            "via": null,
            "futureChangeTo": null,
            "assocIsCancelled": null
          }
        ],
        "currentOrigins": null,
        "currentDestinations": null,
        "subsequentCallingPointsList": [
          {
            "subsequentCallingPoints": [
              {
                "locationName": "Milton Keynes Central",
                "crs": "MKC",
                "st": "14:40",
                "et": "On time",
                "at": null,
                "length": null
              },
              {
                "locationName": "Chester",
                "crs": "CTR",
                "st": "16:08",
                "et": "On time",
                "at": null,
                "length": null
              }
            ]
          }
        ]
      }
    },
    {
      "crs": "MAN",
      "service": {
        "serviceID": "j+meN1h8C66roUaXvm+iXg==",
        "serviceIDUrlSafe": "j!meN1h8C66roUaXvm!iXg--",
        "serviceType": "train",
        "isCancelled": null,
        "filterLocationCancelled": null,
        "cancelReason": null,
        "delayReason": null,
        "adhocAlert": null,
        "std": "14:00",
        "etd": "On time",
        "sta": null,
        "eta": null,
        "platform": "1",
        "operator": "Virgin Trains",
        "operatorCode": "VT",
        "isCircularRoute": null,
        "length": null,
        "detachFront": null,
        "isReverseFormation": null,
        "rsid": null,
        "origin": [
          {
            "locationName": "London Euston",
            "crs": "EUS",
            "via": null,
            "futureChangeTo": null,
            "assocIsCancelled": null
          }
        ],
        "destination": [
          {
            "locationName": "Manchester Piccadilly",
            "crs": "MAN",
            "via": null,
            "futureChangeTo": null,
            "assocIsCancelled": null
          }
        ],
        "currentOrigins": null,
        "currentDestinations": null,
        "subsequentCallingPointsList": [
          {
            "subsequentCallingPoints": [
              {
                "locationName": "Stoke-on-Trent",
                "crs": "SOT",
                "st": "15:24",
                "et": "On time",
                "at": null,
                "length": null
              },
              {
                "locationName": "Manchester Piccadilly",
                "crs": "MAN",
                "st": "16:05",
                "et": "On time",
                "at": null,
                "length": null
              }
            ]
          }
        ]
      }
    }
  ]
}
```
getFastestDeparturesByCRS is used to get the fastest next service running between two stations. Multiple destinations can be specified. This will typically return a single train service, but will also return a replacement bus or ferry service if in place.

### HTTP Request

`GET https://api.departureboard.io/api/v2.0/getFastestDeparturesByCRS/{CRS}/`

### URL Parameters

Parameter | Required | Default     | Description
--------- | -------  | ----------- | -----------
CRS       | **true**     | none        | The CRS (Computer Reservation System) for the station you wish to get departure information for, e.g. KGX for London Kings Cross.

### Query Parameters

Parameter | Required | Default     | Description
--------- | -------  | ----------- | -----------
apiKey    | **true**     | none        | The National Rail OpenLDBWS API Key to use for looking up service information. You must [register with National Rail](http://realtime.nationalrail.co.uk/OpenLDBWSRegistration/.) to obtain this key.
filterList | **true**     | none    | The CRS (Computer Reservation System) codes to show the fastest departing services to. Up to 20 destination stations can be specified. These should be split by a comma, for example `HAY,EAL,PAD`.
timeOffset | false     | 0          | The time window in minutes to offset the departure information by. For example, a value of 20 will show the fastest services departing after 20 minutes.
timeWindow | false     | 120        | The time window to show train services for in minutes. For example, a value of 60 will show the fastest services departing the station in the next 60 minutes.
serviceDetails | false     | true   | Should the response contain information on the calling points for each service? If set to false, calling points will not be returned.

### Example Queries

* Lookup the fastest next departure from London Kings Cross to: Huntingdon, Peterborough and St. Neots:

`curl -X GET "https://api.departureboard.io/api/v2.0/getFastestDeparturesByCRS/KGX/?apiKey=0000a0a0-00aa-00dc-0000-0a000a00a0aa&filterList=HUN,PBO,SNO" -H "accept: */*"`

* Lookup the fastest next departure from London Kings Cross to: Huntingdon, Peterborough and St. Neots, departing within the next 5 minutes:

`curl -X GET "https://api.departureboard.io/api/v2.0/getFastestDeparturesByCRS/KGX/?apiKey=0000a0a0-00aa-00dc-0000-0a000a00a0aa&filterList=HUN,PBO,SNO&timeWindow=5" -H "accept: */*"`

* Lookup the fastest next departure from London Kings Cross to: Huntingdon, Peterborough and St. Neots, don't return Calling Points:

`curl -X GET "https://api.departureboard.io/api/v2.0/getFastestDeparturesByCRS/KGX/?apiKey=0000a0a0-00aa-00dc-0000-0a000a00a0aa&filterList=HUN,PBO,SNO&serviceDetails=false -H "accept: */*"`

### Notes

When the `serviceDetails` query parameter is set to `true` for `getFastestDeparturesByCRS`, only **subsequent calling points** will be returned. This is by design from National Rail.

To get subsequent **and** previous calling points for a service, take the `serviceIDUrlSafe` value is provided in the response and use the `getServiceDetailsByID` endpoint to lookup more details.

It is possible to have two sets of `subsequentCallingPoints` returned for a service (when looking up train services that split). Consequently, `subsequentCallingPoints` are returned in the `subsequentCallingPointsList` array. **However**, in 99% of circumstances there are only 1 set of `subsequentCallingPoints` returned. For services that do not split, you can use `subsequentCallingPointsList[0].subsequentCallingPoints` to access the previous calling points. The [Python Code Samples](https://github.com/benjamin-maynard/departureboard-io-api-code-examples) have examples of how this works.

# Service Information

The following set of API Endpoints allow you to get arrival and departure information for train stations across the UK.

## Get Service Details by ID

```shell
curl -X GET "https://api.departureboard.io/api/v2.0/getServiceDetailsByID/ERZbOJ8wYnOnNgNyaa1Xaw--/?apiKey=0000a0a0-00aa-00dc-0000-0a000a00a0aa" -H "accept: */*"
```


> The above command returns JSON structured like this:

```json
{
  "GeneratedAt": "2019-09-10T13:57:42.8199973+01:00",
  "nationalRailResponseTimeMilliseconds": 91,
  "serviceID": "h+eimq/ol4h/pG9h8Ut8eg==",
  "serviceIDUrlSafe": "h!eimq_ol4h_pG9h8Ut8eg--",
  "locationName": "London Paddington",
  "serviceType": "train",
  "isCancelled": null,
  "cancelReason": null,
  "delayReason": null,
  "overdueMessage": null,
  "adhocAlert": null,
  "sta": null,
  "eta": null,
  "ata": null,
  "std": "13:50",
  "etd": null,
  "atd": "On time",
  "length": null,
  "detachFront": null,
  "isReverseFormation": null,
  "platform": null,
  "operator": "Great Western Railway",
  "operatorCode": "GW",
  "rsid": null,
  "subsequentCallingPointsList": [
    {
      "subsequentCallingPoints": [
        {
          "locationName": "Slough",
          "crs": "SLO",
          "st": "14:05",
          "et": "On time",
          "at": null,
          "length": null
        },
        {
          "locationName": "Oxford",
          "crs": "OXF",
          "st": "14:48",
          "et": "On time",
          "at": null,
          "length": null
        }
      ]
    }
  ],
  "previousCallingPointsList": null
}
```

getServiceDetailsByID is used to get information on a service, by the Service ID. This will typically return a train service, but will also return a bus and ferry services.

The Service ID must be provided in the `serviceIDUrlSafe` format that is provided in the response for Arrival and Departure Boards. If required, these can be manually converted replacing:

* `=` with `-`
* `/` with `_`
* `+` with `!`

For example, a National Rail Service ID of `qsec4O8LW7k8ITcOt/ir4Q==` becomes `qsec4O8LW7k8ITcOt_ir4Q--`. However, you should be using the value provided in the Arrival and Departure Board responses.

A service ID is specific to a station, and can only be looked up for a short time after a train/bus/ferry arrives at, or departs from a station. This is a National Rail limitation.

### HTTP Request

`GET https://api.departureboard.io/api/v2.0/getServiceDetailsByID/{serviceID}/`

### URL Parameters

Parameter | Required | Default     | Description
--------- | -------  | ----------- | -----------
serviceID | **true**     | none        | The Service ID for the Train Service you wish to look up in the URL Safe format. For example, `qsec4O8LW7k8ITcOt_ir4Q--`.


### Query Parameters

Parameter | Required | Default     | Description
--------- | -------  | ----------- | -----------
apiKey    | **true**     | none        | The National Rail OpenLDBWS API Key to use for looking up service information. You must [register with National Rail](http://realtime.nationalrail.co.uk/OpenLDBWSRegistration/.) to obtain this key.

### Example Queries

* Lookup a service with the ID `qsec4O8LW7k8ITcOt_ir4Q--`:

`curl -X GET "http://api.departureboard.io/api/v2.0/getServiceDetailsByID/0gN6ZPyTiOC1WfjQv7iNKQ--/?apiKey=0000a0a0-00aa-00dc-0000-0a000a00a0aa" -H "accept: */*"`

### Notes

It is possible to have two sets of `previousCallingPoints` or `subsequentCallingPoints` returned for a service (when looking up train services that split). Consequently, `previousCallingPoints` and `subsequentCallingPoints` are returned in the `previousCallingPointsList` and `subsequentCallingPointsList` arrays. **However**, in 99% of circumstances there are only 1 set of `previousCallingPoints` and `subsequentCallingPoints` returned. For services that do not split, you can use `previousCallingPointsList[0].previousCallingPoints` or `subsequentCallingPointsList[0].subsequentCallingPoints` to access the calling points. The [Python Code Samples](https://github.com/benjamin-maynard/departureboard-io-api-code-examples) have examples of how this works.


# Station Lookups

The following set of API endpoints can be used to get information about Train Stations across the United Kingdom.

The National Rail SOAP API does not have the capability to lookup information on Train Stations. The following endpoints are custom designed and served directly from the departureboard.io API. The data is generated from static data feeds from National Rail, which are automatically parsed and updated each night by the departureboard.io API at 00:30 United Kingdom time.

**You do not need to specify your National Rail API Key to use these feeds. Responses are served directly by the departureboard.io API**

## Get Train Station Basic Info

```shell
curl -X GET "https://api.departureboard.io/api/v2.0/getStationBasicInfo/?station=London%20W" -H "accept: */*"
```


> The above command returns JSON structured like this:

```json
[
  {
    "crsCode": "WAE",
    "stationName": "London Waterloo East"
  },
  {
    "crsCode": "WAT",
    "stationName": "London Waterloo"
  }
]
```

getStationBasicInfo is used to look up a Train Station by CRS Code, or by name.

The most common use for this endpoint is for converting between CRS Codes and Train Station names. For example, displaying friendly names like "London Paddington" instead of "PAD" in your application. You can also use this endpoint in AJAX powered search boxes, to allow your users to search for stations by name.

getTrainStationBasicDetails returns a JSON Array of Objects, containing each stations `crsCode` and `stationName`.

The optional `stationCRS` parameter can also be used to lookup basic information on a specific train station by its CRS Code. This changes the response of the endpoint to a single JSON Object instead of a JSON Array. This can be used in applications to quickly convert CRS Codes to station names.


### HTTP Request

`GET https://api.departureboard.io/api/v2.0/getStationBasicInfo/`


### Query Parameters

Parameter | Required | Default     | Description
--------- | -------  | ----------- | -----------
station   | false     | none       | The string you wish to use to search for stations. This can be a CRS Code, such as `PAD`, a partial train station name such as `Wate` or a full train station name such as `London Waterloo`. If this query parameter is omitted, then all train stations in the UK will be returned. This parameter cannot be used alongside `stationCRS`. You must use one or the other.<br><br>**This query parameter must be URL encoded. For example `London Paddington` becomes `London%20Paddington`, and `Hayes & Harlington` becomes `Hayes%20%26%20Harlington`.**
stationCRS| false     | none       | Used to lookup a specific train station by CRS Code. This parameter should not be used for searching, it must be a valid and complete CRS code. **This parameter will change the response to return a single JSON object instead of an array**. This parameter cannot be used alongside `station`. You must use one or the other.

### Example Queries

* Lookup all UK Train Stations with `London` in their name:

`curl -X GET "https://api.departureboard.io/api/v2.0/getStationBasicInfo/?station=London" -H "accept: */*"`

* Return a list of all UK Train Stations:

`curl -X GET "https://api.departureboard.io/api/v2.0/getStationBasicInfo/" -H "accept: */*"`

* Lookup London Paddington:

`curl -X GET "https://api.departureboard.io/api/v2.0/getStationBasicInfo/station=London%20Paddington" -H "accept: */*"`

* Lookup London Kings Cross by CRS Code Only:

`curl -X GET "https://api.departureboard.io/api/v2.0/getStationBasicInfo/stationCRS=KGX" -H "accept: */*"`

## Get Detailed Train Station Info By CRS


```shell
curl -X GET "https://api.departureboard.io/api/v2.0/getStationDetailsByCRS/KGX/" -H "accept: */*"
```


> The above command returns a JSON response. The response contains a lot of detail so a preview of the response is not included.



getStationDetailsByCRS is a really powerful, custom designed API Endpoint that allows the retrival of extremely detailed information about a Train Station.

It returns a wealth of information, such as:

* Opening Times
* Address
* Staffing Schedules
* Accessibility Information
* +100's of other attributes


### HTTP Request

`GET https://api.departureboard.io/api/v2.0/getStationDetailsByCRS/{CRS}`


### Query Parameters

Parameter | Required | Default     | Description
--------- | -------  | ----------- | -----------
CRS       | **true** | none       | The CRS (Computer Reservation System) for the Station you wish to get detailed Train Station information for, e.g. KGX for London Kings Cross.

### Example Queries

* Lookup detailed Train Station information for London Kings Cross:

`curl -X GET "https://api.departureboard.io/api/v2.0/getStationDetailsByCRS/KGX/" -H "accept: */*"`

# HTTP Response Codes

The departureboard.io API uses the below error codes:


Code | Meaning
---------- | -------
200 | **OK**: The API Request was successful and returned data.
400 | **Bad Request**: Your request was not valid. <br><br>The departureboard.io API performs validation of queries to make sure they make sense and will return a valid response from National Rail. Bad queries made directly to the National Rail API return a generic `500 Internal Server Error` response which is difficult to troubleshoot.<br><br>By validating requests the departureboard.io API can provide improved error messages to help with troubleshooting, and prevent bad queries from ever reaching National Rail.
401 | **Unauthorized**: Your National Rail API key is wrong.<br><br>This error indicates that your National Rail API Key is wrong. It means National Rail returned a `401 Unauthorized` response to the departureboard.io API.<br><br>You should double check your API Key is valid. Newly generated API Keys can take up to 15 minutes to become active.
429 | **Too Many Requests**: You have exceeded the rate limit. <br><br>[Get in touch](mailto:benjamin@maynard.io) to arrange an exception, or slow down your requests.
503 | **Service Unavailable**: The National Rail upstream API is experiencing issues.<br><br>This error is returned when the departureboard.io API detects issues with the upstream National Rail API. The departureboard.io API is functioning but cannot return a result as the National Rail API is down.
500 | **Internal Server Error**: There was an unknown error returning the response.<br><br>This error is served when an unknown error is encountered returning your response. This is often due to upstream problems with National Rail, for example them returning a bad response or malformed data. In rare circumstances it can be caused by a bad request that was not caught in the departureboard.io validation.
