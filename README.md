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
    │   │                   │   ├── Account.java
    │   │                   │   ├── Authorization.java
    │   │                   │   ├── GrantCode.java
    │   │                   │   ├── RefreshToken.java
    │   │                   │   ├── Role.java
    │   │                   │   └── Scope.java
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
    │               │       │   │   ├── access-tokens.jsp
    │               │       │   │   ├── accounts.jsp
    │               │       │   │   ├── authorizations.jsp
    │               │       │   │   ├── grant-codes.jsp
    │               │       │   │   ├── index.jsp
    │               │       │   │   ├── refresh-tokens.jsp
    │               │       │   │   ├── roles.jsp
    │               │       │   │   └── scopes.jsp
    │               │       │   ├── agent
    │               │       │   │   ├── authorization.jsp
    │               │       │   │   ├── authorizations.jsp
    │               │       │   │   └── index.jsp
    │               │       │   ├── index.jsp
    │               │       │   ├── login.jsp
    │               │       │   ├── register.jsp
    │               │       │   └── user
    │               │       │       ├── authorization.jsp
    │               │       │       ├── authorizations.jsp
    │               │       │       ├── confirmation.jsp
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
        │                   │   ├── TestAccount.java
        │                   │   ├── TestAuthorization.java
        │                   │   ├── TestGrantCode.java
        │                   │   ├── TestRefreshToken.java
        │                   │   ├── TestRole.java
        │                   │   └── TestScope.java
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

## Product Structure
```
.
├── res
│   ├── css
│   ├── img
│   └── js
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
    │       │   ├── admin
    │       │   │   └── index.jsp
    │       │   ├── agent
    │       │   │   └── index.jsp
    │       │   ├── index.jsp
    │       │   ├── login.jsp
    │       │   ├── register.jsp
    │       │   └── user
    │       │       ├── confirmation.jsp
    │       │       └── index.jsp
    │       ├── tags
    │       └── tiles
    └── web.xml
```

## URLs Design
```
.
├── admin
│   ├── access-tokens
│   ├── accounts
│   ├── authorizations
│   ├── grant-codes
│   ├── refresh-tokens
│   ├── roles
│   └── scopes
├── agent
│   ├── authorizations
│   └── update
├── api
│   ├── access-tokens
│   ├── accounts
│   ├── authorizations
│   ├── grant-codes
│   ├── refresh-tokens
│   ├── roles
│   └── scopes
├── oauth
│   ├── access-token
│   ├── grant-code
│   └── refresh-token
├── user
│   ├── authorizations
│   └── update
└── webhook
    └── example
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
<tr>
<th>&nbsp;URL</th>
<th>Controller</th>
<th>Get</th>
<th>Post</th>
<th>Put</th>
<th>Delete</th>
</tr>
<tr>
<td>/register</td>
<td>RegisterController</td>
<td>HTML</td>
<td>Register</td>
<td>&nbsp;-</td>
<td>&nbsp;-</td>
</tr>
<tr>
<td>/login</td>
<td>LoginController</td>
<td>HTML</td>
<td>Login</td>
<td>&nbsp;-</td>
<td>&nbsp;-</td>
</tr>
<tr>
<td>/api</td>
<td rowspan="17">ApiController</td>
<td>-</td>
<td>-</td>
<td>&nbsp;-</td>
<td>&nbsp;-</td>
</tr>
<tr>
<td>/api/access-token</td>
<td>List</td>
<td>Create</td>
<td>&nbsp;-</td>
<td>&nbsp;-</td>
</tr>
<tr>
<td>/api/access-token/{token}</td>
<td>Get</td>
<td>&nbsp;-</td>
<td>&nbsp;-</td>
<td>Delete</td>
</tr>
<tr>
<td>/api/accounts</td>
<td>&nbsp;-</td>
<td>Create</td>
<td>&nbsp;-</td>
<td>&nbsp;-</td>
</tr>
<tr>
<td>/api/accounts/{account}</td>
<td>Get</td>
<td>&nbsp;-</td>
<td>Update</td>
<td>Delete</td>
</tr>
<tr>
<td>/api/accounts/{account/oauth</td>
<td>List</td>
<td>&nbsp;-</td>
<td>&nbsp;-</td>
<td>&nbsp;-</td>
</tr>
<tr>
<td>/api/accounts/{account/oauth/{authorization}</td>
<td>&nbsp;-</td>
<td>Authorize</td>
<td>&nbsp;-</td>
<td>Unauthorize</td>
</tr>
<tr>
<td>/api/authorizations</td>
<td>List</td>
<td>Create</td>
<td>-</td>
<td>-</td>
</tr>
<tr>
<td>/api/authorizations/{authorization}</td>
<td>Get</td>
<td>-</td>
<td>-</td>
<td>Delete</td>
</tr>
<tr>
<td>/api/grant-code</td>
<td>List</td>
<td>Create</td>
<td>&nbsp;-</td>
<td>&nbsp;-</td>
</tr>
<tr>
<td>/api/grant-code/{code}</td>
<td>Get</td>
<td>&nbsp;-</td>
<td>&nbsp;-</td>
<td>Delete</td>
</tr>
<tr>
<td>/api/refresh-token</td>
<td>List</td>
<td>Create</td>
<td>&nbsp;-</td>
<td>&nbsp;-</td>
</tr>
<tr>
<td>/api/refresh-token/{token}</td>
<td>Get</td>
<td>&nbsp;</td>
<td>&nbsp;-</td>
<td>Delete</td>
</tr>
<tr>
<td>/api/roles</td>
<td>List</td>
<td>Create</td>
<td>&nbsp;-</td>
<td>&nbsp;-</td>
</tr>
<tr>
<td>/api/roles/{role}</td>
<td>Get</td>
<td>&nbsp;-</td>
<td>Update</td>
<td>Delete</td>
</tr>
<tr>
<td>/api/scopes</td>
<td>List</td>
<td>Create</td>
<td>-</td>
<td>-</td>
</tr>
<tr>
<td>/api/scopes/{scope}</td>
<td>Get</td>
<td>-</td>
<td>-</td>
<td>Delete</td>
</tr>
<tr>
<td>/oauth/grant-code</td>
<td rowspan="3">OauthController</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
</tr>
<tr>
<td>/oauth/access-token</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
</tr>
<tr>
<td>/oauth/refresh-token</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
</tr>
<tr>
<td>/admin</td>
<td rowspan="15">AdminController</td>
<td>HTML</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
</tr>
<tr>
<td>/admin/access-token</td>
<td>HTML</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
</tr>
<tr>
<td>/admin/access-token/{token}</td>
<td>HTML</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
</tr>
<tr>
<td>/admin/accounts</td>
<td>HTML</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
</tr>
<tr>
<td>/admin/accounts/{account}</td>
<td>HTML</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
</tr>
<tr>
<td>/admin/authorizations</td>
<td>HTML</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
</tr>
<tr>
<td>/admin/authorizations/{authorization}</td>
<td>HTML</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
</tr>
<tr>
<td>/admin/grant-code</td>
<td>HTML</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
</tr>
<tr>
<td>/admin/grant-code/{code}</td>
<td>HTML</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
</tr>
<tr>
<td>/admin/refresh-token</td>
<td>HTML</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
</tr>
<tr>
<td>/admin/refresh-token/{token}</td>
<td>HTML</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
</tr>
<tr>
<td>/admin/roles</td>
<td>HTML</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
</tr>
<tr>
<td>/admin/roles/{role}</td>
<td>HTML</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
</tr>
<tr>
<td>/admin/scopes</td>
<td>HTML</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
</tr>
<tr>
<td>/admin/scopes/{scope}</td>
<td>HTML</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
</tr>
<tr>
<td>/user</td>
<td rowspan="4">UserController</td>
<td>HTML</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
</tr>
<tr>
<td>/user/authorizations</td>
<td>HTML</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
</tr>
<tr>
<td>/usr/authorizations/{authorization}</td>
<td>HTML</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
</tr>
<tr>
<td>/user/update</td>
<td>HTML</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
</tr>
<tr>
<td>&nbsp;</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
</tr>
<tr>
<td>&nbsp;</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
<td>&nbsp;</td>
</tr>
</tbody>
</table>
