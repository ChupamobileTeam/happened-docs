happened-docs
=============

This is the official documentation for the happened server.

**Is** a simple daemon for easy events aggregation.

Happened **Accepts** request with POST method, 
using JWT for authorization.

## Client API

### A. Single Event 

#### [POST] /smtg

##### Request Headers: 

`Content-Type: application/json`

`Authorization: BEARER eyJhbGciOiJIUzI1NiJ9.eyJhcGlfaWQiOiJkZWJ1Z2dlciIsImV4cCI6MTQ1MTYwNjQwMCwiYmhhIjoiYzIzNTQzZmQ2OGZlNmM4YjgyNjkxYWIyYjQwMmY0MjMifQ.yC0qeyxTy_QfMBhoHdAq68KIDOaqFCJNHf6g9HBD4z8`
 
  more about how to create this value [in the Authorization chapter](#authorization).

##### Request content:

A `something` object:

``` json
{"api_id":"A124","at":"2011-04-10T20:09:31Z","os":"ANDROID","ver":"2.1","what":"admov.adcall","value":""}
```

| Name       |     Type    | Description |
| ---------- | ----------- | ----------- |
| "api_id" (mandatory)| string      | The Api Id. |
| "at" (mandatory)| string      | When the event happened, this is a timestamp in ISO 8601 format: YYYY-MM-DDTHH:MM:SSZ. |
| "os" (mandatory)| string      | Operative System used by the client `AND`, `IOS`, `WIN` |
| "ver" (mandatory)| float       | Version of the OS used `2.1`, `4.1`, `9` |
| "what" (mandatory)| string      | This is the name of the event eg. vendorClass.function eg. `admob.setMainMenuBannerId` |
| "value" (optional) | string | Optional could be the value of the function called eg. `2` |
 
The hash of the content is used by the token the hash of that token is `c23543fd68fe6c8b82691ab2b402f423`
 
### B. Multiple Events, array

#### [POST] /api/smtgs

##### Request Headers:

Same of `[POST] /smtg`

##### Request content:

Is a JSON array of the `something` object:

example:

``` json
[
    {
        "api_id": "A124",
        "at": "2011-04-10T20:09:31Z",
        "oS": "ANDROID",
        "ver": "2.1",
        "what": "admov.adcall"
    },
    {
       "api_id": "A125",
        "at": "2013-04-10T20:09:31Z",
        "oS": "IOS",
        "ver": "8",
        "what": "flurry.xyz"
    }
]
```

## Authorization

In order to authorize the request, the client should have a `apiId` and a `apiKey`

for debugging purpose the use as apiId `debugger` and as apiKey `secret`

###  JWT TOKEN

The JWT token is composed by 3 parts joined by ".", the first is the **header** about the algorithm used, the second is the **claims** called also payload, the third part is the **signed** that certificate the first two.

The token will be composed by `base64(header).base64(claims).signed`

#### 1. token.Header

Nothing to know on the header is pretty standard and fixed, rarely change.

``` json
{
  "alg": "HS256"
}
```

|   Attribute     |     Type    | Description |
| ----------      | ----------- | ----------- |
| "alg" (mandatory) | String	 |Algorithm. specifies the algorithm used to sign the token. In atlassian-connect version 1.0 we support the HMAC SHA-256 algorithm, which the JWT specification identifies using the string "HS256". |

##### Header Encoded

base64UrlEncode(header) = `eyJhbGciOiJIUzI1NiJ9`

#### 2. token.Claims

```json
{
    "api_id": "debugger",
    "exp": 1451606400,
    "bha": "c23543fd68fe6c8b82691ab2b402f423"
}
```

|   Attribute     |     Type    | Description |
| ----------      | ----------- | ----------- |
| api_id (mandatory) | String	| the ApiID of the Request. It is used it to identify the application making the call. |
| exp (mandatory) | 	Long | Expiration time. It contains the UTC Unix time after which you should no longer accept this token. It must not be after 15 min after the issued time. Only for debugging purpose the 15 mins limit has been removed.|
| bha (mandatory) | String | The hash of the body content, using the Md5 algorithm eg. `md5(request.body)` |

##### Claims Encoded

base64UrlEncode(claims) = `eyJhcGlfaWQiOiJkZWJ1Z2dlciIsImV4cCI6MTQ1MTYwNjQwMCwiYmhhIjoiYzIzNTQzZmQ2OGZlNmM4YjgyNjkxYWIyYjQwMmY0MjMifQ`

#### 3. The signed part

The signed part is about signing the header and the claim with the shared secret `api_key`.

##### A Pseudo Algorithm

Given `api_id` and a `api_key` and a `Request.Body` we can generate the Signed Part.

##### Forging the `token`

```
header = {
    "alg":"HS256"
}

claims = {
    "api_id": "debugger",
    "exp": 1451606400,
    "bha": "c23543fd68fe6c8b82691ab2b402f423"
}

signed = HMACSHA256(
    base64UrlEncode(header)+"."+base64UrlEncode(claims),
    "secret"
)

token = base64UrlEncode(header)+"."+base64UrlEncode(claims)+"."+signed
```
The token now is:
  `eyJhbGciOiJIUzI1NiJ9.eyJhcGlfaWQiOiJkZWJ1Z2dlciIsImV4cCI6MTQ1MTYwNjQwMCwiYmhhIjoiYzIzNTQzZmQ2OGZlNmM4YjgyNjkxYWIyYjQwMmY0MjMifQ.yC0qeyxTy_QfMBhoHdAq68KIDOaqFCJNHf6g9HBD4z84`

Then add to the the request 

`Request.addHeader("Authorization", "BEARER "+token)`

## Make a request

The server will return 201

``` bash
curl -X POST  http://happened.herokuapp.com/smtg -H 'Authorization: BEARER eyJhbGciOiJIUzI1NiJ9.eyJhcGlfaWQiOiJkZWJ1Z2dlciIsImV4cCI6MTQ1MTYwNjQwMCwiYmhhIjoiYzIzNTQzZmQ2OGZlNmM4YjgyNjkxYWIyYjQwMmY0MjMifQ.yC0qeyxTy_QfMBhoHdAq68KIDOaqFCJNHf6g9HBD4z8' -H "Content-Type: application/json" -d '{"api_id":"A124","at":"2011-04-10T20:09:31Z","os":"ANDROID","ver":"2.1","what":"admov.adcall","value":""}'
```

### Debug 

#### [GET] /dump

Reaching /dump route you can have the latest events.

### Tools 

1. http://jwt.io/
2. http://www.timestampgenerator.com/1451606400/#result

### Related articles

1. https://developer.atlassian.com/static/connect/docs/concepts/understanding-jwt.html
2. https://developers.google.com/wallet/instant-buy/about-jwts
3. http://docs.aws.amazon.com/AmazonS3/latest/dev/RESTAuthentication.html#ConstructingTheAuthenticationHeader
