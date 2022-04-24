# OAUTH LAB

In this lab you will learn how to configure your API to use one variant of OAuth, in wich you send a bearer token to a service.
You will verify this token in three ways:

- The token is generatied by the correct auothority.
- The token is meant for the API (uses th correct audience)
- The token contains the correct claim/role.

The Oauth-endpoint is an Azure AD endpoint belong to the course leader. This endpoint will not work beyond this course.

## The steps

1. [Call the OAUTH endpoint](#call-the-oauth-endpoint) to get a bearer token.
2. [Analyze the token](#analyze-the-token).
3. [Create a new API](#create-a-new-api) and protect it using policies.
4. Call the API with the token to verify the setup.

## Call the OAUTH endpoint

First you need to authenticate to get a token. This token can then be used in all subsequent calls. If you are using [Postman](https://www.postman.com/downloads/), or [Thunder Client](https://marketplace.visualstudio.com/items?itemName=rangav.vscode-thunder-client), you can use [this collection](Session%202.postman_collection.json).

If not, you are on your own.

- Import the collection. When you are done, you should have a new collection called APIm OAuth Demo.
- You need to update a collection variable. Find the collection variables and enter the name of your APIm instance under `YourAPImInstanceName`. Make sure it is in the current value.
- The other settings are the identifier (userid) and client secret (password) for getting a token.
- When you have updated the setting, make sure to save. Then click Send
- If you get back a 200 OK, you are set. Postman will update a variable to contain the token, and use it in later calls.

## Analyze the token

You need to make sure the token is correct and usable.

- Open a brower to [https://jwt.ms/](https://jwt.ms/).
- Go back to Postman and copy the `access_token` value. Make sure you get the whole token and exclude the "-signs.
- Paste the token into the upper textbox in the site. The site will decode the token for you, making it easier to read.
- The decoded token should look like this:
```JSON
{
  "typ": "JWT",
  "alg": "RS256",
  "kid": "jS1Xo1OWDj_52vbwGNgvQO2VzMc"
}.{
  "aud": "cc565aa4-a1e5-4c05-b1cf-df19ca4c95be",
  "iss": "https://login.microsoftonline.com/26ef3687-262a-44fa-95f4-208111b732e9/v2.0",
  "iat": 1650818803,
  "nbf": 1650818803,
  "exp": 1650822703,
  "aio": "E2ZgYNC/92TXIfdAHaXPcRdf91yZBAA=",
  "azp": "a7364de7-ac75-4be9-93b3-895053c0a9e5",
  "azpacr": "1",
  "oid": "c64b33a3-8f65-49f6-a736-6c4a2a87f24d",
  "rh": "0.AU8AhzbvJiom-kSV9CCBEbcy6aRaVszloQVMsc_fGcpMlb5PAAA.",
  "roles": [
    "Employees.Write",
    "Employees.Read"
  ],
  "sub": "c64b33a3-8f65-49f6-a736-6c4a2a87f24d",
  "tid": "26ef3687-262a-44fa-95f4-208111b732e9",
  "uti": "lcmiOtmW80GyOumGX0ueAA",
  "ver": "2.0"
}.[Signature]
```

- Verify:
  - `aud` equals `cc565aa4-a1e5-4c05-b1cf-df19ca4c95be`
  - `iss` equals `https://login.microsoftonline.com/26ef3687-262a-44fa-95f4-208111b732e9/v2.0`
  - `roles` contains `Employees.Read`

These are the parts we will be using to authorize the call.

## Create a new API

You will create the new API by using the built in Import OpenAPI functionality in APIm. The OpenAPI definition is located [here](Oauth%20Demo.openapi+json.json). It will create a new API with one single operation.

- In the portal, find the APIs listing in the left menu.
- Start adding a new API by clicking `+ Add API` at the top of the APIs list.
- In the right pane, look for `Create from defintion` and click `OpenAPI`.
- Click 