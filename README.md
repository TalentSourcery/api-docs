# API

## 1. OVERVIEW

TalentSourcery has a RESTful API. The API is consumed by the web app, and also by users who interact directly with the API via integration.

## 2. AUTHENTICATION

There are 2 methods of authentication in the API:

- Signing in to get a Jason Web Token - JWT. This method is intended for web app users.
- Make requests directly to the API with the API token. This method is intended for users that have an integration with our API.

## 3. RATE LIMIT

Users in the 'Pay ahead' billing plan are limited by the amount of credits they have to make searches.
Users in the 'Pay as you go' billing plan have no limit.
The number of requests per second and per hour our API can handle depends on the source of data.
- The GitHub search can handle 85 requests per hour, and around 10 requests per second, for all users combined. 
- The LinkedIn search can handle 500 requests per hour, and 10 requests per second, for all users combined.
These limits will be expanded as demand increases for the search endpoints.

## 4. RESPONSE

All API endpoints respond with a JSON object with the following properties:
  
  - success<br> 
    Type: Boolean<br>
    Description: True if there was no error, false if there was an error
  - data<br>
    Type: Object or Null<br>
    Description: An object containing data if there was no error. The data changes from endpoint to endpoint. If there was an error, this field will be null
  - error<br>
    Type: Null or Object
    Description: Null if there was no error. If there was an error, an object with statusCode and message properties

Example error response:

```
  {
    "success": false,
    "data": null,
    "error": {
      "statusCode": 500,
      "message": "Validation failed"
    }
  }
```

## 4. USAGE

All API endpoints work with JavaScript Object Notation - JSON, data.
Hence, all requests must have the content type header set for JSON apps:

Headers:
```
  Content-Type: application/json
```

The base URL of the API is:
```
  https://api.talentsourcery.io/v1
```

## 5. ROUTES

`POST /user/signin`

`GET /user`

`GET /search`

`GET /searches`

`POST /calibrate/github`

Description:

This endpoint allows an user to preview a GitHub search before actually executing it. Does not spend search credits.

Authentication:
  - JWT for web app users
  - API token for integrated users

Headers:
  - Content-Type: application/json
  - Authorization: Bearer {JWT OR API TOKEN}

Body:
  - searchForm
    - searchName<br>
      Type: String<br>
      Description: Name of the search
    - location<br>
      Type: String<br>
      Description: Location of the sourced talent
    - language<br>
      Type: String<br>
      Main programming language of the sourced talent
    - repos<br>
      Type: String<br>
      Description: Number of public repositories of the sourced talent. Must have a specific format Examples: 1..100, >1, <100
    - followers<br>
      Type: String<br>
      Number of followers of the sourced talent. Must have a specific format. Examples: 1..100, >1, <100

Example body:
```
  {
    "searchForm": {
      "searchName": "Test",
      "location": "São Paulo",
      "language": "JavaScript",
      "repos": "1..50",
      "followers": ">1"
    }
  }
```

Query:<br>
  None

Response:
  - searchString<br>
    Type: String<br>
    Description: Search string created for the user based on his search form. This search string is passed to the GitHub internal search API to get the user profiles of interest
  - results<br>
    Type: Number<br>
    Description: Number of results in the GitHub internal search API.
  - url
    Type: String<br>
    Description: URL of the search results in the GitHub internal search API.

Example response:
```
  {
    "success": true,
    "data": {
      "searchString": "location:\"São Paulo\" language:JavaScript repos:1..50 followers:>1",
      "results": 5054,
      "url": "https://github.com/search?q=location%3A%22S%C3%A3o%20Paulo%22%20language%3AJavaScript%20repos%3A1..50%20followers%3A%3E1&type=Users"
    },
    "error": null
  }
```

`POST /search/github`

Description:

This endpoint allows an user to source talent from GitHub. Spends 1 search credit.

Authentication:
  - JWT for web app users
  - API token for integrated users

Headers:
  - Content-Type: application/json
  - Authorization: Bearer {JWT OR API TOKEN}

Body:
  - searchForm
    - searchName<br>
      Type: String<br>
      Description: Name of the search
    - location<br>
      Type: String<br>
      Description: Location of the sourced talent
    - language<br>
      Type: String<br>
      Main programming language of the sourced talent
    - repos<br>
      Type: String<br>
      Description: Number of public repositories of the sourced talent. Must have a specific format Examples: 1..100, >1, <100
    - followers<br>
      Type: String<br>
      Number of followers of the sourced talent. Must have a specific format. Examples: 1..100, >1, <100
    - sort<br>
      Type: String<br>
      Description: Method used to sort the sourced talent. Will not affect the results, only how they are ordered. Must be one of:
        - Best match
        - Most followers
        - Fewest followers
        - Most recently joined
        - Least recently joined
        - Most repositories
        - Fewest repositories

Example body:
```
  {
    "searchForm": {
      "searchName": "Test",
      "location": "São Paulo",
      "language": "JavaScript",
      "repos": "1..50",
      "followers": ">1"
      "sort": "Best match"
    }
  }
```

Query:<br>
  None

Response:
  - userName: String,
  - email: String,
  - source: String,
  - searchName: String,
  - searchString: String,
  - searchForm: Object,
  - numberOfPeopleSourced: Number,
  - talentInfo
    - csv: String,
    - json: Object,
  - billing
    - plan: String,
    - paid: Boolean,
  - createdAt
    - readableDate: String,
    - isoDate: String,

Example response:
```
{
	"success": true,
	"data": {
		"userName": "User",
		"email": "user@domain.com",
		"source": "GitHub",
		"searchName": "Test",
		"searchString": "location:\"São Paulo\" language:JavaScript repos:1..50 followers:>1",
		"searchForm": {
			"searchName": "Test",
			"location": "São Paulo",
			"language": "JavaScript",
			"repos": "1..50",
			"followers": ">1",
			"sort": "Most recently joined"
		},
		"talentInfo": {
			"csv": "Name,Email in profile,Email in commits,Hireable,Handle,GitHub profile,Company,Location,Bio,Website,Public repositories,Public gists,Followers,Following,Twitter,Date of creation (yyyy-mm-dd),\nMateus V. Farias,    me@mateusfarias.com.br,    -,    true,    fariasmateuss,    https://github.com/fariasmateuss,    -,    São Paulo; SP; Brazil,    I'm Software engineer based in Brazil. Frontend dev is my focus; but always up for learning new things.,    mateusfarias.com.br,    38,    3,    974,    1816,    fariasmateuss,    2019-09-23\n",
			"json": [
				{
					"name": "Mateus V. Farias",
					"emailInProfile": "me@mateusfarias.com.br",
					"emailInCommits": "-",
					"hireable": true,
					"handle": "fariasmateuss",
					"url": "https://github.com/fariasmateuss",
					"company": "-",
					"location": "São Paulo; SP; Brazil",
					"bio": "I'm Software engineer based in Brazil. Frontend dev is my focus; but always up for learning new things.\r\n",
					"website": "mateusfarias.com.br",
					"publicRepos": 38,
					"publicGists": 3,
					"followers": 974,
					"following": 1816,
					"twitter": "fariasmateuss",
					"createdAt": "2019-09-23"
				},
			]
		},
		"billing": {
			"plan": "Pay ahead",
			"paid": true
		},
		"createdAt": {
			"readableDate": "Tue, 31 Jan 2023 22:00:29 GMT",
			"isoDate": "2023-01-31T22:00:29.001Z"
		},
		"_id": "63d98f7de1f384dfe6c126aa",
		"numberOfPeopleSourced": 1,
		"__v": 0
	},
	"error": null
}
  
```
