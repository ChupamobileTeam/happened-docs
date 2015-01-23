happened-docs
-------------

This is the official documentation for the happened server.

## What is

Is a simple daemon for easy events aggregation.

## How it works

Happened accepts request with POST method.

## API 

- [POST] /api/smtg1

  * Headers: 

    - `Content-Type: application/json`

    - `Authorization: BEARER $HEADER.$CLAIMS.$SIGNEDVALUE`

       see below for the meaning of $HEADER $CLAIMS and $SIGNEDVALUE

  * Body:

``` json
{
    "appid": "A123",
    "os": "AND",
    "osvers": "2.1",
    "sdk": "admob",
    "name": "setMainMenuBannerId",
    "value": ""
}
```

## Debugging

@todo

- [POST] /api/debug/smtg 

## 

## Authentication

###  JWT TOKEN

#### Header

``` json
{
    "typ":"JWT",
    "alg":"HS256"
}
```

Attribute	Type	Description
"typ" (mandatory)	String	Type for the token, defaulted to "JWT". Specifies that this is a JWT token
"alg" (optional)	String	Algorithm. specifies the algorithm used to sign the token. In atlassian-connect version 1.0 we support the HMAC SHA-256 algorithm, which the JWT specification identifies using the string "HS256".

#### Payload

{
    "iss": "abc123",
    "iat":  1300819370,
    "exp": 1300819380,
    "cha": "8063ff4ca1e41df7bc90c8ab6d0f6207d491cf6dad7c66ea797b4614b71922e9"
}


Attribute	Type	Description

iis (mandatory)	String	the Issuer of the Request. Connect uses it to identify the application making the call. for example: If the Atlassian product is the calling application: contains the unique identifier of the tenant. This is the clientKey that you receive in the installed callback. You should reject unrecognised issuers. If the add-on is the calling application: the add-on key specified in the add-on descriptor

iat (mandatory)	Long	Issued at time. Contains the UTC Unix time at which this token was issued. There are no hard requirements around this claim but it does not make sense for it to be significantly in the future. Also, significantly old issued-at times may indicate the replay of suspiciously old tokens.

exp (mandatory)	Long	Expiration time. It contains the UTC Unix time after which you should no longer accept this token. It should be after the issued-at time.

cha (mandatory)	String	Content hash. 

### Client Algorithm:

Given `ApiId` and a `secret`
has to send a `Request.Body`

forging the `token`

```
cha = md5(Request.Body)

header = {
    "typ":"JWT",
    "alg":"HS256"
}

payload = {
	"aid": aid
	"hat": unix
	"exp": unix
    "cha": cha
}

token = HMACSHA256(
	base64UrlEncode(header) + "." +
    base64UrlEncode(payload),
    secret
)
```

Then add to the the request 

`Request.addHeader("Authorization", "BEARER "+token)`


### JWT

1. https://developer.atlassian.com/static/connect/docs/concepts/understanding-jwt.html
2. https://developers.google.com/wallet/instant-buy/about-jwts
3. http://jwt.io/
4. https://github.com/gin-gonic/contrib/blob/master/jwt/jwt.go
5. https://github.com/dgrijalva/jwt-go/blob/master/jwt.go

http://docs.aws.amazon.com/AmazonS3/latest/dev/RESTAuthentication.html#ConstructingTheAuthenticationHeader
