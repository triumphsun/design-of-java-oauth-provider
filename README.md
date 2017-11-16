# Java Webapp Refference Design

Author: Sun, Chia-Yang \<<triumph.sun@gmail.com>\>


This documents is about how to create a OAuth 2.0 provider.


## Table of Content




## Attributes
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
1. Test existence, signature, and expiration of header "Authorization: Bearer ${JWT}"
	1.1. If Yes, extract ${JWT} to PAGE, and continue to next filter
	1.2. if No, continue to next filter
```

### C. AuthBasicFilter
```
1. Test existence of header "Authorization: Basic ${passcode}"
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
| /auth/ | | 

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
Application-Type: test/html

<!DOCTYPE html>
<html>
	...
</html>
```

### RegisterController

| URL | GET | POST | PUT | DELETE |
|--|--|--|--|--|
| /register | Get Page| Register

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
	"iss": "http://",
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

