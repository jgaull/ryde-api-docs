# Ryde Public API
Ryde is an app that utilizes low power, always on location tracking to record all of a user's travel. User's then categorize their trips as either "by bike" or "not by bike". The Ryde public API gives developers access to trips marked as "by bike".

## Registering A Client
An API Client is an application created by a developer that is designed to consume data provided by the Ryde Public API. At this time API credentials are provided on an as-needed basis. If you're interested in integrating with the Ryde Public API [contact us](http://www.modeo.co/contact).

## Authentication
The Ryde Public API utilizes the OAuth 2.0 protocol for authentication. OAuth 2.0 is a widely accepted standard for web security.

### Obtaining and access_token
In order to receive an access token you must do the following:

1. Direct the user to our authorization url.
	* If the user is not logged in, they will be asked to log in.
	* The user will be asked if they would like to grant your application access to their Ryde data.
2. The Ryde server will redirect the user to a URI of your choice. Take the provided `code` parameter and exchange it for an `access_token` by POSTing the `code` to our `access_token` url.

### Notes
At this time the Ryde Public API only supports the Server-side (Explicit) Flow. This method is appropriate for any API client that utilizes its own backend. The Implicit Flow, which is better suited to applications that do not have a server, is on the roadmap for a future release.

### Important
Do not assume your `access_token` is valid forever. Even though our access tokens do not specify an expiration time, your app should handle the case that either the user revokes access, or the Ryde Public API expires the token after some period of time. If the token is no longer valid, API responses will contain an `error_type=OAuthAccessTokenException`. In this case you will need to re-authenticate the user to obtain a new token.

### Server-Side (Explicit) Flow
#### 1) Direct your user to our authorization URL
```
https://modeo-ryde-prod.herokuapp.com/api/authorize?response_type=code&client_id=<YOUR CLIENT ID>&redirect_uri=<YOUR REDIRECT URI>
```
This will present the user with a login screen and then an authorization dialog. The user can choose to either allow or deny access to your app at this point.

#### 2) Receive the redirect from the Ryde server
After the user chooses to allow or deny your app we direct the user to the `redirect_uri` you supplied in step 1. If the user chose to allow your app the request will contain a `code` query parameter:
```
http://your-redirect-uri?code=CODE
```
If the user chooses to deny access you will receive an error:
```
http://your-redirect-uri?error=access_denied
```

Please not that the `redirect_uri` you supply must match a redirect uri we have in our database for your app. If the `redirect_uri` does not match you will receive an error. Please [contact us](http://www.modeo.co/contact) if you need to add `redirect_uris` to your app's configuration.
```
http://your-redirect-uri?error=invalid_redirect_uri
```

#### 3) Request the access_token
Now you need to exchange the `code` you received in step 2 for an `access_token`. In order to make this exchange you have to POST the `code`, along with some app identification parameters, to our `/api/authorize/token` endpoint. These are the required parameters:
* __client_id:__ Your client id (provided by Modeo upon request)
* __client_secret:__ Your client secret (provided by Modeo along with your client id)
* __grant_type:__ `authorization_code` is currently the only supported value
* __redirect_uri:__ The same `redirect_uri` you used in step 1
* __code:__  The `code` you received during step 2

Here is an example curl request to obtain a token:
```
curl -d 'client_id=CLIENT_ID' \
    -d 'client_secret=CLIENT_SECRET' \
    -d 'grant_type=authorization_code' \
    -d 'redirect_uri=REDIRECT_URI' \
    -d 'code=CODE' \
    https://modeo-ryde-prod.herokuapp.com/api/authorize/token
```

If successful, this call will return an `access_token` that you can use to make authenticated calls to the API.
```
{
	"access_token": "...",
	"token_type": "Bearer"
}
```
You are now ready to make requests to the Ryde Public API!

## Making Requests
The base URL for the Ryde production server is `https://modeo-ryde-prod.herokuapp.com/`.

All API enpoints can be reached using the following URL format:
```
https://modeo-ryde-prod.herokuapp.com/api/json/:endpoint
```

### Success Response Format
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
### Error Response Format
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
Only one for now.

#### Recent Rides
The `recentRides` endpoint is used to fetch a user's rides. The request will succeed as long as the user has no unsaved trips. If the user does have unsaved trips the server will return an error with code 10. This is the only request that will return code 10 so you can use the code to identify this error case.

##### Parameters
`auth_token`: Required. Must be passed with every request to the Ryde Public API. Obtain an `auth_token` through the OAuth2 flow.

`date`: Required. The request will return all Rides where the `arrivalTime` is after `date`. This parameter will accept any [ISO 8601 formatted date](https://momentjs.com/docs/#/parsing/string/).

##### Example Request
```https://modeo-ryde-prod.herokuapp.com/api/json/recentRides?access_token=57NX2540PKi6U6QJnqmTEkAESWnOeu7w&date=2017-03-16T21:32:21.852Z
```

##### Example Response
Rides are sorted from newest to oldest. Please note the `mostRecentRide` timestamp. It contains the `arrivalTime` of the most recent ride in the list of `rides`. Store this timestamp in your database for use next time you call `recentRides`.
```
{
	"meta": {
		"code": 200
	},
	"data": {
		"mostRecentRide": "2017-03-16T21:33:19.163Z"
		"rides": [{
			"totalPoints": 196,
			"pointSources": [{ //a breakdown of the user's score for this ride
				"type": 0, //At this time type 0 is the only supported type.
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
			"approvedAt": "2017-03-16T21:35:03.487Z", //the time the user approved their trip
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
			"creator": {
				"timeZone": "America/Los_Angeles",
				"totalPoints": 523,
				"totalDistance": 13389, //in meters
				"numRides": 3,
				"totalDuration": 75.28, //in seconds
				"name": "Test1",
				"createdAt": "2017-03-16T21:05:26.114Z",
				"updatedAt": "2017-03-16T21:35:36.205Z",
				"mostRecentRide": "2017-03-16T21:32:42.327Z",
				"website": "",
				"bio": "This account has a couple of rides.",
				"objectId": "sAdnGOFL3G" //unique identifier for the user
			},
			"distance": 5326, //in meters
			"duration": 28.343, //in seconds
			"objectId": "Vgw4Msc7dW" //unique identifier for the ride
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

### Revoke OAuth Token
If you are testing the OAuth flow it may become necessary to revoke a previously granted OAuth Token. Use the following endpoint:
```
https://ryde-dev.herokuapp.com/api/revoke?access_token=ACCESS_TOKEN //development
https://modeo-ryde-prod.herokuapp.com/api/revoke?access_token=ACCESS_TOKEN //production
```



