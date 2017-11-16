# Java Webapp Refference Design

Author: Sun, Chia-Yang \<<triumph.sun@gmail.com>\>


This documents is about how to create a OAuth 2.0 provider.


## Table of Content

## Project Structure
```
.
├── build.gradle
├── docs
├── LICENSE.txt
├── README.md
├── scripts
├── settings.gradle
└── src
    ├── main
    │   ├── java
    │   │   └── com
    │   │       └── suntri
    │   │           └── oauth
    │   │               ├── common
    │   │               └── provider
    │   │                   ├── bean
    │   │                   │   ├── AccessToken.java
    │   │                   │   └── Account.java
    │   │                   ├── service
    │   │                   │   ├── AccessTokenService.java
    │   │                   │   ├── AccountService.java
    │   │                   │   └── impl
    │   │                   │       ├── DummyAccessTokenService.java
    │   │                   │       ├── DummyAccountService.java
    │   │                   │       ├── MySqlAccountService.java
    │   │                   │       └── RedisAccessTokenService.java
    │   │                   ├── servlet
    │   │                   │   ├── controller
    │   │                   │   │   ├── AccountController.java
    │   │                   │   │   ├── LoginController.java
    │   │                   │   │   ├── OAuthController.java
    │   │                   │   │   └── RegisterController.java
    │   │                   │   ├── filter
    │   │                   │   │   ├── AssertAuthenticatedFilter.java
    │   │                   │   │   ├── AuthBasicFilter.java
    │   │                   │   │   ├── AuthBearerFilter.java
    │   │                   │   │   └── EncodingUtf8Filter.java
    │   │                   │   └── listener
    │   │                   │       └── ApplicationListener.java
    │   │                   └── spring
    │   │                       └── interceptor
    │   └── resources
    │       ├── static
    │       └── webapp
    │           ├── res
    │           │   ├── css
    │           │   ├── img
    │           │   └── js
    │           └── WEB-INF
    │               ├── view
    │               │   └── jsp
    │               │       ├── plain
    │               │       │   ├── 401.jsp
    │               │       │   ├── 404.jsp
    │               │       │   ├── admin
    │               │       │   │   └── index.jsp
    │               │       │   ├── index.jsp
    │               │       │   ├── login.jsp
    │               │       │   ├── register.jsp
    │               │       │   └── user
    │               │       │       ├── grant-privilege.jsp
    │               │       │       └── index.jsp
    │               │       └── tiles
    │               └── web.xml
    └── test
        ├── java
        │   └── com
        │       └── suntri
        │           └── oauth
        │               ├── common
        │               └── provider
        │                   ├── bean
        │                   │   ├── TestAccessToken.java
        │                   │   └── TestAccount.java
        │                   ├── service
        │                   │   └── impl
        │                   │       ├── TestDummyAccessTokenService.java
        │                   │       ├── TestDummyAccountService.java
        │                   │       ├── TestMySqlAccountService.java
        │                   │       └── TestRedisAccessTokenService.javva
        │                   ├── servlet
        │                   │   ├── controller
        │                   │   ├── filter
        │                   │   └── listener
        │                   └── spring
        │                       └── interceptor
        └── resources
```

## URLs Design

```
.
├── admin                                --> AdminController
├── api                                  --> ApiController
│   ├── access-token
│   ├── accounts
│   ├── grant-code
│   └── refresh-token
├── oauth                                --> OauthController
│   ├── access-token
│   ├── grant-code
│   └── refresh-token
├── res
│   ├── css
│   ├── img
│   └── js
├── user                                 --> UserController
├── webhook
│   └── example
└── WEB-INF
    ├── classes
    │   └── com
    │       └── suntri
    │           └── oauth
    ├── lib
    ├── tags
    ├── view
    │   └── jsp
    │       ├── plain
    │       │   ├── 401.jsp
    │       │   ├── 404.jsp
    │       │   ├── grant-privilege.jsp
    │       │   ├── index.jsp
    │       │   ├── login.jsp
    │       │   └── register.jsp
    │       ├── tags
    │       └── tiles
    └── web.xml
```

## Attributes Design
### Identity Attribute
Identity Attribute after succesfull authentication and authorization

```javascript
{
	"loginAs": {
		"userName": "suntri.com",
		"userGroup": "oauth",
		"displayName":"Third Party Developer",
		"oauth": {
			"onBehalfOf": "triumph",
			"authedScope": "account",
			"expire": 1300819380
		}
	}
}
```

##  Filters
javax.servlet.Filter

### A. EncodeUtf8Filter
```
1. Convert encoding
```

### B. AuthBearerFilter
```
1. Test existence, signature, and expiration of http header "Authorization: Bearer ${JWT}"
	1.1. If Yes, extract ${JWT} to PAGE, and continue to next filter
	1.2. if No, continue to next filter
```

### C. AuthBasicFilter
```
1. Test existence of http header "Authorization: Basic ${passcode}"
	1. If Yes, extract ${passcode} and apply login procedure
		1. If login success, contiune to next filter
		2. If login failed, discard this request, and RETURN HTTP 403 FORBIDDEN
	2. If No, discard this request, and RETURN HTTP 401 UNAUTHORIZED
```

### D. AssertAuthenticatedFilter
```
1. Test existence of attribute ${loginAs} in either PAGE or SESSION
	1. If Yes, continue to next filter.
	2. If No, discard this request, and RETURN HTTP 401 UNAUTHORIZED
```

### E. AssertGroupAdminFilter
```
```


## Filters & URL Patterns


| URL Pattern | Filters |
|--|--|
| /index.jsp | A
| /register | A
| /login | A
| /api/** | A > B > D
| /auth/** | A > C > D
| /admin/** | A > C > D > E
| /webhook/** | A > C > D
| /user/** | A > C > D
| /res/** | A
| / | A > C > D

## Servlets

### AccountController

| URL | GET | POST | PUT | DELETE |
|--|--|--|--|--|
| /api/accounts | | Create
| /api/accounts/${id} | Retrieve | | Update
| /api/accounts/${id}/oauth/ | List All
| /api/accounts/${id}/oauth/${auth} | | Authorize | | Unauthorize


### OauthController

| URL | GET | POST | PUT | DELETE |
|--|--|--|--|--|
| /auth/grant-code | | Request
| /auth/access-token | | Request
| /auth/refresh-token | | Request

```
POST /auth/ HTTP/1.1
Host: foo.bar
Content-Type: application/json

{

}
```

### LoginController

| URL | GET | POST | PUT | DELETE |
|--|--|--|--|--|
| /login | Get Page| Login

##### Request

If login failed, user will be redirect back to login page.

```
POST /login HTTP/1.1
Host: foo.bar
Content-Type: application/x-www-form-urlencoded;charset=utf-8

action=login&
username=example_username&
password=example_password&
redirectTo=/user/info.jsp&
csrf=abcde
```

|Key | Value | Optional? | Description |
|--|--|--|--|
|action | login, logout 
|username | ${username} | Optional | Optional when *action* equals to logout.
|password | ${password} | Optional | Optional when *action* equals to logout.
|redirectTo |${url}| |Servlet will redirect to specified URL after login successfully.
|csrf | ${csrf} | Optional | Cross-Site Request Forgery token


##### Response
```
HTTP/1.1 200 OK
Application-Type: text/html

<!DOCTYPE html>
<html>
	...
</html>
```

### RegisterController

| URL | GET | POST | PUT | DELETE |
|--|--|--|--|--|
| /register | Get Page| Register

### AdminController
| URL | GET | POST | PUT | DELETE |
|--|--|--|--|--|
| /admin/ | Get Page |

### UserController
| URL | GET | POST | PUT | DELETE |
|--|--|--|--|--|
| /user | Get Page|

##### Request
```
POST /login HTTP/1.1
Host: foo.bar
Content-Type: application/x-www-form-urlencoded;charset=utf-8

action=login&
username=example_username&
password=example_password&
redirectTo=/user/info.jsp&
csrf=abcde
```


## JWT
Java Web Token (JWT) is made up with HEADER, BODY and SIGNATURE.

Header
```javascript
{
    "typ": "JWT",
    "alg": "HS256"
}
```

Body
```javascript
{
    "iss": "http://foo.bar/",
    "exp": 1300819380,
    "scope": ""
}
```

Key | Value | Description
--|--|--
iss | | Issuer of this JWT
exp | | Expiration of this JWT
scope | | Authorized Scope

### Scopes

| Scope | Description |
|--|--|
|Basic | 


### Table
<table>
<tbody>
<tr style="height: 23px;">
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">Controller</td>
<td style="height: 23px;">Get</td>
<td style="height: 23px;">Post</td>
<td style="height: 23px;">Put</td>
<td style="height: 23px;">Delete</td>
</tr>
<tr style="height: 23px;">
<td style="height: 23px;">admin</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 207px;" rowspan="9">AdminController</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
</tr>
<tr style="height: 23px;">
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">access-token</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">Get Page</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
</tr>
<tr style="height: 23px;">
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">{token}</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">Get Page</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
</tr>
<tr style="height: 23px;">
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">account</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">Get Page</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
</tr>
<tr style="height: 23px;">
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">{account}</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">Get Page</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
</tr>
<tr style="height: 23px;">
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">grant-code</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">Get Page</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
</tr>
<tr style="height: 23px;">
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">{code}</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">Get Page</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
</tr>
<tr style="height: 23px;">
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">refresh-token</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">Get Page</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
</tr>
<tr style="height: 23px;">
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">{token}</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">Get Page</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
</tr>
<tr style="height: 23px;">
<td style="height: 23px;">api</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 299px;" rowspan="13">ApiController</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
</tr>
<tr style="height: 23px;">
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">access-token</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
</tr>
<tr style="height: 23px;">
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">{token}</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
</tr>
<tr style="height: 23px;">
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">accounts</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">Create</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
</tr>
<tr style="height: 23px;">
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">{account}</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">Retrieve</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">Update</td>
<td style="height: 23px;">Delete</td>
</tr>
<tr style="height: 23px;">
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">oauth</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">List All</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
</tr>
<tr style="height: 23px;">
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">{authorization}</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">Authorize</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">Unauthorize</td>
</tr>
<tr style="height: 23px;">
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">grant-code</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">List All</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
</tr>
<tr style="height: 23px;">
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">{code}</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
</tr>
<tr style="height: 23px;">
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">refresh-token</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">List All</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
</tr>
<tr style="height: 23px;">
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">{token}</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
</tr>
<tr style="height: 23px;">
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">roles</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">Create</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
</tr>
<tr style="height: 23px;">
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">{role}</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">Retrieve</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">Update</td>
<td style="height: 23px;">Delete</td>
</tr>
<tr style="height: 23px;">
<td style="height: 23px;">user</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 92px;" rowspan="4">UserController</td>
<td style="height: 23px;">Get Page</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
</tr>
<tr style="height: 23px;">
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">
<p>authorizations</p>
</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">Get Page</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
</tr>
<tr style="height: 23px;">
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">{authorization}</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">Get Page</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
</tr>
<tr style="height: 23px;">
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">update</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">Get Page</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
</tr>
<tr style="height: 23px;">
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
</tr>
<tr style="height: 23px;">
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
</tr>
<tr style="height: 23px;">
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
</tr>
<tr style="height: 23px;">
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
</tr>
<tr style="height: 23px;">
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
</tr>
<tr style="height: 23px;">
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
</tr>
<tr style="height: 23px;">
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
</tr>
<tr style="height: 23px;">
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
</tr>
<tr style="height: 23px;">
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
</tr>
<tr style="height: 23px;">
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
</tr>
<tr style="height: 23px;">
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
</tr>
<tr style="height: 23px;">
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
<td style="height: 23px;">&nbsp;</td>
</tr>
</tbody>
</table>
