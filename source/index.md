---
title: API Reference Rotterdampas

toc_footers:
  - <a href='http://yipyip.nl'>Copyright YipYip 2015</a>
  - <a href='http://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---

# Introduction

The following document is written to enable the developers of Intermediad to implement the API for the Rotterdampas apps. This document is guided by the interaction and visual design created by IN10, and the preliminary database diagram supplied by IN10. This document is still a work in progress and therefor can and will be changed during development. However, this document will be used as the guide to write tests and validate the implementation so changes must always be commited in this document.

## RESTful

This API is designed to confirm to some RESTful best practices as described [here](http://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api), while omitting some of the more advanced features to keep the develop time at a minumum.

## JSON

All requests and respons bodies (except when specifically stated otherwise) are sent and received in valid JSON. Text fields will always be displayed as-is and not be parsed, so they should not contain HTML. All requests should be encoded as UTF-8.

# Authentication

In every service there is a basic authentication. And in almost all others there is also a Token verification. This will be set in header with header name 'X-AUTHENTICATION-TOKEN'. A Token can be retrieved with the REST API GetToken.
In Acceptatie the credentials of the basic authentication are:
```
login name: wsrest2
password: Intermediad!2
```
Authentication in Productie will follow after testing.

# Cache

Caching should be used whenever possible. This reduces server load significantly. It is recommended to at least use caching with a "Last Modified" header and / or ETag. This will reduce server time since the body does not have to be calculated, and since we only request the HEAD, bandwidth is also saved.

# Versioning

The version is determined by the version number that is passed as a parameter in the request.

The API should be versioned in order to allow for multiple versions of the app. This way we can migrate to a new version without disabling functionality for users that have not updated yet. The versioning scheme should be relatively simple. For example: `/api/v1/actions`

The latest published version of the api can be retrieved using the `/api/{version}/about` call.

# Optional Global Call Parameters

## Paging

All requests can be done with the optional `page` and `per_page` parameter to specify pagination. `page` is the zero-based index of the page to retrieve and `per_page` the maximum number of items. Pagination must be passed as a parameter

# General

## Get API status

Returns the version status of the API, consisting of the minimal supported version and the highest published version.

> Reponse

```json
{
  "latest_api_version": 4,
  "minimal_api_version": 1
}
```

### Request

`GET /api/{version}/about`


# User management

<aside class="warning">
User management will be specified when the authentication research has concluded.
</aside>

## Login

### Get Token

Checks the account name and password and returns a token. 

```
Request endpoint Acceptatie:
GET https://rotterdampas-acc.passcloud.nl/rest/gettoken/
Request endpoint Productie:
GET https://rotterdampas.passcloud.nl/rest/gettoken/
```

### Headers

Header | Default | Description
------ | ------- | -----------
pass_owner_code | APAS | The code of the 'Organization'.
api_version | 1 | The version number of the API.
pass_type_number | 354 | The number of the 'PasSoort'.

#### Parameters

Parameter | Type | Optional | Default | Description
--------- | ---- | -------- | ------- | -----------
account_login | string | false | - | The inlog name of the user.
account_password | string | false | - | The password for login.


> Response

```json
{
  "token": "JmlVy3BLx86Hxb23iBGMuD1cfHp7GU9d1XrmwVLVpp3Iu5XqU63TLx2AoqoaZ0sh",
  "account_login": "pietjepuk@gmail.com"
}
```

Code | Description
---- | -----------
400 | One or more mandatory parameters and/or headers are empty.
401 | Wrong values in the basic authentication.
403 | Can't find the PasSoort.
403 | Can't find the organization.
404 | Can't find the Pashouder.
200 | Everything is ok.

## Registration

Includes all the user info like name, e-mail adress and pass number.

### PutRegister

Activate a Pashouder with a Pass.

#### Request

```
Request endpoint Acceptatie: 
PUT https://rotterdampas-acc.passcloud.nl/rest/putregister/
Request endpoint Productie: 
PUT https://rotterdampas.passcloud.nl/rest/putregister/
```

### Headers

Header | Default | Description
------ | ------- | -----------
pass_owner_code | APAS | The code of the 'Organization'.
api_version | 1 | The version number of the API.
pass_type_number | 354 | The number of the 'PasSoort'.


#### Parameters

Parameter | Type | Optional | Default | Description
--------- | ---- | -------- | ------- | -----------
password | string | false | - | A password for login.
email_address | string | false | - | The emailaddress of the 'Pashouder'. This will be the login name.
birthdate | string | false | - | The date of birth of the 'Pashouder'.
passnumber | long | false | - | The number of the pass that belongs to the 'Pashouder'.


> Response

```json
{
"token": "pU9BnOY5gkXl3J8lLOCMD2psIS8spAoBpvljhzupASzdPeQ35W3Jt5S3w2dJUVap"
}
```

#### Status code

Code | Description
---- | -----------
400 | One or more mandatory parameters and/or headers are empty.
401 | Wrong values in the basic authentication.
403 | Can't find the 'PasSoort'.
403 | Can't find the 'organization' of the pass.
403 | Error while trying to change the password.
404 | Can't find a Pashouder with the input parameters.
404 | Can't find a Pas with the 'passnumber' parameter.
200 | Everything is ok.


## Update user

Includes all the user info like name, e-mail adress and pass number.

## Update Password

### PutChangePassword

Changes the users password.

### Request

```
Request endpoint Acceptatie: 
PUT https://rotterdampas-acc.passcloud.nl/rest/putchangepassword/
Request endpoint Productie: 
PUT https://rotterdampas.passcloud.nl/rest/putchangepassword/
```

### Headers

Header | Default | Description
------ | ------- | -----------
X-AUTHENTICATION-TOKEN | - | The personal token of the user.
pass_owner_code | APAS | The code of the 'Organization'.
api_version | 1 | The version number of the API.
pass_type_number | 354 | The number of the 'PasSoort'.


### Parameters

Parameter | Type | Optional | Default | Description
--------- | ---- | -------- | ------- | -----------
old_password | string | false | - | The old password for login.
new_password | string | false | - | A new password for login.
email_address | string | false | - | The email/ inlog name of the Pashouder.

> Response

```
The response will only return a status code. 
```

### Status code

Code | Description
---- | -----------
400 | One or more mandatory parameters and/or headers are empty.
401 | Wrong values in the basic authentication.
401 | Wrong values in X-AUTHENTICATION-TOKEN authentication.
403 | Can't find the 'PasSoort'.
403 | Can't find the 'organization' of the pass.
403 | Wrong value for the new password.
404 | Can't find a Pashouder with the input parameters.
200 | Everything is ok.


## Forgot Password

### PutForgotPassword

Sends an email to the users email address. In this mail the user can click on a link which will link to a website where the user can change his or her password.

This REST API Forgot Password will follow after the implementation of the DeepLink Module in PASS.


## Update user photo

Posts an image belonging to the user.

### Request

A multipart request containing an image under fieldname "image".

`POST /api/{version}/users/me/images`

## Get settings

Returns all settings for the current authenticated user.

```
Request endpoint Acceptatie:
GET https://rotterdampas-acc.passcloud.nl/rest/getsettings/
Request endpoint Productie:
GET https://rotterdampas.passcloud.nl/rest/getsettings/
```

```
Authentication in Acceptatie: Basic authentication with a rest user.
username: wsrest2
password: Intermediad!2
```

```
Authentication in Productie will follow after testing.
```

### Parameters

Parameter | Type | Optional | Default | Description
--------- | ---- | -------- | ------- | -----------
pass_owner_code | string | verplicht | - | The code of the 'Organization'.
api_version | float | verplicht | - | The version number of the API.
pass_type_number | integer | verplicht | - | The number of the 'PasSoort'.
account_login | string | verplicht | - | The inlog name of the Pashouder.
account_password | string | verplicht | - | The password of the Pashouder.

> Response

```json
{
   "preference_sports": 0,
   "email_partner_reaction": false,
   "preference_making_fun": 0,
   "email_last_minute": false,
   "email_review_invitation": true,
   "preference_eating_drinking": 1,
   "notification_personal_tips": true,
   "notification_review_invitation": false,
   "notification_action": true,
   "preference_learn": -1,
   "email_pass_information": true,
   "preference_tour": 1,
   "preference_watching_listening": -1,
   "notification_last_minute": false,
   "preference_relax": 0,
   "email_personal_tips": true
}
```

### Status code

Code | Description
---- | -----------
400 | One or more mandatory parameters are empty.
400 | Can't find the PasSoort.
401 | Can't find the organization.
401 | Wrong values in the basic authentication.
401 | Can't verify Pashouder.
404 | Can't find the Pashouder.
404 | Can't find any settings of the given Pashouder.
500 | No values in the basic authentication.
200 | Everything is ok.


> Response

```json
{
  "notification_last_minute": true,
  "notification_review_invitation": true,
  "notification_personal_tips": true,
  "notification_action": true,
  "email_last_minute": true,
  "email_review_invitation": true,
  "email_personal_tips": true,
  "email_partner_reaction": true,
  "email_pass_information": true,
  "preferences" : {
      "1": 0.0,
      "2": -1.0,
      "3": 1.0
  }
}
```
> preferences are based on the pillar enum and a floating point value from -1.0 to 1.0.


### Request
`GET /api/{version}/users/me/settings`

## Update settings

Update settings for the current authenticated user.

```
Request endpoint Acceptatie:
PUT https://rotterdampas-acc.passcloud.nl/rest/putsettings/
Request endpoint Productie:
PUT https://rotterdampas.passcloud.nl/rest/putsettings/
```

```
Authentication in Acceptatie: Basic authentication with a rest user.
username: wsrest2
password: Intermediad!2
```

```
Authentication in Productie will follow after testing.
```

### Parameters

Parameter | Type | Optional | Default | Description
--------- | ---- | -------- | ------- | -----------
pass_owner_code | string | verplicht | - | The code of the 'Organization'.
api_version | float | verplicht | - | The version number of the API.
pass_type_number | integer | verplicht | - | The number of the 'PasSoort'.
account_login | string | verplicht | - | The inlog name of the Pashouder.
account_password | string | verplicht | - | The password of the Pashouder.
preference_making_fun | float | optioneel | current value | Options are: 1 = meer, 0 = gemiddeld, -1 = minder
preference_eating_drinking | float | optioneel | current value | Options are: 1 = meer, 0 = gemiddeld, -1 = minder
preference_tour | float | optioneel | current value | Options are: 1 = meer, 0 = gemiddeld, -1 = minder
preference_watching_listening | float | optioneel | current value | Options are: 1 = meer, 0 = gemiddeld, -1 = minder
preference_learn | float | optioneel | current value | Options are: 1 = meer, 0 = gemiddeld, -1 = minder
preference_sports | float | optioneel | current value | Options are: 1 = meer, 0 = gemiddeld, -1 = minder
preference_relax | float | optioneel | current value | Options are: 1 = meer, 0 = gemiddeld, -1 = minder
notification_last_minute | boolean | optioneel | current value |  				
notification_review_invitation | boolean | optioneel | current value | 
notification_personal_tips | boolean | optioneel | current value | 
notification_action | boolean | optioneel | current value | 
email_last_minute | boolean | optioneel | current value | 
email_review_invitation | boolean | optioneel | current value | 
email_personal_tips | boolean | optioneel | current value | 
email_partner_reaction | boolean | optioneel | current value | 
email_pass_information | boolean | optioneel | current value | 

> Response

```json
{
   "preference_sports": 0,
   "email_partner_reaction": false,
   "preference_making_fun": -1,
   "email_last_minute": true,
   "email_review_invitation": true,
   "preference_eating_drinking": -1,
   "notification_personal_tips": true,
   "notification_review_invitation": false,
   "notification_action": false,
   "preference_learn": 1,
   "email_pass_information": true,
   "preference_tour": -1,
   "preference_watching_listening": 0,
   "notification_last_minute": true,
   "preference_relax": 1,
   "email_personal_tips": false
}
```

### Status code

Code | Description
---- | -----------
400 | One or more mandatory parameters are empty.
400 | Can't find the PasSoort.
401 | Can't find the organization.
401 | Wrong values in the basic authentication.
401 | Can't verify Pashouder.
404 | Can't find the Pashouder.
404 | Can't find any settings of the given Pashouder.
500 | No values in the basic authentication.
200 | Everything is ok.

> Request & Response

```json
{
  "notification_last_minute": true,
  "notification_review_invitation": true,
  "notification_personal_tips": true,
  "notification_action": true,
  "email_last_minute": true,
  "email_review_invitation": true,
  "email_personal_tips": true,
  "email_partner_reaction": true,
  "email_pass_information": true,
  "preferences" : {
      "1": 0.0,
      "2": -1.0,
      "3": 1.0
  }
}
```
> preferences are based on the pillar enum and a floating point value from -1.0 to 1.0.

### Request
`PUT /api/{version}/users/me/settings`


## Add Device id

Register a new device id to the users account.

```
Request endpoint Acceptatie:
POST https://rotterdampas-acc.passcloud.nl/rest/postdevice/
Request endpoint Productie:
POST https://rotterdampas.passcloud.nl/rest/postdevice/
```

```
Authentication in Acceptatie: Basic authentication with a rest user.
username: wsrest2
password: Intermediad!2
```

```
Authentication in Productie will follow after testing.
```

### Parameters

Parameter | Type | Optional | Default | Description
--------- | ---- | -------- | ------- | -----------
pass_owner_code | string | verplicht | - | The code of the 'Organization'.
api_version | float | verplicht | - | The version number of the API.
pass_type_number | integer | verplicht | - | The number of the 'PasSoort'.
account_login | string | verplicht | - | The inlog name of the Pashouder.
account_password | string | verplicht | - | The password of the Pashouder.
device_id | string | verplicht | - | The id of the device.
type_ | string | verplicht | - | Type can be 'android' or 'ios'. 

> Response: returns only a status code.

### Status code

Code | Description
---- | -----------
400 | One or more mandatory parameters are empty.
400 | Can't find the PasSoort.
401 | Can't find the organization.
401 | Wrong values in the basic authentication.
401 | Can't verify Pashouder.
404 | Can't find the Pashouder.
500 | No values in the basic authentication.
200 | Everything is ok.

> Request

```json
{
  "device_id": "ahasdjhdaskjd-asdjkaskjd-sad-asd-ascscx-zxc",
  "type": "android"
}
```
> Type can be "android" or "ios"

> Response

> A status code should be enough.


Register your device id for notifications.

### Request
`POST /api/{version}/users/me/devices`

# Actions

## Get All Actions

```
Request endpoint Acceptatie:
GET https://rotterdampas-acc.passcloud.nl/rest/getallactions/
Request endpoint Productie:
GET https://rotterdampas.passcloud.nl/rest/getallactions/
```

> The date is formatted following ISO 8601: `yyyy-MM-dd'T'HH:mm:ssZZZZZ`

> Offer.type is an enum: 1 = or, 2 = and

This returns the basic information for all the actions that are active right now ordered by `end_date` ascending.
<aside class="success">
Remember — The results of this call can also be changed by using one of the global call parameters.
</aside>

### Filter

<aside class="warning">
Filters are still a work in progress and open for discussion.
</aside>

Example — Filters are based on an Enum e.g. "Eten & Drinken" = 1, "Zonder kinderen" = 2 and are sent as a comma seperated string.
These enums will translate to the "Pijler", Boolean flags of the "Acties", location regions and "Aanbiedingen" values.

`GET /api/{version}/actions?filter=1,3,6`

### Headers

Header | Default | Description
------ | ------- | -----------
X-AUTHENTICATION-TOKEN | - | The personal token of the user.
pass_owner_code | APAS | The code of the 'Organization'.
api_version | 1 | The version number of the API.
pass_type_number | 354 | The number of the 'PasSoort'.

### Parameters

Parameter | Type | Optional | Default | Description
--------- | ---- | -------- | ------- | -----------
page | integer | false | - | The result page number.
per_page | integer | false | - | The count of the results per page.
meta | boolean | true | false | If set, the body will not contain the results but just the meta data for this call.
pillar | string | true | - | Specifies the type of actions.
latitude | float | true | - | If set, the actions are returned based on distance from the latitude & longitude. Only applies if both are set.
longitude | float | true | - | If set, the actions are returned based on distance from the latitude & longitude. Only applies if both are set.
partner_id | integer | true | - | If set, results are constrained to this partner id.
action_type | string | true | - | If set, results are constrained to this action type.

> Response

```json
{
  "page_size": 1,
  "meta_all_actions": [{
    "type_": "Standaard",
    "has_user_shared_experience": false,
    "all_actions_locations": [ 	{
		"id_": 545,
		"title": "Amsterdam",
		"street": "straat",
		"zipcode": "4444 ZZ",
		"street_number": 44,
		"region": "Zuid-Holland",
		"latitude": 999999,
		"longitude": 999999,
	}],
    "all_actions_offers": [ 	{
		"title": "Titel"
		"percentage": 4.75,
		"amount": 7.95,
	}],
    "title": "\"gratis toegang tot het Allard Pierson Museum voor Stadspashouders van 4 t/m 16 jaar\"",
    "end_date": 1451602799000,
    "distance": 9.99999999E8,
    "thumbnail": "http://example.com/example.jpg",
    "all_actions_partners": [{
      "region": "Amsterdam",
      "phone_number": "020 525 25 56",
      "zipcode": "1012 GC",
      "street": "Oude Turfmarkt",
      "name": "Allard Pierson Museum",
      "street_number": "127",
      "id_": 100014,
      "url": "http://www.allardpiersonmuseum.nl",
      "email_address": "allard.pierson.museum@uva.nl"
    }],
    "action_type": "A - Jaarkorting",
    "short_description": "kids 4- 16 jaar",
    "pillar": "Cultuur",
    "id_": 30281,
    "has_user_consumed_action": false,
    "is_user_wishlist_item": false,
    "start_date": 1422745200000
  }],
  "page": 1,
  "total_records": 13
}
```

### Status code

Code | Description
---- | -----------
400 | One or more mandatory parameters and/or headers are empty.
401 | Wrong values in basic authentication.
401 | Wrong values in X-AUTHENTICATION-TOKEN authentication.
403 | Can't find the 'PasSoort'.
403 | Can't find the 'organization' of the pass.
200 | Everything is ok.



## Get Action

Returns a specific action.

```
Request endpoint Acceptatie:
GET https://rotterdampas-acc.passcloud.nl/rest/getaction/
Request endpoint Productie:
GET https://rotterdampas.passcloud.nl/rest/getaction/
```

### Request

`GET /api/{version}/actions/{id}`

### Headers

Header | Default | Description
------ | ------- | -----------
X-AUTHENTICATION-TOKEN | - | The personal token of the user.
pass_owner_code | APAS | The code of the 'Organization'.
api_version | 1 | The version number of the API.
pass_type_number | 354 | The number of the 'PasSoort'.


### Parameters

Parameter | Type | Optional | Default | Description
--------- | ---- | -------- | ------- | -----------
user_review | boolean | true | false | If set, only returns the review belonging to this user.
id_ | integer | false | - | ID number of the specific action.


> Response

```json
{
  "type_": "Standaard",
  "reservation_phone_number": "0172-444705",
  "action_locations": [ 	{
	"id_": 545,
	"title": "Amsterdam",
	"street": "straat",
	"zipcode": "4444 ZZ",
	"street_number": 44,
	"region": "Zuid-Holland",
	"latitude": 999999,
	"longitude": 999999,
  }],
  "has_user_shared_experience": false,
  "action_partners": [{
    "region": "Amsterdam",
    "phone_number": "020 763 0599",
    "zipcode": "1082 ME",
    "street": "G. Mahlerlaan",
    "name": "Amsterdam Expo BV",
    "street_number": "106",
    "id_": 10234,
    "url": "http://www.amsterdamexpo.nl",
    "email_address": "arnold.vandewater@terminal1.nl"
  }],
  "action_tags": [	{
	"title": "Titel Tag"
  }],
  "action_flags": [{
    "is_same_day_consumable": false,
    "is_indoors": false,
    "is_child_friendly": false,
    "is_fun_without_children": false,
    "is_bad_weather": false,
    "is_nice_weather": false
  }],
  "more_information_phone_number": "0172-444705",
  "action_offers": [ 	{
	"title": "Titel"
	"percentage": 4.75,
	"amount": 7.95,
	"get_action_tariffs": [{
		"description": "omschrijving tarief",
		"tariff_original": 50.50,
		"tariff_dicount": 10.50,
		"minimum_age": 18,
		"maximum_age": 65,			
  }]}]
  "title": "Stadspas Aanbieding 100% Korting",
  "end_date": 1441058399000,
  "short_description": "Stadspasje Aanb. A",
  "availability_type": 2,
  "reservation_url": "http://www.example.com/",
  "total_reviews": 3,
  "action_images": [	{
	"cdn_url": "http://www.example.com/example.jpg"
  }],
  "more_information_email": ,
  "thumbnail": "http://example.com/example.jpg",
  "average_review": 5.433333333333334,
  "pillar": "Cultuur",
  "id_": 30274,
  "reservation_email": "test@testen.nl",
  "more_information_url": "http://www.example.com/",
  "is_user_wishlist_item": false,
  "has_user_consumed_action": false,
  "start_date": 1422745200000
}
```


### Status code

Code | Description
---- | -----------
400 | One or more mandatory parameters are empty.
400 | Can't find the 'PasSoort'.
401 | Can't find the 'organization' of the pass.
401 |	Wrong values in the basic authentication
404 | Can't find any action with the 'id_'.
500 | No values in the basic authentication.
200 | Everything is ok.


## Get Related Actions

Returns all actions related to this actions based on tags and usage.

```
Request endpoint Acceptatie:
GET https://rotterdampas-acc.passcloud.nl/rest/getrelatedactions/
Request endpoint Productie:
GET https://rotterdampas.passcloud.nl/rest/getrelatedactions/
```

```
Authentication in Acceptatie: Basic authentication with a rest user.
username: wsrest2
password: Intermediad!2
```

```
Authentication in Productie will follow after testing.
```

### Parameters

Parameter | Type | Optional | Default | Description
--------- | ---- | -------- | ------- | -----------
pass_owner_code | string | verplicht | - | The code of the 'Organization'.
api_version | float | verplicht | - | The version number of the API.
pass_type_number | integer | verplicht | - | The number of the 'PasSoort'.
account_login | string | verplicht | - | The inlog name of the Pashouder.
account_password | string | verplicht | - | The password of the Pashouder.
id_ | integer | verplicht | - | ID number of the specific action.
page | integer | verplicht | - | The result page number.
per_page | integer | verplicht | - | The count of the results per page.
meta | boolean | optioneel | false | If set, the body will not contain the results but just the meta data for this call.
pijler_naam | string | optioneel | - | Specifies the type of actions.
latitude | float | optioneel | - | If set, the actions are returned based on distance from the latitude & longitude. Only applies if both are set.
longitude | float | optioneel | - | If set, the actions are returned based on distance from the latitude & longitude. Only applies if both are set.

> Response

```json
{
  "page_size": 10,
  "get_related_actions_meta_actions": [{
    "get_related_actions_actions_locations": [	 {
		"id_": 545,
		"title": "Amsterdam",
		"street": "straat",
		"zipcode": "4444 ZZ",
		"street_number": 44,
		"region": "Zuid-Holland",
		"latitude": 999999,
		"longitude": 999999,
	}],
    "get_related_actions_actions_offers": [	  {
		"title": "Titel"
		"percentage": 4.75,
		"amount": 7.95,
	}],
    "has_user_shared_experience": false,
    "get_related_actions_actions_partners": [{
		"region": "Amsterdam",
		"phone_number": "010-1234567",
		"zipcode": "1102 AX",
		"street": "Raoul Wallenbergstraat ",
		"name": "Stadsloket Oost",
		"street_number": "43",
		"id_": 10238,
		"url": "www.example.com",
		"email_address": "contact@stichtingjuliusleeft.nl"
    }],
    "title": "Volledige Vergoeding NIK 1 Stadspas Extra Korting",
    "end_date": 1751407199000,
    "short_description": "NIK Stadspas 1",
    "pillar": "Dienstverlening",
    "id_": 30440,
    "is_user_wishlist_item": false,
    "has_user_consumed_action": true,
    "start_date": 1433196000000
  }],
  "page": 1,
  "total_records": 1
}
```

### Status code

Code | Description
---- | -----------
400 | One or more mandatory parameters are empty.
400 | Can't find the PasSoort.
401 | Can't find the organization.
401 | Wrong values in the basic authentication.
401 | Can't verify Pashouder.
404 | Can't find the Pashouder.
404 | Can't find the action with these parameters.
404 | Can't find any tags linked to the action.
500 | No values in the basic authentication.
200 | Everything is ok.

> Reponse

```json
{
  "meta": {
    "page": 0,
    "page_size": 30,
    "total_records": 4000
  },
  "actions": [
    {
      "id": 1,
      "title": "Zin van bewegen",
      "pillar": "EtenDrinken",
      "thumbnail": "http://example.com/example.jpg",
      "short_description": "Letterlijk tussen de zee- en binnenvaartschepen ontdek jij het Rotterdamse havengebied. De indrukwekkende skyline glijdt aan je voorbij. En dan werven, dokken en oneindig veel containers…",
      "is_user_wishlist_item": true,
      "has_user_consumed_action": true,
      "has_user_shared_experience": true,
      "start_date":"2015-12-09T19:33:00 +0000",
      "end_date": "2015-12-13T19:33:00 +0000",
      "offers": {
        "type": 1,
        "offer": [
          {
            "title": "Voor 5 euro naar de dierentuin",
            "percentage": 25.0,
            "amount": 23.0
          }
        ]
      },
      "partner": {
        "id": 1,
        "name": "Spido",
        "url": "http://spido.nl",
        "phone_number": "010 42984039",
        "email_address": "example@spido.nl",
        "street": "Botenlaan",
        "zipcode": "2343KJ",
        "street_number": "45",
        "region": "Rotterdam"
      },
      "locations": [
        {
          "id": 1,
          "title": "Ballenbak",
          "street": "Jan Luykenlaan",
          "zipcode": "2343KJ",
          "street_number": "8",
          "region": "Rotterdam",
          "latitude": 4.0000,
          "longitude": 51.0000
        }
      ]
    }
  ]
}
```
### Request

`GET /api/{version}/actions/{id}/related`

### Query Parameters

Parameter | Optional | Default | Description
--------- | -------- | ------- | -----------
meta | true | false | If set, the body will not contain the results but just the meta data for this call.

# Reviews

## Get action reviews

Returns all reviews of an "Actie". Sorted by date added, descending.

```
Request endpoint Acceptatie:
GET https://rotterdampas-acc.passcloud.nl/rest/getactionreview/
Request endpoint Productie:
GET https://rotterdampas.passcloud.nl/rest/getactionreview/
```

```
Authentication in Acceptatie: Basic authentication with a rest user.
username: wsrest2
password: Intermediad!2
```

```
Authentication in Productie will follow after testing.
```

### Parameters

Parameter | Type | Optional | Default | Description
--------- | ---- | -------- | ------- | -----------
pass_owner_code | string | verplicht | - | The code of the 'Organization'.
api_version | float | verplicht | - | The version number of the API.
pass_type_number | integer | verplicht | - | The number of the 'PasSoort'.
account_login | string | verplicht | - | The inlog name of the Pashouder.
account_password | string | verplicht | - | The password of the Pashouder.
page | integer | verplicht | - | The result page number.
per_page | integer | verplicht | - | The count of the results per page.
action_id | integer | verplicht | - | ID number of the specific action.
meta | boolean | optioneel | false | If set, the body will not contain the results but just the meta data for this call.

> Response

```json
{
  "get_action_reviews_meta_reviews": [{
    "author": "R Janssen",
    "title": "Test Review #3",
    "is_user_review": true,
    "description": "Een stukje tekst",
    "rating": 7,
    "id_": 1,
    "post_date": 1444643804811
  }],
  "page_size": 1,
  "average_review": 4.86,
  "page": 1,
  "total_records": 5
}
```

### Status code

Code | Description
---- | -----------
400 | One or more mandatory parameters are empty.
400 | Can't find the PasSoort.
401 | Can't find the organization.
401 | Wrong values in the basic authentication.
401 | Can't verify Pashouder.
404 | Can't find the Pashouder.
404 | Can't find the action related to the parameter 'action_id'.
404 | Can't find any reviews that belong to the related action.
500 | No values in the basic authentication.
200 | Everything is ok.

> Response

```json
{
  "meta": {
    "page": 0,
    "page_size": 30,
    "total_records": 4000,
    "average_review": 5.4
  },
  "reviews": [
    {
      "id": 3,
      "author": "Robin Kruijt",
      "user_thumbnail": "http://example.com/example.jpg",
      "image_url": "http://example.com",
      "rating": 4.8,
      "title": "Prachtig uitzicht",
      "description": "Prachtig uitzicht, heerlijk eten.",
      "post_date": "2015-12-13T19:33:00 +0000",
      "comment_url": "http://example.com",
      "is_user_review": true
    }
  ]
}
```

### Request

`GET /api/{version}/actions/{id}/reviews`


## Add review

Creates a review and returns it.

```
Request endpoint Acceptatie:
POST https://rotterdampas-acc.passcloud.nl/rest/postreview/
Request endpoint Productie:
POST https://rotterdampas.passcloud.nl/rest/postreview/
```

```
Authentication in Acceptatie: Basic authentication with a rest user.
username: wsrest2
password: Intermediad!2
```

```
Authentication in Productie will follow after testing.
```

### Parameters

Parameter | Type | Optional | Default | Description
--------- | ---- | -------- | ------- | -----------
pass_owner_code | string | verplicht | - | The code of the 'Organization'.
api_version | float | verplicht | - | The version number of the API.
pass_type_number | integer | verplicht | - | The number of the 'PasSoort'.
account_login | string | verplicht | - | The inlog name of the Pashouder.
account_password | string | verplicht | - | The password of the Pashouder.
action_id | integer | verplicht | - | ID number of the specific action.
title | string | verplicht | - | The title of the new review.
rating | float | verplicht | - | The rating of the new review.
description | string | optioneel | - | The description of the review.

> Response

```json
{
  "author": "R Janssen",
  "title": "Test Review #3",
  "is_user_review": true,
  "description": "Een stukje tekst",
  "rating": 7,
  "id_": 1,
  "post_date": 1444643820634
}
```

### Status code

Code | Description
---- | -----------
400 | One or more mandatory parameters are empty.
400 | Can't find the PasSoort.
401 | Can't find the organization.
401 | Wrong values in the basic authentication.
401 | Can't verify Pashouder.
404 | Can't find the Pashouder.
404 | Can't find the action related to the parameter 'action_id'.
500 | Internal server error while creating a new review.
500 | No values in the basic authentication.
200 | Everything is ok.

> Request

```json
{
  "title": "Hallo",
  "description": "Beste dag van mijn leven",
  "rating": 4.8
}
```
> Description is optional.

> Response

```json
{
  "id": 3,
  "author": "Robin Kruijt",
  "user_thumbnail": "http://example.com/example.jpg",
  "image_url": "",
  "rating": 4.8,
  "title": "Prachtig uitzicht",
  "description": "Prachtig uitzicht, heerlijk eten.",
  "post_date": "2015-12-13T19:33:00 +0000",
  "comment_url": "http://example.com",
  "is_user_review": true
}
```

### Request

`POST /api/{version}/actions/{id}/reviews`


## Add review image

Creates an image belonging to the review.

### Request

A multipart request containing an image under fieldname "image".

`POST /api/{version}/actions/{id}/reviews/{id}/images`

## Delete review image

Deletes an image belonging to the review.

### Request

`DELETE /api/{version}/actions/{id}/reviews/{id}/images`

## Update Review

Updates the users review.

```
Request endpoint Acceptatie:
PUT https://rotterdampas-acc.passcloud.nl/rest/putreview/
Request endpoint Productie:
PUT https://rotterdampas.passcloud.nl/rest/putreview/
```

```
Authentication in Acceptatie: Basic authentication with a rest user.
username: wsrest2
password: Intermediad!2
```

```
Authentication in Productie will follow after testing.
```


### Parameters

Parameter | Type | Optional | Default | Description
--------- | ---- | -------- | ------- | -----------
pass_owner_code | string | verplicht | - | The code of the 'Organization'.
api_version | float | verplicht | - | The version number of the API.
pass_type_number | integer | verplicht | - | The number of the 'PasSoort'.
account_login | string | verplicht | - | The inlog name of the Pashouder.
account_password | string | verplicht | - | The password of the Pashouder.
title | string | verplicht | - | The (new) title of the review.
rating | float | verplicht | - | The (new) rating of the review.
review_id | integer | verplicht | - | The id number of the review that the users wants to update.
description | string | optioneel | - | The (new) description of the review.

> Response

```json
{
  "author": "R Janssen",
  "title": "Review #5",
  "is_user_review": true,
  "description": "een groter stukje tekst",
  "rating": 5,
  "id_": 1,
  "post_date": 1444643820634
}
```

### Status code

Code | Description
---- | -----------
400 | One or more mandatory parameters are empty.
400 | Can't find the PasSoort.
401 | Can't find the organization.
401 | Wrong values in the basic authentication.
401 | Can't verify Pashouder.
404 | Can't find the Pashouder.
404 | Can't find a review with the given id that is linked to this user. 
500 | No values in the basic authentication.
200 | Everything is ok.

> Request

```json
{
  "title": "Hallo",
  "description": "Beste dag van mijn leven",
  "rating": 4.8
}
```
> Description is optional.

> Response

```json
{
  "id": 3,
  "author": "Robin Kruijt",
  "user_thumbnail": "http://example.com/example.jpg",
  "image_url": "",
  "rating": 4.8,
  "title": "Prachtig uitzicht",
  "description": "Prachtig uitzicht, heerlijk eten.",
  "post_date": "2015-12-13T19:33:00 +0000",
  "comment_url": "http://example.com",
  "is_user_review": true
}
```

### Request

`PUT /api/{version}/actions/{id}/reviews/{id}`

## Delete Review

### Request

`DELETE /api/{version}/actions/{id}/reviews/{id}`

# User

## Get user actions

Returns the actions consumed by the current user ordered by consumption date.

```
Request endpoint Acceptatie:
GET https://rotterdampas-acc.passcloud.nl/rest/getuseractions/
Request endpoint Productie:
GET https://rotterdampas.passcloud.nl/rest/getuseractions/
```

```
Authentication in Acceptatie: Basic authentication with a rest user.
username: wsrest2
password: Intermediad!2
```

```
Authentication in Productie will follow after testing.
```

### Parameters

Parameter | Type | Optional | Default | Description
--------- | ---- | -------- | ------- | -----------
pass_owner_code | string | verplicht | - | The code of the 'Organization'.
api_version | float | verplicht | - | The version number of the API.
pass_type_number | integer | verplicht | - | The number of the 'PasSoort'.
account_login | string | verplicht | - | The inlog name of the Pashouder.
account_password | string | verplicht | - | The password of the Pashouder.
page | integer | verplicht | - | The result page number.
per_page | integer | verplicht | - | The count of the results per page.
meta | boolean | optioneel | false | If set, the body will not contain the results but just the meta data for this call.
include_family | boolean | optioneel | false | If set, the result will include the actions consumed by the users family.
year | integer | optioneel | - | If included, the results will constrained to the specific “pass year”.

> Response

```json
{
  "page_size": 1,
  "page": 1,
  "get_user_actions_meta_actions": [{
    "has_user_shared_experience": false,
    "consumption_date": 1433376000000,
    "get_user_actions_actions_partners": [{
      "region": "Amsterdam",
      "phone_number": "020 820 8122",
      "zipcode": "1053 RT",
      "street": "Hannie Dankbaarpassage",
      "name": "De Filmhallen",
      "street_number": "12",
      "id_": 10235,
      "url": "http://www.filmhallen.nl",
      "email_address": "bob@themovies.nl"
    }],
    "end_date": 1441058399000,
    "title": "Gratis naar film1",
    "get_user_actions_actions_offers": [	{
		"title": "Titel",
		"percentage": 4.75,
		"amount": 9.95,
	}],
    "short_description": "Gratis naar de Filmhallen",
    "get_user_actions_actions_locations": [	{
		"id_": 545,
		"title": "Amsterdam",
		"street": "straat",
		"zipcode": "4444 ZZ",
		"street_number": 44,
		"region": "Zuid-Holland",
		"latitude": 999999,
		"longitude": 999999, 
	}],
    "pillar": "Cultuur",
    "id_": 557,
    "is_user_wishlist_item": false,
    "start_date": 1422745200000
  }],
  "total_records": 6
}
```

### Status code

Code | Description
---- | -----------
400 | One or more mandatory parameters are empty.
400 | Can't find the PasSoort.
401 | Can't find the organization.
401 | Wrong values in the basic authentication.
401 | Can't verify Pashouder.
404 | Can't find the Pashouder.
404 | Can't find a pass.
404 | Can't find any usage.
500 | No values in the basic authentication.
200 | Everything is ok.

> Reponse

```json
{
  "meta": {
    "page": 0,
    "page_size": 30,
    "total_records": 100
  },
  "actions": [
    {
      "id": 1,
      "title": "Zin van bewegen",
      "pillar": "EtenDrinken",
      "thumbnail": "http://example.com/example.jpg",
      "short_description": "Letterlijk tussen de zee- en binnenvaartschepen ontdek jij het Rotterdamse havengebied. De indrukwekkende skyline glijdt aan je voorbij. En dan werven, dokken en oneindig veel containers…",
      "is_user_wishlist_item": true,
      "consumption_date": "2015-12-09T19:33:00 +0000",
      "has_user_shared_experience": true,
      "start_date":"2015-12-09T19:33:00 +0000",
      "end_date": "2015-12-13T19:33:00 +0000",
      "offers": {
        "type": 1,
        "offer": [
          {
            "title": "Voor 5 euro naar de dierentuin",
            "percentage": 25.0,
            "amount": 23.0
          }
        ]
      },
      "partner": {
        "id": 1,
        "name": "Spido",
        "url": "http://spido.nl",
        "phone_number": "010 42984039",
        "email_address": "example@spido.nl",
        "street": "Botenlaan",
        "zipcode": "2343KJ",
        "street_number": "45",
        "region": "Rotterdam"
      },
      "locations": [
        {
          "id": 1,
          "title": "Ballenbak",
          "street": "Jan Luykenlaan",
          "zipcode": "2343KJ",
          "street_number": "8",
          "region": "Rotterdam",
          "latitude": 4.0000,
          "longitude": 51.0000
        }
      ]
    }
  ]
}
```
### Request

`GET /api/{version}/users/me/actions`

### Query Parameters

Parameter | Optional | Default | Description
--------- | -------- | ------- | -----------
meta | true | false | If set, the body will not contain the results but just the meta data for this call. 
include_family | true | false | If set, the result will include the actions consumed by the users family. 
year | true | - | If included, the results will constrained to the specific "pass year".

## Get current authenticated user

Returns all data specific to the logged in user. Excluding wishlists, devices and savings.

```
Request endpoint Acceptatie:
GET https://rotterdampas-acc.passcloud.nl/rest/getcurrentauthenticateduser/
Request endpoint Productie:
GET https://rotterdampas.passcloud.nl/rest/getcurrentauthenticateduser/
```

```
Authentication in Acceptatie: Basic authentication with a rest user.
username: wsrest2
password: Intermediad!2
```

```
Authentication in Productie will follow after testing.
```

### Parameters

Parameter | Type | Optional | Default | Description
--------- | ---- | -------- | ------- | -----------
pass_owner_code | string | verplicht | - | The code of the 'Organization'.
api_version | float | verplicht | - | The version number of the API.
pass_type_number | integer | verplicht | - | The number of the 'PasSoort'.
account_login | string | verplicht | - | The inlog name of the Pashouder.
account_password | string | verplicht | - | The password of the Pashouder.

> Response

```json
{
  "username": "pietjepuk@gmail.com",
  "currently_active": true,
  "initials": "R",
  "amount_reviews": 6,
  "last_name": "Janssen",
  "passnumber": 2,
  "id_": 174739,
  "amount_actions_consumed": 6
}
```

### Status code

Code | Description
---- | -----------
400 | One or more mandatory parameters are empty.
400 | Can't find the PasSoort.
401 | Can't find the organization.
401 | Wrong values in the basic authentication.
401 | Can't verify Pashouder.
404 | Can't find the Pashouder.
500 | No values in the basic authentication.
500 | Internal server error while creating a response.
200 | Everything is ok.

> Response

```json
{
  "id": 1,
  "pasnumber": 5,
  "username": "timpelgrim",
  "first_name": "Tim",
  "last_name": "Pelgrim",
  "user_thumbnail": "http://example.com/example.jpg",
  "currently_active": true,
  "amount_reviews": 16,
  "amount_actions_consumed": 12
}
```

### Request
`GET /api/{version}/users/me`

## Get user reviews

Returns all reviews for the current authenticated user.

```
Request endpoint Acceptatie:
GET https://rotterdampas-acc.passcloud.nl/rest/getuserreview/
Request endpoint Productie:
GET https://rotterdampas.passcloud.nl/rest/getuserreview/
```

```
Authentication in Acceptatie: Basic authentication with a rest user.
username: wsrest2
password: Intermediad!2
```

```
Authentication in Productie will follow after testing.
```

### Parameters

Parameter | Type | Optional | Default | Description
--------- | ---- | -------- | ------- | -----------
pass_owner_code | string | verplicht | - | The code of the 'Organization'.
api_version | float | verplicht | - | The version number of the API.
pass_type_number | integer | verplicht | - | The number of the 'PasSoort'.
account_login | string | verplicht | - | The inlog name of the Pashouder.
account_password | string | verplicht | - | The password of the Pashouder.
page | integer | verplicht | - | The result page number.
per_page | integer | verplicht | - | The count of the results per page.

> Response

```json
{
  "meta_get_user_reviews": [{
    "author": "pietjepuk@gmail.com",
    "title": "Title",
    "is_user_review": true,
    "reviews_get_user_reviews_actions": [{"id_": 270}],
    "description": "Een stukje tekst",
    "rating": 4,
    "id_": 3,
    "post_date": 1444727339056
  }],
  "page_size": 1,
  "average_review": 4.716666666666667,
  "page": 1,
  "total_records": 6
}
```

### Status code

Code | Description
---- | -----------
400 | One or more mandatory parameters are empty.
400 | Can't find the PasSoort.
401 | Can't find the organization.
401 | Wrong values in the basic authentication.
401 | Can't verify Pashouder.
404 | Can't find the Pashouder.
404 | Can't find any reviews for this user.
500 | No values in the basic authentication.
200 | Everything is ok.

> Response

```json
{
  "meta": {
    "page": 0,
    "page_size": 30,
    "total_records": 4000,
    "average_review": 5.4
  },
  "reviews": [
    {
      "id": 3,
      "author": "Robin Kruijt",
      "user_thumbnail": "http://example.com/example.jpg",
      "image_url": "http://example.com",
      "rating": 4.8,
      "title": "Prachtig uitzicht",
      "description": "Prachtig uitzicht, heerlijk eten.",
      "post_date": "2015-12-13T19:33:00 +0000",
      "comment_url": "http://example.com",
      "is_user_review": true,
      "action": {
        "id": 1
      }
    }
  ]
}
```
> The action field includes just an ID, but could include more data if the view requires it. (To be discussed)

### Request
`GET /api/{version}/users/me/reviews`

## Get savings

> Response

```json
{
  "meta": {
    "page": 0,
    "page_size": 30,
    "total_savings": 23.48
  },
  "savings":[
    {
      "id": 1,
      "date_of_use": "2015-12-09T19:33:00 +0000",
      "actual_savings": 12.30,
      "calculated_savings": 12.50,
      "action":{
        "id": 1,
        "title": "Zin van bewegen",
        "pillar": "EtenDrinken",
        "thumbnail": "http://example.com/example.jpg",
        "short_description": "Letterlijk tussen de zee- en binnenvaartschepen ontdek jij het Rotterdamse havengebied. De indrukwekkende skyline glijdt aan je voorbij. En dan werven, dokken en oneindig veel containers…",
        "is_user_wishlist_item": true,
        "has_user_consumed_action": true,
        "has_user_shared_experience": true,
        "start_date":"2015-12-09T19:33:00 +0000",
        "end_date": "2015-12-13T19:33:00 +0000",
        "offers": {
          "type": 1,
          "offer": [
            {
              "title": "Voor 5 euro naar de dierentuin",
              "percentage": 25.0,
              "amount": 23.0
            }
          ]
        },
        "partner": {
          "id": 1,
          "name": "Spido",
          "url": "http://spido.nl",
          "phone_number": "010 42984039",
          "email_address": "example@spido.nl",
          "street": "Botenlaan",
          "zipcode": "2343KJ",
          "street_number": "45",
          "region": "Rotterdam"
        },
        "locations": [
          {
            "id": 1,
            "title": "Ballenbak",
            "street": "Jan Luykenlaan",
            "zipcode": "2343KJ",
            "street_number": "8",
            "region": "Rotterdam",
            "latitude": 4.0000,
            "longitude": 51.0000
          }
        ]
      }
    }
  ]
}
```

Returns all savings for the current authenticated user.

### Request
`GET /api/{version}/users/me/savings`

### Query parameters
Parameter | Optional | Default | Description
--------- | -------- | ------- | -----------
include_family | true | false | If set, the result will include the savings by the users family.

## Add saving

A user can add a finished "Actie" and its savings manually.

> Request

```json
{
  "action_id": 1,
  "amount": 23.00,
  "date_of_use": "2015-12-09T19:33:00 +0000"
}
```

> Response

```json
{
  "id": 1,
  "date_of_use": "2015-12-09T19:33:00 +0000",
  "actual_savings": 23.00,
  "calculated_savings": 23.00,
  "action":{
    "id": 1,
    "title": "Zin van bewegen",
    "pillar": "EtenDrinken",
    "thumbnail": "http://example.com/example.jpg",
    "short_description": "Letterlijk tussen de zee- en binnenvaartschepen ontdek jij het Rotterdamse havengebied. De indrukwekkende skyline glijdt aan je voorbij. En dan werven, dokken en oneindig veel containers…",
    "is_user_wishlist_item": true,
    "has_user_consumed_action": true,
    "has_user_shared_experience": true,
    "start_date":"2015-12-09T19:33:00 +0000",
    "end_date": "2015-12-13T19:33:00 +0000",
    "offers": {
      "type": 1,
      "offer": [
        {
          "title": "Voor 5 euro naar de dierentuin",
          "percentage": 25.0,
          "amount": 23.0
        }
      ]
    },
    "partner": {
      "id": 1,
      "name": "Spido",
      "url": "http://spido.nl",
      "phone_number": "010 42984039",
      "email_address": "example@spido.nl",
      "street": "Botenlaan",
      "zipcode": "2343KJ",
      "street_number": "45",
      "region": "Rotterdam"
    },
    "locations": [
      {
        "id": 1,
        "title": "Ballenbak",
        "street": "Jan Luykenlaan",
        "zipcode": "2343KJ",
        "street_number": "8",
        "region": "Rotterdam",
        "latitude": 4.0000,
        "longitude": 51.0000
      }
    ]
  }
}
```


### Request
`POST /api/{version}/users/me/savings`

## Update saving

A user can update a finished "Actie" and its savings manually.

> Request

```json
{
  "action_id": 1,
  "amount": 23.00,
  "date_of_use": "2015-12-09T19:33:00 +0000"
}
```
> All fields are optional.

> Response

```json
{
  "id": 1,
  "date_of_use": "2015-12-09T19:33:00 +0000",
  "actual_savings": 23.00,
  "calculated_savings": 23.00,
  "action":{
    "id": 1,
    "title": "Zin van bewegen",
    "pillar": "EtenDrinken",
    "thumbnail": "http://example.com/example.jpg",
    "short_description": "Letterlijk tussen de zee- en binnenvaartschepen ontdek jij het Rotterdamse havengebied. De indrukwekkende skyline glijdt aan je voorbij. En dan werven, dokken en oneindig veel containers…",
    "is_user_wishlist_item": true,
    "has_user_consumed_action": true,
    "has_user_shared_experience": true,
    "start_date":"2015-12-09T19:33:00 +0000",
    "end_date": "2015-12-13T19:33:00 +0000",
    "offers": {
      "type": 1,
      "offer": [
        {
          "title": "Voor 5 euro naar de dierentuin",
          "percentage": 25.0,
          "amount": 23.0
        }
      ]
    },
    "partner": {
      "id": 1,
      "name": "Spido",
      "url": "http://spido.nl",
      "phone_number": "010 42984039",
      "email_address": "example@spido.nl",
      "street": "Botenlaan",
      "zipcode": "2343KJ",
      "street_number": "45",
      "region": "Rotterdam"
    },
    "locations": [
      {
        "id": 1,
        "title": "Ballenbak",
        "street": "Jan Luykenlaan",
        "zipcode": "2343KJ",
        "street_number": "8",
        "region": "Rotterdam",
        "latitude": 4.0000,
        "longitude": 51.0000
      }
    ]
  }
}
```


### Request
`PUT /api/{version}/users/me/savings`


## Delete saving

A user can delete a finished "Actie" and its savings manually.


### Request
`DELETE /api/{version}/users/me/savings/{id}`

# Wishlists

## Get Wishlists

Returns all wishlists including a few of their actions.

```
Request endpoint Acceptatie:
GET https://rotterdampas-acc.passcloud.nl/rest/getwishlists/
Request endpoint Productie:
GET https://rotterdampas.passcloud.nl/rest/getwishlists/
```

```
Authentication in Acceptatie: Basic authentication with a rest user.
username: wsrest2
password: Intermediad!2
```

```
Authentication in Productie will follow after testing.
```

### Parameters

Parameter | Type | Optional | Default | Description
--------- | ---- | -------- | ------- | -----------
pass_owner_code | string | verplicht | - | The code of the 'Organization'.
api_version | float | verplicht | - | The version number of the API.
pass_type_number | integer | verplicht | - | The number of the 'PasSoort'.
account_login | string | verplicht | - | The inlog name of the Pashouder.
account_password | string | verplicht | - | The password of the Pashouder.
page | integer | verplicht | - | The result page number.
per_page | integer | verplicht | - | The count of the results per page.
amount_actions | boolean | optioneel | false | If set, returns this amount of actions per wishlist.

> Response

```json
{
   "meta_get_wishlists":    [   {
         "title": "6666",
         "id_": 1,
         "get_wishlists_wishlists_action_amount": [],
         "get_wishlists_wishlists_actions": [         {
            "has_user_shared_experience": true,
            "get_wishlists_actions_locations": [	{
				"id_": 512,
				"title": "Titel",
				"street": "Hoofdstraat",
				"zipcode": "3501 BC",
				"street_number": "18",
				"region": "Amsterdam",
				"longitude": 57.00120155,
				"latitude": 12.25369844,
			}],
            "get_wishlists_actions_offers": [	{
				"title": "Titel",
				"percentage": 4.75,
				"amount": 9.95,
			}],
            "title": "Stadspasje Aanb. A",
            "end_date": 1441058399000,
            "short_description": "Stadspas Aanbieding 100% Korting",
            "pillar": "Cultuur",
            "id_": 30274,
            "get_wishlists_actions_partners": [            {
               "region": "Amsterdam",
               "phone_number": "020 763 0599",
               "zipcode": "1082 ME",
               "street": "G. Mahlerlaan",
               "name": "Amsterdam Expo BV",
               "street_number": "106",
               "id_": 10234,
               "url": "http://www.amsterdamexpo.nl",
               "email_address": "arnold.vandewater@terminal1.nl"
            }],
            "is_user_wishlist_item": true,
            "has_user_consumed_action": false,
            "start_date": 1422745200000
         }]
      }],
   "page_size": 1,
   "page": 1,
   "total_records": 3
}
```

### Status code

Code | Description
---- | -----------
400 | One or more mandatory parameters are empty.
400 | Can't find the PasSoort.
401 | Can't find the organization.
401 | Wrong values in the basic authentication.
401 | Can't verify Pashouder.
404 | Can't find the Pashouder.
404 | Can't find any wishlists that are connected to the user.
500 | No values in the basic authentication.
200 | Everything is ok.

> Response

```json
{
  "meta": {
    "page": 0,
    "page_size": 30,
    "total_records": 1200
  },
  "wishlists":[
    {
      "id": 1,
      "title": "Zomervakantie",
      "actions": [
        {
          "id": 1,
          "title": "Zin van bewegen",
          "pillar": "EtenDrinken",
          "thumbnail": "http://example.com/example.jpg",
          "short_description": "Letterlijk tussen de zee- en binnenvaartschepen ontdek jij het Rotterdamse havengebied. De indrukwekkende skyline glijdt aan je voorbij. En dan werven, dokken en oneindig veel containers…",
          "is_user_wishlist_item": true,
          "has_user_consumed_action": true,
          "has_user_shared_experience": true,
          "start_date":"2015-12-09T19:33:00 +0000",
          "end_date": "2015-12-13T19:33:00 +0000",
          "offers": {
            "type": 1,
            "offer": [
              {
                "title": "Voor 5 euro naar de dierentuin",
                "percentage": 25.0,
                "amount": 23.0
              }
            ]
          },
          "partner": {
            "id": 1,
            "name": "Spido",
            "url": "http://spido.nl",
            "phone_number": "010 42984039",
            "email_address": "example@spido.nl",
            "street": "Botenlaan",
            "zipcode": "2343KJ",
            "street_number": "45",
            "region": "Rotterdam"
          },
          "locations": [
            {
              "id": 1,
              "title": "Ballenbak",
              "street": "Jan Luykenlaan",
              "zipcode": "2343KJ",
              "street_number": "8",
              "region": "Rotterdam",
              "latitude": 4.0000,
              "longitude": 51.0000
            }
          ]
        }
      ]
    }
  ]
}

```

### Request

`GET /api/{versions}/users/me/wishlists`

### Query Parameters

Parameter | Optional | Default | Description
--------- | -------- | ------- | -----------
amount_actions | true | 3 | If set, returns this amount of actions per wishlist.


## Get Wishlist

Returns a specific wislist and all of its actions.

```
Request endpoint Acceptatie:
GET https://rotterdampas-acc.passcloud.nl/rest/getwishlist/
Request endpoint Productie:
GET https://rotterdampas.passcloud.nl/rest/getwishlist/
```

```
Authentication in Acceptatie: Basic authentication with a rest user.
username: wsrest2
password: Intermediad!2
```

```
Authentication in Productie will follow after testing.
```

### Parameters

Parameter | Type | Optional | Default | Description
--------- | ---- | -------- | ------- | -----------
pass_owner_code | string | verplicht | - | The code of the 'Organization'.
api_version | float | verplicht | - | The version number of the API.
pass_type_number | integer | verplicht | - | The number of the 'PasSoort'.
account_login | string | verplicht | - | The inlog name of the Pashouder.
account_password | string | verplicht | - | The password of the Pashouder.
page | integer | verplicht | - | The result page number.
per_page | integer | verplicht | - | The count of the results per page.
id_ | integer | verplicht | - | ID number of the specific action.

> Response

```json
{
   "title": "Wenslijst #1",
   "get_wishlist_actions_actions": [   {
      "get_wishlist_actions_partners": [      {
         "region": "Amsterdam",
         "phone_number": "020 763 0599",
         "zipcode": "1082 ME",
         "street": "G. Mahlerlaan",
         "name": "Amsterdam Expo BV",
         "street_number": "106",
         "id_": 10234,
         "url": "http://www.amsterdamexpo.nl",
         "email_address": "arnold.vandewater@terminal1.nl"
      }],
      "has_user_shared_experience": true,
      "title": "Stadspasje Aanb. A",
      "end_date": 1441058399000,
      "short_description": "Stadspas Aanbieding 100% Korting",
      "get_wishlist_actions_offers": [	{
		"title": "Titel",
		"percentage": 4.75,
		"amount": 9.95,
	  }],
      "pillar": "Cultuur",
      "id_": 270,
      "is_user_wishlist_item": true,
      "has_user_consumed_action": false,
      "start_date": 1422745200000,
      "get_wishlist_actions_locations": [	{
		"id_": 15,
		"title": "Titel",
		"street": "Hoofdstraat",
		"zipcode": "3501 BC",
		"street_number": "18",
		"region": "Amsterdam",
		"latitude": 57.0001254,
		"longitude": 1.20498867,
	  }]
   }],
   "id_": 1
}
```

### Status code

Code | Description
---- | -----------
400 | One or more mandatory parameters are empty.
400 | Can't find the PasSoort.
401 | Can't find the organization.
401 | Wrong values in the basic authentication.
401 | Can't verify Pashouder.
404 | Can't find the Pashouder.
404 | Can't find a wishlist related to the parameter 'id_'.
500 | No values in the basic authentication.
200 | Everything is ok.

> Response

```json
{
  "id": 1,
  "title": "Zomervakantie",
  "actions": [
    {
      "id": 1,
      "title": "Zin van bewegen",
      "pillar": "EtenDrinken",
      "thumbnail": "http://example.com/example.jpg",
      "short_description": "Letterlijk tussen de zee- en binnenvaartschepen ontdek jij het Rotterdamse havengebied. De indrukwekkende skyline glijdt aan je voorbij. En dan werven, dokken en oneindig veel containers…",
      "is_user_wishlist_item": true,
      "has_user_consumed_action": true,
      "has_user_shared_experience": true,
      "start_date":"2015-12-09T19:33:00 +0000",
      "end_date": "2015-12-13T19:33:00 +0000",
      "offers": {
        "type": 1,
        "offer": [
          {
            "title": "Voor 5 euro naar de dierentuin",
            "percentage": 25.0,
            "amount": 23.0
          }
        ]
      },
      "partner": {
        "id": 1,
        "name": "Spido",
        "url": "http://spido.nl",
        "phone_number": "010 42984039",
        "email_address": "example@spido.nl",
        "street": "Botenlaan",
        "zipcode": "2343KJ",
        "street_number": "45",
        "region": "Rotterdam"
      },
      "locations": [
        {
          "id": 1,
          "title": "Ballenbak",
          "street": "Jan Luykenlaan",
          "zipcode": "2343KJ",
          "street_number": "8",
          "region": "Rotterdam",
          "latitude": 4.0000,
          "longitude": 51.0000
        }
      ]
    }
  ]
}
```

### Request
`GET /api/{versions}/users/me/wishlists/{id}`

## Post Wishlist

Creates a new wishlist.

```
Request endpoint Acceptatie:
POST https://rotterdampas-acc.passcloud.nl/rest/postwishlist/
Request endpoint Productie:
POST https://rotterdampas.passcloud.nl/rest/postwishlist/
```

```
Authentication in Acceptatie: Basic authentication with a rest user.
username: wsrest2
password: Intermediad!2
```

```
Authentication in Productie will follow after testing.
```

### Parameters

Parameter | Type | Optional | Default | Description
--------- | ---- | -------- | ------- | -----------
pass_owner_code | string | verplicht | - | The code of the 'Organization'.
api_version | float | verplicht | - | The version number of the API.
pass_type_number | integer | verplicht | - | The number of the 'PasSoort'.
account_login | string | verplicht | - | The inlog name of the Pashouder.
account_password | string | verplicht | - | The password of the Pashouder.
title | string | verplicht | - | The title of the new wishlist.

> Response

```json
{
  "title": "wishlist#1",
  "id_": 1,
  "post_wishlist_response_actions": []
}
```

### Status code

Code | Description
---- | -----------
400 | One or more mandatory parameters are empty.
400 | Can't find the PasSoort.
401 | Can't find the organization.
401 | Wrong values in the basic authentication.
401 | Can't verify Pashouder.
404 | Can't find the Pashouder.
500 | No values in the basic authentication.
200 | Everything is ok.

> Request

```json
{
  "title": "Zomervakantie"
}
```

> Response

```json
{
  "id": 1,
  "title": "Zomervakantie",
  "actions": []
}
```

### Request
`POST /api/{versions}/users/me/wishlists`

## Update Wishlist

Updates the wishlist and its parameters. 

```
Request endpoint Acceptatie:
PUT https://rotterdampas-acc.passcloud.nl/rest/putwishlist/
Request endpoint Productie:
PUT https://rotterdampas.passcloud.nl/rest/putwishlist/
```

```
Authentication in Acceptatie: Basic authentication with a rest user.
username: wsrest2
password: Intermediad!2
```

```
Authentication in Productie will follow after testing.
```

### Parameters

Parameter | Type | Optional | Default | Description
--------- | ---- | -------- | ------- | -----------
pass_owner_code | string | verplicht | - | The code of the 'Organization'.
api_version | float | verplicht | - | The version number of the API.
pass_type_number | integer | verplicht | - | The number of the 'PasSoort'.
account_login | string | verplicht | - | The inlog name of the Pashouder.
account_password | string | verplicht | - | The password of the Pashouder.
title | string | verplicht | - | The new title of the wishlist.
id_ | integer | verplicht | - | The id number of the wishlist.

> Response

```json
{
  "title": "Een nieuwe titel",
  "id_": 1,
  "put_wishlist_response_actions": []
}
```

### Status code

Code | Description
---- | -----------
400 | One or more mandatory parameters are empty.
400 | Can't find the PasSoort.
401 | Can't find the organization.
401 | Wrong values in the basic authentication.
401 | Can't verify Pashouder.
404 | Can't find the Pashouder.
404 | Can't find a wishlist with the given id.
500 | No values in the basic authentication.
200 | Everything is ok.

> Request

```json
{
  "title": "Zomervakantie"
}
```

> Response

```json
{
  "id": 1,
  "title": "Zomervakantie",
  "actions": []
}
```

### Request
`PUT /api/{versions}/users/me/wishlists/{id}`

## Add action to wishlist

Adds an action to a wishlist

```
Request endpoint Acceptatie:
POST https://rotterdampas-acc.passcloud.nl/rest/postactiontowishlist/
Request endpoint Productie:
POST https://rotterdampas.passcloud.nl/rest/postactiontowishlist/
```

```
Authentication in Acceptatie: Basic authentication with a rest user.
username: wsrest2
password: Intermediad!2
```

```
Authentication in Productie will follow after testing.
```

### Parameters

Parameter | Type | Optional | Default | Description
--------- | ---- | -------- | ------- | -----------
pass_owner_code | string | verplicht | - | The code of the 'Organization'.
api_version | float | verplicht | - | The version number of the API.
pass_type_number | integer | verplicht | - | The number of the 'PasSoort'.
account_login | string | verplicht | - | The inlog name of the Pashouder.
account_password | string | verplicht | - | The password of the Pashouder.
action_id | integer | verplicht | - | The id number of the action.
wishlist_id | integer | verplicht | - | The is number of the wishlist.

> Response: Only returns a status code.

### Status code

Code | Description
---- | -----------
400 | One or more mandatory parameters are empty.
400 | Can't find the PasSoort.
401 | Can't find the organization.
401 | Wrong values in the basic authentication.
401 | Can't verify Pashouder.
404 | Can't find the Pashouder.
404 | Can't find a wishlist for the given id.
404 | Can't find an action for the given id.
500 | No values in the basic authentication.
200 | Everything is ok.

> Request

```json
{
  "action_id": 1
}
```

### Request

`POST /api/{versions}/users/me/wishlists/{id}`

## Delete Wishlist

Deletes an authenticated users wishlist.

### Request
`DELETE /api/{versions}/users/wishlists/{id}`

## Remove action from wishlist

Removes an action from a wishlist

```
Request endpoint Acceptatie:
DELETE https://rotterdampas-acc.passcloud.nl/rest/deleteactiontowishlist/
Request endpoint Productie:
DELETE https://rotterdampas.passcloud.nl/rest/deleteactiontowishlist/
```

```
Authentication in Acceptatie: Basic authentication with a rest user.
username: wsrest2
password: Intermediad!2
```

```
Authentication in Productie will follow after testing.
```

### Parameters

Parameter | Type | Optional | Default | Description
--------- | ---- | -------- | ------- | -----------
pass_owner_code | string | verplicht | - | The code of the 'Organization'.
api_version | float | verplicht | - | The version number of the API.
pass_type_number | integer | verplicht | - | The number of the 'PasSoort'.
account_login | string | verplicht | - | The inlog name of the Pashouder.
account_password | string | verplicht | - | The password of the Pashouder.
action_id | integer | verplicht | - | The id number of the action.
wishlist_id | integer | verplicht | - | The is number of the wishlist.

> Response: Only returns a status code.

### Status code

Code | Description
---- | -----------
400 | One or more mandatory parameters are empty.
400 | Can't find the PasSoort.
401 | Can't find the organization.
401 | Wrong values in the basic authentication.
401 | Can't verify Pashouder.
404 | Can't find the Pashouder.
404 | Can't find a wishlist for the given id.
404 | Can't find an action for the given id.
404 | Can't find the wishlist item.
500 | No values in the basic authentication.
200 | Everything is ok.

> Request

```json
{
  "action_id": 1
}
```

### Request

`DELETE /api/{versions}/users/me/wishlists/{id}`

# Passes

## Get Passes

Returns all the passes of the current user.

```
Request endpoint Acceptatie:
GET https://rotterdampas-acc.passcloud.nl/rest/getpasses/
Request endpoint Productie:
GET https://rotterdampas.passcloud.nl/rest/getpasses/
```

```
Authentication in Acceptatie: Basic authentication with a rest user.
username: wsrest2
password: Intermediad!2
```

```
Authentication in Productie will follow after testing.
```

### Parameters

Parameter | Type | Optional | Default | Description
--------- | ---- | -------- | ------- | -----------
pass_owner_code | string | verplicht | - | The code of the 'Organization'.
api_version | float | verplicht | - | The version number of the API.
pass_type_number | integer | verplicht | - | The number of the 'PasSoort'.
account_login | string | verplicht | - | The inlog name of the Pashouder.
account_password | string | verplicht | - | The password of the Pashouder.
page | integer | verplicht | - | The result page number.
per_page | integer | verplicht | - | The count of the results per page.
include_family | boolean | optioneel | false | If set, the result will include the passes of the users family.

> Response

```json
{
  "page_size": 20,
  "page": 1,
  "get_passes_meta_passes": [
    {
      "end_date": 1451606399000,
      "owner_name": "K. Janssen",
      "active": true,
      "number": 6.011011329443E12,
      "id_": 175310
    },
    {
      "end_date": 1483228799000,
      "owner_name": "R Janssen",
      "active": true,
      "number": 6.011011325987E12,
      "id_": 175309
    }
  ],
  "total_records": 2
}
```

### Status code

Code | Description
---- | -----------
400 | One or more mandatory parameters are empty.
400 | Can't find the PasSoort.
401 | Can't find the organization.
401 | Wrong values in the basic authentication.
401 | Can't verify Pashouder.
404 | Can't find the Pashouder.
500 | No values in the basic authentication.
200 | Everything is ok.

### Request
 
`GET /api/{version}/users/me/passes`
 
### Query Parameters

Parameter | Optional | Default | Description
--------- | -------- | ------- | -----------
include_family | true | false | If set, includes the passes of the users family.

> Response

```json
{
  "meta": {
    "page": 0,
    "page_size": 20,
    "total_records": 5
  },
  "passes":[
    {
      "id": 1,
      "owner_name": "N. de Vries",
      "number": 086951,
      "image": "wwww.example.com/image.png",
      "active": true,
      "end_date": "2015-12-13T19:33:00 +0000"
    }
  ]
}
```
