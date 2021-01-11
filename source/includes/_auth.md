# Authorization

We use the OAuth 2.0 protocol, an industry standard for authorization management.

Make sure you have the _CLIENT ID_ an a _SECRET KEY_ provided by our support team. If you don't have them contact us at <a class="link-primary" href="mailto:info@stratifi.com">support@stratifi.com</a>.

## Authorization Code Flow

Following the steps below you will get an access token that can be used to access StratiFi on behalf of the advisor.

1.  Redirect the advisor to the StratiFi authorization page:

    `https://{{ domain }}/o/authorize/?response_type=code&state={{ state }}&client_id={{ client_id }}`

    | Parameter | Description                                 |
    | --------- | ------------------------------------------- |
    | domain    | StratiFi sandbox or production domain       |
    | state     | Random string generated by your application |
    | client_id | Provided by StratiFi                        |

1.  The advisor submits the StratiFi credentials.

    ![login](https://s3.amazonaws.com/api.stratifi.com/login.2.png "Login")

1.  The advisor allow your application access the resources in StratiFi.

    ![grant](https://s3.amazonaws.com/api.stratifi.com/grant.2.png "Grant")

1.  We redirect the browser to your return url with a single-usage access code.

    `https://{{ callback-url }}code={{ code }}&state={{ state }}`

    | Parameter    | Description                                                |
    | ------------ | ---------------------------------------------------------- |
    | callback-url | URL provided by you when during the application setup      |
    | state        | Random string generated by your application                |
    | code         | Single-usage code that can be exchanged by an access token |

1.  Your application backend exchange the access code by an access token.

    ```shell
    curl -X POST "https://backend.stratifi.com/o/token/" \
    -d '{
    "grant_type": "authorization_code",
    "code": "{{ code }}",
    "redirect_uri": "{{ redirect_url }}",
    "client_id": "{{ client_id }}",
    "client_secret": "{{ secret_key }}"
    }'

    {
    "access_token": "{{ access_token }}",
    "token_type": "Bearer",
    "expires_in": 3600,
    "refresh_token": "{{ refresh_token }}",
    "scope": "read write"
    }
    ```

    **Request Parameters**

    | Parameter     | Type   |                                                |
    | ------------- | ------ | ---------------------------------------------- |
    | grant_type    | string | Always "authorization_code"                    |
    | code          | string | The single-usage code received in the callback |
    | redirect_uri  | string | The callback URL                               |
    | client_id     | string | Your application client id                     |
    | client_secret | string | Your application secret key                    |

    **Response**

    | Parameter     | Type   |                                                                        |
    | ------------- | ------ | ---------------------------------------------------------------------- |
    | token_type    | string | Always "Bearer"                                                        |
    | access_token  | string | An access token valid to consult other endpoints on behalf of the user |
    | expires_in    | string | Expiration time of the access token in seconds                         |
    | refresh_token | string | An refresh token valid to renew the access token                       |
    | scope         | string | The scopes with granted access for this token                          |

1.  Include the _Authorization_ header in your requests as follows:

    `Authorization: Bearer {{ access-token }}`

## Refresh Token Flow

If your access token expired you can renew it using a valid refresh token.

```shell
curl -X POST "https://backend.stratifi.com/o/token/" \
-d '{
    "grant_type": "refresh_token",
    "refresh_token": "{{ refresh_token }}",
    "client_id": "{{ client_id }}",
    "client_secret": "{{ secret_key }}"
}'

{
    "access_token": "{{ access_token }}",
    "token_type": "Bearer",
    "expires_in": 3600,
    "refresh_token": "{{ refresh_token }}",
    "scope": "read write"
}
```

**Request Parameters**

| Parameter     | Type   |                             |
| ------------- | ------ | --------------------------- |
| grant_type    | string | Always "refresh_token"      |
| refresh_token | string | Your Refresh Token          |
| client_id     | string | Your application client id  |
| client_secret | string | Your application secret key |

**Response**

| Parameter     | Type   |                                                                        |
| ------------- | ------ | ---------------------------------------------------------------------- |
| token_type    | string | Always "Bearer"                                                        |
| access_token  | string | An access token valid to consult other endpoints on behalf of the user |
| expires_in    | string | Expiration time of the access token in seconds                         |
| refresh_token | string | An refresh token valid to renew the access token                       |
| scope         | string | The scopes with granted access for this token                          |