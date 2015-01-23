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
    "sdk": "admob",
    "name": "setMainMenuBannerId",
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
| "sdk" (mandatory)| string      | Vendor name of the third party tool eg. `admob`, `flurry`... |
| "name" (mandatory)| string      | Function name called composed by class.function eg. `admob.setMainMenuBannerId` |
| "value" (optional) | string | Optional is the value of the function called eg. `2` |

 
## Authorization

In order to authorize the request, the client should have a `apiId` and a `apiKey`

for debugging purpose the use as apiId `debugger` and as apiKey `secret`

###  JWT TOKEN

The JWT token is composed by 3 parts, the first is the **header** about the algorithm used, the second is the **claims** called also payload, the third part is the **signed** that certificate the first two.

#### 1 Header

``` json
{
  "alg": "HS256"
}
```

|   Attribute     |     Type    | Description |
| ----------      | ----------- | ----------- |
| "typ" (mandatory) | String    | Type for the token, defaulted to "JWT". Specifies that this is a JWT token | 

##### Header Encoded

base64UrlEncode(header) = `eyJhbGciOiJIUzI1NiJ9`

#### 2 Claims

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

### 3 The Signed part

The signed part is about signing the header and the claim with the shared secret `api_key`.

### A Pseudo Algorithm

Given `api_id` and a `api_key` and a `Request.Body` we can generate the Signed Part.

#### Forging the `token`

```
header = {
    "alg":"HS256"
}

claims = {
    "api_id": "debugger",
    "exp": 1451606400,
    "bha":  md5(Request.Body) //"d41d8cd98f00b204e9800998ecf8427e"
}

token = HMACSHA256(
    base64UrlEncode(header) +"." +base64UrlEncode(claims),
    "secret"
)
```
The token is `eyJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJkZWJ1Z2dlciIsImV4cCI6MTQ1MTYwNjQwMCwiYmhhIjoiZDQxZDhjZDk4ZjAwYjIwNGU5ODAwOTk4ZWNmODQyN2UifQ.jvhMSPJBM9zG1QBAvlW3ICOUNsBZ0J-NL5Sy9q_maI4`

Then add to the the request 

`Request.addHeader("Authorization", "BEARER "+token)`

## Debugging

the root is /api/debug/smtg 


### Articles and tools 

1. http://jwt.io/
2. https://developer.atlassian.com/static/connect/docs/concepts/understanding-jwt.html
3. https://developers.google.com/wallet/instant-buy/about-jwts
4. https://github.com/gin-gonic/contrib/blob/master/jwt/jwt.go
5. https://github.com/dgrijalva/jwt-go/blob/master/jwt.go
6. http://www.timestampgenerator.com/1451606400/#result
http://docs.aws.amazon.com/AmazonS3/latest/dev/RESTAuthentication.html#ConstructingTheAuthenticationHeader
