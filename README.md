API
===

Implementation details for the Gatecoin Http API

---
##General Usage

All API requests should be directed to https://www.gatecoin.com/api.
Please find the list of all APIs here: https://www.gatecoin.com/api/swagger-ui/index.html#!. 
All input parameters(if any), with a POST request, need to be encoded in JSON format.

##API Accessibility

__*Public*__: Everyone can execute the API functions through http without authentication. All public APIs are prefixed by "/Public". Eg. https://www.gatecoin.com/api/Public/MarketDepth/{CurrencyPair}

__*Private*__: Only http requests with required headers can execute the API function.


##Required Headers

`API_PUBLIC_KEY`: The public key which can be obtained after login. Alternatively, the API key can also be generated at:  https://gatecoin.com/login?path=%2Fplatform%2FapiKeyManagement

`API_REQUEST_SIGNATURE`: A valid API Signature signed with API key in HMAC

`API_REQUEST_DATE`: The datetime of request in UTC format


##API Key

__*Long term API Key*__: An API Key generated on demand by the user *(using the /APIKey/APIKey function)*, valid until expiry date or deleted by user. Will not be removed upon logout.

__*Short term API Key*__: An API Key generated at successful login by the user to the system through the API.

##System Login

The user can login to the system through the API in the path /Auth/Login by passing username and password in a POST request. For example: `https://www.gatecoin.com/api/Auth/Login` with json data 
```json
{ UserName : "gtcuser", Password : "oassword" }
```
The json string below will be returned on successful login:
```json
{
  "isSuccess": true,
  "apiKey": "A19DC6BE417615645646216D8589D9818",
  "publicKey": "6967615654684689A7E8FD41E5AB3C6DD",
  "alias": "alias1",
  "defaultCurrency": "BTCUSD",
  "defaultLanguage": "en-us",
  "verifLevel": 3,
  "userHasUnreadTickets": false,
  "lastLogonTime": "1420791056",
  "isPendingUnlockSecret": false,
  "responseStatus": {
    "message": "OK"
  }
}
```

*isSuccess*: *true* if success, otherwise *false*

*apiKey*: The API Key will be expired after 15 mins. The expiry date of API Key will be refreshed after each successful private API function call for up to 24 hour after creation.

For example, for an API Key created on `2013-11-01 07:00:00`, it will be expired on `2013-11-01 07:15:00`. If there is a private API function call at `2013-11-01 07:02:00`, the expiry date will be extended to `2013-11-01 07:17:00`. But the API Key will expire on `2013-11-02 07:00:00` no matter how many times the API Key was refreshed.

##Requested Signature

A valid request signature must be signed using HMAC with an active API Key and converted to base64 string.

The javascripts below contains required library for HMAC and base64 string conversion:

http://crypto-js.googlecode.com/svn/tags/3.0.2/build/rollups/hmac-sha256.js

http://crypto-js.googlecode.com/svn/tags/3.0.2/build/components/enc-base64-min.js

The code below demonstrate how to send required header using ajax:

```javascript
$.ajaxSetup({
	beforeSend: function (jqXHR, settings) {
		var publicKey = $("#input_public_key").val();
		var key = $("#input_key").val();
		if (publicKey == "") {
			publicKey = gPublicKey; // gPublicKey is a variable which stores the publicKey when login
		}
		if (key == "") {
			key = gApiKey; // gApiKey is a variable which stores the apiKey when login
		}
		if (publicKey != null && key != null) {
			var now = (new Date(Date.now())).getTime() / 1000;
			var httpMethod = settings.type;
			var ct = settings.contentType;
			if (ct == false) {
				ct = "multipart/form-data";
			}
			var contentType = (httpMethod == "GET") ? "" : ct;
			var message = settings.type + settings.url + contentType + now;
			var hash = CryptoJS.HmacSHA256(message.toLowerCase(), key);
			var hashInBase64 = CryptoJS.enc.Base64.stringify(hash);

			jqXHR.setRequestHeader("API_PUBLIC_KEY", publicKey);
			jqXHR.setRequestHeader("API_REQUEST_SIGNATURE", hashInBase64);
			jqXHR.setRequestHeader("API_REQUEST_DATE", now);
		}
	}
});

Please check out the folder Client-Code-Examples for more examples in different programming languages.
```
## Response 
Please visit the page https://gatecoin.com/api/swagger-ui/index.html#!/Public for trying out the API and checking the JSON response from server.

## API Rate Limits
The following APIs are rate-limited:

**Balance**

Maximum allowed requests per minute: 30 

Applicable to:

```
GET /Balance/Balances : Gets the available balance for each currency for the logged in account.
GET /Balance/Deposits : Get all account deposits, including wire, okpay and digital currency, of the logged in user
GET /Balance/Withdrawals:  Get all account withdrawals, including wire, okpay digital currency, of the logged in user
GET /Balance/Balances/{Currency}:  Gets the available balance for s currency for the logged in account.
```

**Login**

Maximum allowed requests per minute: 10 

Applicable to:

```
POST /Auth/Login:  Trader session log in.
```

**Order**

Maximum allowed requests per minute: 150 

Applicable to:

```
GET /Trade/Orders:  Gets open orders for the logged in trader.
POST /Trade/Orders:  Place an order at the exchange.
DELETE /Trade/Orders:  Cancels all existing order
GET /Trade/StopOrders:  Gets all stop orders for the logged in trader. Max 1000 record.
GET /Trade/StopOrdersHistory:  Gets all stop orders for the logged in trader. Max 1000 record.
POST /Trade/StopOrders:  Place a stop order at the exchange.
DELETE /Trade/StopOrders:  Cancels all existing stop orders
GET /Trade/Orders/{OrderID}:  Gets an order for the logged in trader.
DELETE /Trade/Orders/{OrderID}:  Cancels an existing order
DELETE /Trade/StopOrders/{ID}:  Cancels an existing stop order
```

**Ping**

Maximum allowed requests per minute: 70

Applicable to:

```
POST /Ping Post a string, then get it back.
```

**Register**

Maximum allowed requests per minute: 3

Applicable to:

```
POST /RegisterUser:  Initial trader registration.
```

**Reset password**

Maximum allowed requests per minute: 10

Applicable to:

```
POST /Auth/ResetPassword:  Reset password

```
**Withdrawal**

Maximum allowed requests per minute: 10

Applicable to:

```
GET /Balance/Withdrawals Get all account withdrawals, including wire, okpay digital currency, of the logged in user
POST /Okpay/Withdrawal Withdraw flat currency through Okpay
POST /ElectronicWallet/Withdrawals/{DigiCurrency} Request a transfer from the traders account to an external address. This is only available for crypto currencies.

```

** Note: For all APIs other than Public ones, the default allowed rate is 150 requests per minute.** 


##System Logout

The user can logout from the system through the API in the path /Logout. All short term API Key will be removed from the system.



