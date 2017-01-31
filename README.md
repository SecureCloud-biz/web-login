# Truecaller Sign Up for Web Apps

## Preconditions

- You have a web application that allow users to sing up/sign in to use your services
- You have identified the benefits that Truecaller can provide you, by allowing your users one click sign up with their Truecaller profile
- You have set up a callback endpoint, that we will use to post the access token, once the user has approved your app to use their Truecaller profile
- This documentation to efficiently pass through all the steps of the process.

## Get started

### Already a Truecaller Developer?

- Login to your account to create a new app. Add your App Name and the Callback URL, where we'll post your access token. Make sure you follow the [guidelines for the CallbackURL](#guidelines-for-the-callbackurl).
- Once we've got the info about your account and your app, we will provide you with a unique "appKey" for that application. You'll use this key in the header, for us to be able to authorize your requests.

### New to Truecaller Devs?

To ensure the authenticity of the interactions between your app and Truecaller, you need to sign up for an account and add the information about your application.

- Add your email, password, and personal info to create the account. These will be your credentials to login to your Truecaller Developer account.
- Now add your Application. Insert your App Name, your App Domain and the Callback URL, where we'll post the access tokens. Make sure you follow the [guidelines for the CallbackURL](#guidelines-for-the-callbackurl).

Once we've got the info about your account and your app, we will provide you with a unique "appKey" for that application.

## So How Can Truecaller Users Sign Up Quickly to Your App?

This is how it looks in a glance:

![Diagram](https://raw.githubusercontent.com/truecaller/web-signin/master/documentation/images/diagram.png)

But let's get down to the details.


### Ask for User's Number

When using Truecaller Sign Up, the only thing the user will need to insert in your app, is their phone number (e164 phone number format without the "+" sign). You can apply a basic validation to the number to ensure maximum success of the request (ex. use open source library such as the google libphonenumber, or if your users are located in one country only, just use the validation rules for mobile numbers for that country (just contact us and we'll provide you with all the input you need).

Once the user has inserted their number, your backend services should trigger the following request to our API:

**Endpoint:**  
https://api4.truecaller.com/v1/apps/requests

**Method:**  
POST

**Header parameters:**

| **Parameter [Type]** | **Required** | **Description**         | **Example**                  |
| -------------------  | ------------ | ----------------------- | ---------------------------- |
| appKey [String]      | yes          | Applications secret key | 9493249349-0944994595odmdnri |

**Body parameters:**

| **Parameter [Type]** | **Required** | **Description**                                                     | **Example**              |
| -------------------  | ------------ | ------------------------------------------------------------------- | ------------------------ |
| phoneNumber [Long]   | yes          | User's phone number                                                 | 46761234567              |
| state [String]       | no           | Optional parameter that will be send back to partner's callbak url. | ne4_cRtzx73-alui_XDvzS5h |

**Initiate User Authorization**  
```bash
curl -X POST -H "Content-Type: application/json" -H "appKey: Js9t3518a9eb2b6804160957f7b438dcc01ac" -H "Cache-Control: no-cache" -d '{
  "phoneNumber": 46760369715
}' "https://api4.truecaller.com/v1/apps/requests"
```

The **"state"** param is an optional param that you might wanna generate as a session id, once the user triggers the authorization with Truecaller. The same param will be routed back to you along with the access token to your callback endpoint, which will help you complete the session.

Alternatively, you can create and keep a server-side session, using the **requestId** you get in the response.

**Response**

A successful request (200 response code), means your request is accepted and we'll ask the holder of the number to approve signing up with his Truecaller profile.

In the response, you'll get the corresponding requestID back.

```json
{
  "requestId": "92849748hueh9"
}
```

You can use this requestID to match it against an access token you'll get and complete the user sign up.

In case of failed request, the response codes in return are:

- 404 Not Found - **means your credentials (appKey) are not valid.**
- 5xx Server error - **any other error**

Note: Once the user triggers the authorization from your app, it would be good to lock the behavior for a certain period of time (5 mins). The reason is to prevent multiple unnecessary requests in a short period of time towards our platform, and allowing proper completion of the cycle (request authorization, authorize by the user, fetch profile).

### Fetch User Profile

Once the user approves the sign up to your app with their Truecaller profile, we'll immediately post the accessToken and the requestID to your Callback endpoint. To be able to fetch the user's profile information you should use the following endpoint.

**Endpoint:**  
https://profile4.truecaller.com/v1/default

**Header Authorization Parameters:**  

| **Parameter [Type]** | **Required** | **Description**  | **Example**                             |
| -------------------  | ------------ | ---------------- | --------------------------------------- |
| Authorization        | yes          | Bearer {token}   | Bearer WcB~aSJYbCr5yla5z0CdAGfyj3Rruk~8 |

**Get User Profile**  
```bash
curl -X GET -H "Authorization: Bearer a3sAB0KnGANg4VZwIXfhUyFmPbzoONofl4FjIItac0JQSODp6niW8oBr33uOI-u7" -H "Cache-Control: no-cache" "https://profile4.truecaller.com/v1/default"
```

**Response**

```json
{
 "name": {
  "first": "first_name_field_value3",
  "last": "last_name_field_value",
 },
 "aboutMe": "status_message_field_value",
 "phoneNumbers": [46795087665],
 "onlineIdentity": {
  "facebookId": "facebook_id_field_value",
  "twitterId": "twitter_id_field_value",
  "email": "miroznak@yandex.ru",
  "url": "url_field_value"
 },
 "companyName": "w_company_name_field_value",
 "jobTitle": "w_title_field_value",
 "addresses": [{
  "countryCode": "se",
  "city": "city_field_value",
  "street": "street_field_value",
  "zipcode": "1234567"
 }],
 "gender": "Female"
}
```

**Response Codes**

- 200 OK
- 401 Unauthorized - **If your credentials are not valid**
- 5xx Server error - **Any other error**

## Guidelines for the CallbackURL

The CallbackURL should correspond to an endpoint where we will post the access token for you to fetch the user's profile. Every access token can be used to fetch the profile only of the related user granting the authorization to your app. The access token has a time-to-live (10 minutes) and if not used within the TTL, the user needs to re-trigger the authorization process from the beginning.

When setting up the service, please consider the following:

**Method:**  
All access token requests will be submitted as POST request

**Security:**  
To ensure security and privacy, HTTPS should be used.

**Request Params:**

| **Param [String]**   | **Mandatory** | **Description**                                                                      | **Example value**                                                 |
| -------------------- | ------------- | ------------------------------------------------------------------------------------ | ----------------------------------------------------------------- |
| requestId [String]   | yes           |                                                                                      | vXbyFPwqiCAHZyxAldA9M9DDXKk=                                      |
| accessToken [String] | yes           |                                                                                      | a3sAB0KnGANg4VZwIXfhUyFmPbzoONofl4FjIItac0JQSODp6niW8oBr33uOI-u7  |
| state [String]       | no            | This parameter will exist in json if it is sent in "ask for user's number" endpoint. | ne4_cRtzx73-alui_XDvzS5h                                          |

**Expected Response Codes:**

- 200 OK
- 500 - any other error type that will retrigger retry on our side