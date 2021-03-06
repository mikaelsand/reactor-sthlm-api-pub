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
3. [Create a new API](#create-a-new-api)
4. [Configure the API](#configure-the-api) and protect it using policies.
5. [Call the API](#call-the-api) with the token to verify the setup.

Lastly you have some additional exercies.

## Call the OAUTH endpoint

First you need to authenticate to get a token. This token can then be used in all subsequent calls. If you are using [Postman](https://www.postman.com/downloads/), or [Thunder Client](https://marketplace.visualstudio.com/items?itemName=rangav.vscode-thunder-client), you can use [this collection]([Session%202.postman_collection.json](https://raw.githubusercontent.com/mikaelsand/reactor-sthlm-api-pub/main/Session2/Lab3/OAuth.postman_collection.json)).

If not, you are on your own.

- Import the collection. When you are done, you should have a new collection called APIm OAuth Demo.
- You need to update a collection variable. Find the collection variables and enter the name of your APIm instance under `YourAPImInstanceName`. Make sure it is in the current value.
- The other settings are the identifier (userid) and client secret (password) for getting a token.
- When you have updated the setting, make sure to save.
- Open the `Get Token` request and click Send.
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

You will create the new API by using the built in Import OpenAPI functionality in APIm. It will create a new API with one single operation.

If you want to know more about open API you can find it [here](Oauth%20Demo.openapi+json.json).

- In the portal, find the APIs listing in the left menu.
- Start adding a new API by clicking `+ Add API` at the top of the APIs list.
- In the right pane, look for `Create from defintion` and click `OpenAPI`.
- In the OpenAPI specification box, paste this URI: `https://raw.githubusercontent.com/mikaelsand/reactor-sthlm-api-pub/main/Session2/Lab3/Oauth%20Demo.openapi%2Bjson.json`
- Leave the other as is and click Create. This will create a new API called `OAuth Demo` with one operation called `Read employee`.

## Configure the API

The new API is empty. It contans definitions but it does not do anything. You will now configure the API to protect itself using OAuth and return a standard message if the authentication is successful.

### Configure the operation

- Click the GET Read Employee Operation.
- In the middle under Inbound processing find and click the `</>` symbol. This will open up the policy editor.
- Replace everything in the editor with the content of [this policy file](read-employee-policy.xml).
- The end result looks like this:

```XML
<policies>
    <inbound>
        <base />
        <validate-jwt header-name="Authorization" failed-validation-httpcode="401" failed-validation-error-message="Token invalid or did not contain required role" require-expiration-time="true" require-scheme="Bearer" require-signed-tokens="true" clock-skew="60" output-token-variable-name="jwt">
            <openid-config url="https://login.microsoftonline.com/26ef3687-262a-44fa-95f4-208111b732e9/v2.0/.well-known/openid-configuration" />
            <audiences>
                <audience>cc565aa4-a1e5-4c05-b1cf-df19ca4c95be</audience>
            </audiences>
            <issuers>
                <issuer>https://login.microsoftonline.com/26ef3687-262a-44fa-95f4-208111b732e9/v2.0</issuer>
            </issuers>
            <required-claims>
                <claim name="roles" match="any">
                    <value>Employees.Read</value>
                </claim>
            </required-claims>
        </validate-jwt>
        <return-response>
            <set-status code="200" reason="OK" />
            <set-header name="content-type" exists-action="override">
                <value>application/json</value>
            </set-header>
            <set-body>{"Message":"If you can read this, you have successfully authorized."}</set-body>
        </return-response>
    </inbound>
    <backend>
        <base />
    </backend>
    <outbound>
        <base />
    </outbound>
    <on-error>
        <base />
    </on-error>
</policies>
```

The operation will now check the token for:

- Is it valid? The `open-id config` tag.
- Is it meant for me? The `audiences` tag
- Was it issued by the correct authority? The `issuer` tag.
- Is the caller allowed to read employee data? The `required-claims` tag.

If anything is wrong a standard message and code 401 will be returned. If everything is ok a standard message and 200 OK is returned.

> This API does not call any backend, it contructs the response itself.

Do not forget to save your policy.

## Call the API

If you imported the Postman collection there is a call created for you. It is called Get Employee and is located under the Get Token request.

> You need to update the API-key for the request before sending the call. You can your your admin key. If you need help finding your admin-key, you can find the instructions [here](howtofindtheadminkey.md).

- Update the request header `Ocp-Apim-Subscription-Key` to contain your admin key.
- Make sure you have a fresh token by using the `Get Token` request first. If this is successful, the token will be updated and can be used in the `Get Employee` request.
- Execute the `Get Employee` request. If everything is working, you should get back a 200 OK and this message.

```JSON
{
    "Message": "If you can read this, you have successfully authorized."
}
```

## Additional exercises

- Update the policy to use another claim and role. See what happens.
- Create a new POST operation and protect that using the claim `Employee.Write`. The token contains this claim so it will work.
- Create a new PUT operation and protect it using the claim `Employee.Update`. The token does not contain this claim.
- What could you do to improve the policy code? There is a lot of repeating code. Can some things be put into named values?
