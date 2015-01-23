happened-docs
=============

This is the official documentation for the happened server.

**Is** a simple daemon for easy events aggregation.

Happened **Accepts** request with POST method.

## Client API

### [POST] /api/smtg

#### Request Headers: 

`Content-Type: application/json`

`Authorization: BEARER eyJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJkZWJ1Z2dlciIsImV4cCI6MTQ1MTYwNjQwMCwiYmhhIjoiZDQxZDhjZDk4ZjAwYjIwNGU5ODAwOTk4ZWNmODQyN2UifQ.jvhMSPJBM9zG1QBAvlW3ICOUNsBZ0J-NL5Sy9q_maI4`
 
  more about how to create this value [in the Authorization chapter](#authorization).

#### Request content:

``` json
{
    "api_id": "debugger",
    "os": "AND",
    "osvers": "2.1",
    "what": "admob.setMainMenuBannerId",
    "value": "",
    "at": "2011-04-22T13:33:48Z"
}
```

| Name       |     Type    | Description |
| ---------- | ----------- | ----------- |
| "api_id" (mandatory)| string      | The Api Id. |
| "at" (mandatory)| string      | When the event happened, this is a timestamp in ISO 8601 format: YYYY-MM-DDTHH:MM:SSZ. |
| "os" (mandatory)| string      | Operative System used by the client `AND`, `IOS`, `WIN` |
| "ver" (mandatory)| float       | Version of the OS used `2.1`, `4.1`, `9` |
| "what" (mandatory)| string      | This is the name of the event eg. vendorClass.function eg. `admob.setMainMenuBannerId` |
| "value" (optional) | string | Optional could be the value of the function called eg. `2` |
 
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

``` json
{
    "api_id": "debugger",
    "exp": 1451606400,
    "bha": "d41d8cd98f00b204e9800998ecf8427e"
}
```

|   Attribute     |     Type    | Description |
| ----------      | ----------- | ----------- |
| api_id (mandatory) | String	| the ApiID of the Request. It is used it to identify the application making the call. |
| exp (mandatory) | 	Long | Expiration time. It contains the UTC Unix time after which you should no longer accept this token. It must not be after 15 min after the issued time. Only for debugging purpose the 15 mins limit has been removed.|
| bha (mandatory) | String | The hash of the body content, using the Md5 algorithm eg. `md5(request.body)` |

##### Claims Encoded

base64UrlEncode(claims) = `eyJpc3MiOiJkZWJ1Z2dlciIsImV4cCI6MTQ1MTYwNjQwMCwiYmhhIjoiZDQxZDhjZDk4ZjAwYjIwNGU5ODAwOTk4ZWNmODQyN2UifQ`

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
    "bha":  md5(Request.Body) //"d41d8cd98f00b204e9800998ecf8427e"
}

signed = HMACSHA256(
    base64UrlEncode(header)+"."+base64UrlEncode(claims),
    "secret"
)

token = base64UrlEncode(header)+"."+base64UrlEncode(claims)+"."+signed
```
The token now is:
  `eyJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJkZWJ1Z2dlciIsImV4cCI6MTQ1MTYwNjQwMCwiYmhhIjoiZDQxZDhjZDk4ZjAwYjIwNGU5ODAwOTk4ZWNmODQyN2UifQ.jvhMSPJBM9zG1QBAvlW3ICOUNsBZ0J-NL5Sy9q_maI4`

Then add to the the request 

`Request.addHeader("Authorization", "BEARER "+token)`

### Tools 

1. http://jwt.io/
2. http://www.timestampgenerator.com/1451606400/#result

### Related articles

1. https://developer.atlassian.com/static/connect/docs/concepts/understanding-jwt.html
2. https://developers.google.com/wallet/instant-buy/about-jwts
3. http://docs.aws.amazon.com/AmazonS3/latest/dev/RESTAuthentication.html#ConstructingTheAuthenticationHeader
