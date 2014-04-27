API
===

Implementation details for the gatecoin Http API

---

##API Accessibility

__*Public*__: Everyone can execute the API functions through http without authentication

__*Private*__: Only http request with required headers can execute the API function


##Required Headers

`API_USER_ID`: The unique id for the user, can be obtained after login

`API_REQUEST_SIGNATURE`: An valid API Signature signed with API key in HMAC

`API_REQUEST_DATE`: The datetime of request in UTC format


##API Key

__*Long term API Key*__: An API Key generated on demand by the user *(using the /GetAPIKey function)*, valid until expiry date or deleted by user. Will not be removed upon logout.

__*Short term API Key*__: An API Key generated at successful login by the user to the system through the API.

##System Login

The user can login to the system through the API in the path /Login by passing username and password in a POST request. For example: `http://api.gatecoin.com/v1/Login?UserName=toto1&Password=password`
The json string below will be returned on successful login:
```json
{
  "isSuccess": true,
  "apiKey": "20D04UG394D42FYJUGVIJ7AB9F5F43AF",
  "userId": 1,
  "userName": "toto1",
  "responseStatus": {
    "errorCode": "200",
    "message": "OK"
  }
}
```

*isSuccess*: *true* if success, otherwise *false*

*apiKey*: The API Key for signature, also stored in cookie `COOKIE_API_KEY`. The API Key will be expired after 10 mins. The expiry date of API Key will be refreshed after each successful private API function call for up to 24 hour after creation.

For example, for an API Key created on `2013-11-01 07:00:00`, it will be expired on `2013-11-01 07:10:00`. If there is a private API function call at `2013-11-01 07:02:00`, the expiry date will be extended to `2013-11-01 07:12:00`. But the API Key will expire on `2013-11-02 07:00:00` no matter how many times the API Key was refreshed.

*userId*: The unique id for user, also stored in cookie `COOKIE_API_USER_ID`


##Requested Signature

A valid request signature must be signed using HMAC with an active API Key and converted to base64 string.

The javascripts below contains required library for HMAC and base64 string conversion:

http://crypto-js.googlecode.com/svn/tags/3.0.2/build/rollups/hmac-sha256.js

http://crypto-js.googlecode.com/svn/tags/3.0.2/build/components/enc-base64-min.js

The code below demonstrate how to send required header using ajax:

```javascript
$.ajaxSetup({
    beforeSend: function (jqXHR, settings) {
   	 var userId = getCookie("COOKIE_API_USER_ID");
   	 var key = getCookie("COOKIE_API_KEY");
   	 if (userId != "" && key != "") {
   		 var url = settings.url;
   		 if (url.indexOf(location.host) < 0) {
   			 url = location.protocol + "//" + location.host + url;
   		 }
   		 var now = new Date(Date.now()).toUTCString();
   		 var httpMethod = settings.type;
   		 var contentType = (httpMethod == "GET") ? "" : settings.contentType;
   		 var message = settings.type + url + contentType + now;
   		 var hash = CryptoJS.HmacSHA256(message.toLowerCase(), key);
   		 var hashInBase64 = CryptoJS.enc.Base64.stringify(hash);
   		 jqXHR.setRequestHeader("API_USER_ID", userId);
   		 jqXHR.setRequestHeader("API_REQUEST_SIGNATURE", hashInBase64);
   		 jqXHR.setRequestHeader("API_REQUEST_DATE", now);
   	 }
    }
});

function getCookie(c_name) {
    var c_value = document.cookie;
    var c_start = c_value.indexOf(" " + c_name + "=");
    if (c_start == -1) {
   	 c_start = c_value.indexOf(c_name + "=");
    }
    if (c_start == -1) {
   	 c_value = null;
    }
    else {
   	 c_start = c_value.indexOf("=", c_start) + 1;
   	 var c_end = c_value.indexOf(";", c_start);
   	 if (c_end == -1) {
   		 c_end = c_value.length;
   	 }
   	 c_value = unescape(c_value.substring(c_start, c_end));
    }
    return c_value;
};
```
##System Logout

The user can logout from the system through the API in the path /Logout. All short term API Key will be removed from the system. The cookies `COOKIE_API_KEY` and `COOKIE_API_USER_ID` will also be removed.



