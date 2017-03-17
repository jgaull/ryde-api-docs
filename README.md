# Ryde Public API
Ryde is an app that utilzes low power, always on location tracking to record all of a user's travel. User's then categorize their trips as either "by bike" or "not by bike". The Ryde public API gives developers access to trips marked as "by bike".

## Registering A Client
An API Client is an application created by a developer that is designed to consume data provided by the Ryde Public API. At this time API credentials are provided on an as-needed basis. If you're interested in integrating with the Ryde Public API [contact us](http://www.modeo.co/contact).

## Authentication
The Ryde Public API will use OAuth2 for authentication. More details to come.a

## Making Requests
At this time the Ryde development server hosts the only available endpoint. The base url for the ryde development server is `https://ryde-dev.herokuapp.com/`.

All API enpoints can be reached using the following URL format:
`https://ryde-dev.herokuapp.com/api/json/:endpoint`

### Responses
Every response follows the following structure
```
{
	"meta": {
		//information about the request is here
	},
	"data": {
		//the stuff you're looking for is in here
	}
}
```
An error response looks like this
```
{
	"meta": {
		"error_type": "Exception", //All errors are of type Exception at this time
		"code": 1,
		"error_message": "Human readable message"
	}
}
```

### Endpoints
The Ryde API provides the following endpoint.

#### Recent Rides
The `recentRides` endpoint is used to fetch rides that have been approved by the user and marked as "by bike". The request will succeed as long as the user has no unsaved trips. If the user does have unsaved trips the server will return an error with code 10. This is the only request that will return code 10 so you can use the code to determine how to message the user.

##### Parameters
`auth_token`: Required. Must be passed with every request to the Ryde Public API. Retreive an `auth_token` through the OAuth2 flow.

`date`: Required. The request will return all Rides where the `arrivalTime` is after `date`. This parameter will accept any [ISO 8601 formatted date](https://momentjs.com/docs/#/parsing/string/).

##### Example Request
`https://ryde-dev.herokuapp.com/api/json/recentRides?access_token=<a token>&date=2016-10-14T20:40:30.973Z`

##### Example Response
Please note the `mostRecentRide` timestamp. It contains the `arrivalTime` of the most recent ride in the list of rides. Store this timestamp in your database for use next time you call `recentRides`.
```
{
	"meta": {
		"code": 200
	},
	"data": {
		"mostRecentRide": "2017-03-16T21:33:19.163Z"
		"rides": [{
			"pointSources": [{
				"type": 0,
				"key": "tripPoints",
				"value": 100
			}, {
				"type": 0,
				"key": "mileagePoints",
				"value": 61
			}, {
				"type": 0,
				"key": "newDestination",
				"value": 35
			}],
			"arrivalTime": "2017-03-16T21:33:19.163Z",
			"approvedAt": "2017-03-16T21:35:03.487Z",
			"startDestination": {
				"coordinate": {
					"lat": 37.7737866,
					"long": -122.3930995
				},
				"name": "Mission Creek Park",
				"createdAt": "2017-03-16T21:35:03.233Z",
				"updatedAt": "2017-03-16T21:35:35.978Z",
				"objectId": "ZPHMoGjdZT"
			},
			"endDestination": {
				"coordinate": {
					"lat": 37.75349303242875,
					"long": -122.4215458251955
				},
				"name": "Home",
				"createdAt": "2017-03-16T21:33:49.324Z",
				"updatedAt": "2017-03-16T21:35:03.312Z",
				"objectId": "SifMFMN0PG"
			},
			"totalPoints": 196,
			"creator": {
				"timeZone": "America/Los_Angeles",
				"totalPoints": 523,
				"totalDistance": 13389,
				"numRides": 3,
				"totalDuration": 75.28,
				"name": "Test1",
				"createdAt": "2017-03-16T21:05:26.114Z",
				"updatedAt": "2017-03-16T21:35:36.205Z",
				"mostRecentRide": "2017-03-16T21:32:42.327Z",
				"website": "",
				"bio": "This account has a couple of rides.",
				"objectId": "sAdnGOFL3G"
			},
			"distance": 5326,
			"duration": 28.343,
			"objectId": "Vgw4Msc7dW"
		}]
	}
}
```

##### Example Error
The `recentRides` endpoint will return an error if it is called and the user has unsaved trips. This use-case is very common so make sure it is handled.
```
{
	"meta": {
		"error_type": "Exception",
		"code": 10, //this is the only case where 10 is the error code. Use this code to handle the error.
		"error_message": "You have unsaved trips. Please launch the Ryde app and save or delete your unsaved trips."
	}
}
```

## Testing
For testing against the development server please use the following authorization tokens:

### test1 User Account
**Auth Token:** `57NX2540PKi6U6QJnqmTEkAESWnOeu7w`
**Oldest Ride Timestamp:** `2017-03-16T21:33:49.506Z`
**Most Recent Ride Timestamp:** `2017-03-16T21:35:03.487Z`
**Number of Unsaved Trips:** `0`
**Total Number of Rides:** `3`
**Example Request:** `https://ryde-dev.herokuapp.com/api/json/recentRides?access_token=57NX2540PKi6U6QJnqmTEkAESWnOeu7w&date=2017-03-16T21:32:21.852Z`
**Notes:** The above request will return 2 of the user's 3 rides. Change the timestamp to an earlier time to return all 3.

### test2 User Account
**Auth Token:** `394L7Sh6cZuA5MP46s9wQQ3eVoTLz7Tm`
**Oldest Ride Timestamp:** `2017-03-16T21:32:21.852Z`
**Most Recent Ride Timestamp:** `2017-03-16T21:23:48.661Z`
**Number of Unsaved Trips:** `1`
**Total Number of Rides:** `2`
**Example Request:** `https://ryde-dev.herokuapp.com/api/json/recentRides?access_token=394L7Sh6cZuA5MP46s9wQQ3eVoTLz7Tm&date=2017-03-16T21:32:21.852Z`
**Notes:** The above request will always return an error because the user has 1 unsaved trip.

