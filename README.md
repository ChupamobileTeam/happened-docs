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
    "appid": "A123",
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
| "appid" (mandatory)| string      | The ApplicationId given by the configuration file |
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

{
    "iss": "debugger",
    "exp": 1451606400,
    "bha": "d41d8cd98f00b204e9800998ecf8427e"
}

|   Attribute     |     Type    | Description |
| ----------      | ----------- | ----------- |
| iis (mandatory) | String	| the Issuer of the Request. Connect uses it to identify the application making the call. for example: If the Atlassian product is the calling application: contains the unique identifier of the tenant. This is the clientKey that you receive in the installed callback. You should reject unrecognised issuers. If the add-on is the calling application: the add-on key specified in the add-on descriptor |
| exp (mandatory) | 	Long | Expiration time. It contains the UTC Unix time after which you should no longer accept this token. It must not be after 15 min after the issued time. Only for debugging purpose the 15 mins limit has been removed.|
| bha (mandatory) | String | Body Content Hash the Md5 of the content. eg. `md5(request.body)` |

##### Claims Encoded

base64UrlEncode(claims) = `eyJpc3MiOiJkZWJ1Z2dlciIsImV4cCI6MTQ1MTYwNjQwMCwiYmhhIjoiZDQxZDhjZDk4ZjAwYjIwNGU5ODAwOTk4ZWNmODQyN2UifQ`

### 3 The Signed


### A Pseudo Algorithm

Given `ApiId` and a `secret`
has to send a `Request.Body`

forging the `token`

```
cha = md5(Request.Body)

header = {
    "alg":"HS256"
}

claims = {
    "iss": "debugger",
    "exp": 1451606400,
    "bha": "d41d8cd98f00b204e9800998ecf8427e"
}

token = HMACSHA256(
    base64UrlEncode(header) + "." +
    base64UrlEncode(payload),
    "secret"
)
```
The token is `eyJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJkZWJ1Z2dlciIsImV4cCI6MTQ1MTYwNjQwMCwiYmhhIjoiZDQxZDhjZDk4ZjAwYjIwNGU5ODAwOTk4ZWNmODQyN2UifQ.jvhMSPJBM9zG1QBAvlW3ICOUNsBZ0J-NL5Sy9q_maI4`

Then add to the the request 

`Request.addHeader("Authorization", "BEARER "+token)`

### Articles and tools 

1. http://jwt.io/
2. https://developer.atlassian.com/static/connect/docs/concepts/understanding-jwt.html
3. https://developers.google.com/wallet/instant-buy/about-jwts
4. https://github.com/gin-gonic/contrib/blob/master/jwt/jwt.go
5. https://github.com/dgrijalva/jwt-go/blob/master/jwt.go
6. http://www.timestampgenerator.com/1451606400/#result
http://docs.aws.amazon.com/AmazonS3/latest/dev/RESTAuthentication.html#ConstructingTheAuthenticationHeader
