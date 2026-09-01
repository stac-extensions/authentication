# Authentication Extension Specification

- **Title:** Authentication
- **Identifier:** <https://stac-extensions.github.io/authentication/v1.1.0/schema.json>
- **Field Name Prefix:** auth
- **Scope:** Catalog, Collection, Item, Asset, Links
- **Extension [Maturity Classification](https://github.com/radiantearth/stac-spec/tree/master/extensions/README.md#extension-maturity):** Proposal
- **Owner**: @jamesfisher-gis

The Authentication extension to the [STAC](https://github.com/radiantearth/stac-spec) specification provides  a standard set of fields to 
describe authentication and authorization schemes, flows, and scopes required to access Assets and Links that align with the 
[OpenAPI security spec](https://swagger.io/docs/specification/authentication/)

The Authentication extension also includes support for other [authentication schemes](https://github.com/stac-utils/stac-asset#clients) specified in 
[stac-asset](https://github.com/stac-utils/stac-asset) library. A `signedUrl` scheme type can be specified that describes authentication via signed 
URLs returned from a user-defined API. See the [Signed URL](#url-signing) section for a Lambda function example.

- Examples:
  - [Item example](examples/item.json): Shows the basic usage of the extension in a STAC Item
  - [Collection example](examples/collection.json): Shows the basic usage of the extension in a STAC Collection
- [JSON Schema](json-schema/schema.json)
- [Changelog](./CHANGELOG.md)

## Fields

The fields in the table below can be used in these parts of STAC documents:

- [x] Catalogs
- [x] Collections
- [x] Item Properties (incl. Summaries in Collections)
- [ ] Assets (for both Collections and Items, incl. Item Asset Definitions in Collections)
- [ ] Links

| Field Name     | Type                                                         | Description |
| -------------- | ------------------------------------------------------------ | ----------- |
| `auth:schemes` | Map<string, [Authentication Scheme Object](#authentication-scheme-object)> | A property that contains all of the [scheme definitions](#authentication-scheme-object) used by Assets and Links in the STAC Item or Collection. |

---

The fields in the table below can be used in these parts of STAC documents:

- [ ] Catalogs
- [ ] Collections
- [ ] Item Properties (incl. Summaries in Collections)
- [x] Assets (for both Collections and Items, incl. Item Asset Definitions in Collections)
- [x] Links

| Field Name  | Type       | Description |
| ----------- | ---------- | ----------- |
| `auth:refs` | \[string\] | A property that specifies which schemes in `auth:schemes` may be used to access an Asset or Link. `auth:refs` MAY also appear on an [Authentication Scheme Object](#authentication-scheme-object) itself — nested inside `auth:schemes`, so within Catalogs, Collections and Item Properties — where it declares which schemes may supply this scheme's input — see below. |

### Scheme Types

The `type` value is not restricted to the following values, so a practitioner may define a custom authentication or authorization scheme not 
included in the scheme type standards below.

| Name                | Description |
| ------------------- | ----------- |
| `http`              | Simple HTTP authentication mechanisms (Basic, Bearer, Digest, etc.). |
| `s3`                | Simple S3 authentication.                                    |
| `signedUrl`         | Signs URLs with a user-defined authentication API.           |
| `oauth2`            | [Open Authentication (OAuth) 2.0](https://datatracker.ietf.org/doc/html/rfc6749) configuration |
| `apiKey`            | Description of [API key](https://swagger.io/docs/specification/authentication/api-keys/) authentication included in request headers, query parameters, or cookies. |
| `openIdConnect`     | Description of [OpenID Connect](https://openid.net/specs/openid-connect-core-1_0.html) authentication |

### Authentication Scheme Object

The Authentication Scheme extends the 
[OpenAPI security spec](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.0.3.md#security-scheme-object)
for support of OAuth2.0, API Key, and OpenID Connect authentication.
All the [authentication clients](https://github.com/stac-utils/stac-asset#clients) included in the 
[stac-asset](https://github.com/stac-utils/stac-asset)
library can be described, as well as a custom signed URL authentication scheme.

| Field Name         | Type                                                         | Applies to            | Description                                                  |
| ------------------ | ------------------------------------------------------------ | --------------------- | ------------------------------------------------------------ |
| `type`             | string                                                       | *All*                 | **REQUIRED**. The authentication scheme type used to access the data (`http` \| `s3` \| `signedUrl` \| `oauth2` \| `apiKey` \| `openIdConnect` \| a custom scheme type ). |
| `description`      | string                                                       | *All*                 | Additional instructions for authentication. [CommonMark 0.29](https://commonmark.org/) syntax MAY be used for rich text representation. |
| `auth:refs`        | \[string\]                                                  | *All*                 | Keys of other entries in `auth:schemes` that may supply this scheme's input: any *one* of the referenced schemes is completed first, and the token it yields is the input to this one. **REQUIRED with at least one entry** for an `oauth2` scheme declaring a `tokenExchange` flow (the token obtained through whichever referenced scheme was used is the RFC 8693 `subject_token`); MAY be used by other types, e.g. a `signedUrl` scheme naming the scheme(s) whose token authenticates requests to its `authorizationApi`. References MUST NOT form cycles. |
| `name`             | string                                                       | `apiKey`              | **REQUIRED.** The name of the header, query, or cookie parameter to be used. |
| `in`               | string                                                       | `apiKey`              | **REQUIRED.** The location of the API key (`query` \| `header` \| `cookie`). |
| `scheme`           | string                                                       | `http`                | **REQUIRED.** The name of the HTTP Authorization scheme to be used in the [Authorization header as defined in RFC7235](https://tools.ietf.org/html/rfc7235#section-5.1).  The values used SHOULD be registered in the [IANA Authentication Scheme registry](https://www.iana.org/assignments/http-authschemes/http-authschemes.xhtml). (`basic` \| `bearer` \| `digest` \| `dpop` \| `hoba` \| `mutual` \| `negotiate` \| `oauth` (1.0) \| `privatetoken` \| `scram-sha-1` \| `scram-sha-256` \| `vapid`) |
| `flows`            | Map<string, ([OAuth2 Flow Object](#oauth2-flow-object)\|[Signed URL Object](#signed-url-object))> | `oauth2`, `signedUrl` | **REQUIRED.** Scenarios an API client performs to get an access token from the authorization server. For `oauth2` the following keys are pre-defined for the corresponding OAuth flows: `authorizationCode` \| `implicit` \| `password` \| `clientCredentials` \| `tokenExchange`. The OAuth2 Flow Object applies for `oauth2`, the Signed URL Object applies to `signedUrl`. |
| `openIdConnectUrl` | string                                                       | `openIdConnect`       | **REQUIRED.** OpenID Connect URL to discover OpenID configuration values. This MUST be in the form of a URL. |

The column "Applies to" specifies for which values of `type` the fields only apply.
They are also only required in this context.

### OAuth2 Flow Object

Based on the [OpenAPI OAuth Flow Object](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.0.3.md#oauth-flows-object).
Allows configuration of the supported OAuth Flows.
The `tokenExchange` flow corresponds to OAuth 2.0 Token Exchange as defined in
[RFC 8693](https://datatracker.ietf.org/doc/html/rfc8693).
The client presents a token obtained through another scheme (for example an OpenID
Connect identity token) at the `tokenUrl` as the `subject_token`, with
`grant_type=urn:ietf:params:oauth:grant-type:token-exchange`, and receives a different
security token back, such as short-lived, scoped credentials for direct data access.
The response is the one defined in
[RFC 8693, Section 2.2.1](https://datatracker.ietf.org/doc/html/rfc8693#section-2.2.1),
which reports the kind of token issued in `issued_token_type`.

Which scheme supplies the input token is declared on the *scheme*, with `auth:refs`
(see the [Authentication Scheme Object](#authentication-scheme-object)): a scheme
declaring a `tokenExchange` flow MUST reference at least one other scheme in
`auth:refs`; the client completes any *one* of the referenced schemes, and the token
obtained through it is the `subject_token`. This is what makes the multi-step flow
machine-discoverable — a client resolves the chain from the document (asset → guarding
scheme → input scheme) instead of hardcoding the order — and it reuses the reference
mechanism clients already implement for Assets and Links, with the same one-of
semantics in both positions: on an Asset or Link the referenced schemes are alternative
ways to access the resource, on a scheme they are alternative ways to obtain its input.
A sequence of steps is expressed as a chain — each scheme referencing the one before
it — not as multiple entries in one list. References MUST NOT form cycles — clients
cannot be expected to resolve one, and the JSON Schema cannot detect it.

The RFC 8693 `subject_token_type` request parameter MUST be taken from the flow's
`subjectTokenType` field when present. When the field is absent it follows from the
`type` of the referenced scheme the client used:
`urn:ietf:params:oauth:token-type:id_token` for `openIdConnect` and
`urn:ietf:params:oauth:token-type:access_token` for `oauth2`. Publishers SHOULD set
`subjectTokenType` explicitly whenever a referenced scheme yields more than one kind
of token (an OpenID Connect provider issues both an ID token and an access token).
Since the field applies to the flow as a whole, referenced alternatives requiring
*different* `subject_token_type` values SHOULD be split into separate exchange
schemes.

To accept identities from more than one provider, list the identity schemes as
alternatives in the exchange scheme's own `auth:refs` — a token obtained through any
one of them is exchanged at the same `tokenUrl`.

| Field Name         | Type                    | Description                                                  |
| ------------------ | ----------------------- | ------------------------------------------------------------ |
| `authorizationUrl` | `string`                | **REQUIRED** for parent keys: `"implicit"`, `"authorizationCode"`. The authorization URL to be used for this flow. This MUST be in the form of a URL. |
| `tokenUrl`         | `string`                | **REQUIRED** for parent keys: `"password"`, `"clientCredentials"`, `"authorizationCode"`, `"tokenExchange"`. The token URL to be used for this flow. This MUST be in the form of a URL. |
| `subjectTokenType` | `string`                | For parent key `"tokenExchange"`: the RFC 8693 `subject_token_type` URN of the token presented at the `tokenUrl` (e.g. `urn:ietf:params:oauth:token-type:id_token`). When absent, it follows from the `type` of the scheme in the scheme's `auth:refs` through which the token was obtained. |
| `scopes`           | Map<`string`, `string`> | **REQUIRED.** The available scopes for the authentication scheme. A map between the scope name and a short description for it. The map MAY be empty. |
| `refreshUrl`       | `string`                | The URL to be used for obtaining refresh tokens. This MUST be in the form of a URL. |

### Signed URL Object

A `signedUrl` scheme MAY declare the scheme(s) through which its `authorizationApi`
requests can be authenticated via `auth:refs` on the scheme (see the
[Authentication Scheme Object](#authentication-scheme-object)).

| Field Name         | Type                                               | Description                                                  |
| ------------------ | -------------------------------------------------- | ------------------------------------------------------------ |
| `method`           | `string`                                           | **REQUIRED.** The method to be used for requests             |
| `authorizationApi` | `string`                                           | **REQUIRED.** The signed URL API endpoint to be used for this flow. If not inferred from the client environment, this must be defined in the authentication flow. |
| `parameters`       | Map<string, [Parameter Object](#parameter-object)> | Parameter definition for requests to the `authorizationApi`  |
| `responseField`    | string                                             | Key name for the signed URL field in an `authorizationApi` response |

### Parameter Object

Definition for a request parameter.

| Field Name    | Type      | Description                                                  |
| ------------- | --------- | ------------------------------------------------------------ |
| `in`          | `string`  | **REQUIRED.** The location of the parameter (`query` \| `header` \| `body`). |
| `required`    | `boolean` | **REQUIRED.** Setting for optional or required parameter.    |
| `description` | `string`  | Plain language description of the parameter                  |
| `schema`      | `object`  | Schema object following the [JSON Schema draft-07](https://json-schema.org/) |

## Examples

`auth:schemes` may be referenced identically in a STAC Asset or Link objects. Examples of these two use-cases are provided below.
A complete, focused example of the two-step token-exchange pattern (identity scheme + exchange scheme linked via scheme-level `auth:refs`)
is provided in [examples/collection-token-exchange.json](examples/collection-token-exchange.json).

### Schema definitions

```json
"auth:schemes": {
  "oauth": {
    "type": "oauth2",
    "description": "requires a login and user token",
    "flows": {
      "authorizationCode": {
        "authorizationUrl": "https://example.com/oauth/authorize",
        "tokenUrl": "https://example.com/oauth/token",
        "scopes": {}
      }
    }
  }
}
```

### Links reference

```json
"links": [
  {
    "href": "https://example.com/examples/collection.json",
    "rel": "self"
  },
  {
    "href": "https://example.com/examples/item.json",
    "rel": "item",
    "auth:refs": [
      "oauth"
    ]
  }
]
```

### Asset reference

```json
"assets": {
  "data": {
    "href": "https://example.com/examples/file.xyz",
    "title": "Secure Asset Example",
    "type": "application/vnd.example",
    "roles": [
      "data"
    ],
    "auth:refs": [
      "oauth"
    ]
  }
}
```

### URL Signing

The `signedUrl` scheme type indicates that authentication will be handled by an API which generates and returns a signed URL. A signed URL 
authentication scheme can be defined with 
```json
"auth:schemes": {
  "signed_url_auth": {
    "type": "signedUrl",
    "description": "Requires an authentication API",
    "flows": {
      "authorizationCode": {
        "authorizationApi": "https://example.com/signed_url/authorize",
        "method": "POST",
        "parameters": {
          "bucket": {
            "in": "body",
            "required": true,
            "description": "asset bucket",
            "schema": {
              "type": "string",
              "examples": "example-bucket"
            }
          },
          "key": {
            "in": "body",
            "required": true,
            "description": "asset key",
            "schema": {
              "type": "string",
              "examples": "path/to/example/asset.xyz"
            }
          }
        },
        "responseField": "signed_url"
      }
    }
  }
}
```

and generated via a Gateway API and the following Lambda function.

```python
import boto3
from botocore.client import Config
import os
import json

def lambda_handler(event, context):
    try:
        s3Client = boto3.client("s3")
    except Exception as e:
        return {
            "statusCode": 400,
            "body": json.dumps({
                "error": (e)
                })
        }

    body = json.loads(event["body"])
    key = body["key"]
    bucketName = body["bucket"]

    try:
        URL = s3Client.generate_presigned_url(
            "get_object",
            Params = {"Bucket": bucketName, "Key":key},
            ExpiresIn = 360
            )

        return ({
            "statusCode": 200,
            "body": json.dumps({
                "signed_url": URL
            }),
            "headers":{
                "Access-Control-Allow-Origin": "*",
                "Access-Control-Allow-Headers": "*"
            }

        })
    except Exception as e:
        return {
            "statusCode": 400,
            "body": json.dumps({
                "error": (e)
                })
        }
```

Where the response looks like

```json
{
  "signed_url": "https://<bucket>.s3.<region>.amazonaws.com/<key>?AWSAccessKeyId=<aws access key>&Signature=<signature>&x-amz-security-token=<auth token>&Expires=<epoch expiration time>"
}
```

The authentication API can be called on the client side based on an AWS S3 href (`https://<bucket>.s3.<region>.amazonaws.com/<key>`) with the 
following code snippet.

```javascript

let signed_url;
const auth_api = "";

function createSignedRequestBody(href) {
  const bucket = href.split(".")[0].split("//")[1];
  const key = href.split("/").slice(3).join("/").replace(/\+/g, " ");
  return {
    method: "POST",
    headers: {
      Accept: "application/json",
      "Content-Type": "application/json",
    },
    body: JSON.stringify({ bucket: bucket, key: key }),
    redirect: "follow",
  };
}

Promise(
  fetch(auth_api, createSignedRequestBody(href))
    .then((resp) => resp.json())
    .then((respJson) => {
      signed_url = respJson.signed_url;
    })
);

```

### Planetary Computer URL Signing

Planetary Computer uses the same signed URL pattern described above. Here is an example of how to configure a `signedUrl` `auth:scheme` for the  [Planetary Computer Data Authentication API](https://planetarycomputer.microsoft.com/docs/reference/sas/)

```json
"auth:schemes": {
  "plantetary_computer_auth": {
    "type": "signedUrl",
    "description": "Requires authorization from Planetary Computer",
    "flows": {
      "authorizationCode": {
        "authorizationApi": "https://planetarycomputer.microsoft.com/api/sas/v1/sign",
        "method": "GET",
        "parameters": {
          "href": {
            "in": "query",
            "required": true,
            "description": "HREF (URL) to sign",
            "schema": {
              "type": "string",
            }
          },
          "duration": {
            "in": "query",
            "required": false,
            "description": "The duration, in minutes, that the SAS token will be valid. Only valid for approved users.",
            "schema": {
              "type": "integer",
            }
          },
          "_id": {
            "in": "query",
            "required": false,
            "description": "Third party user identifier for metrics tracking.",
            "schema": {
              "type": "string"
            }
          }
        },
        "responseField": "href"
      }
    }
  }
}
```

### Simple S3 authentication

To use simple S3 authentication one has to set some environmental variables with S3 credentials:

- `AWS_SECRET_ACCESS_KEY`
- `AWS_ACCESS_KEY_ID`

**or** specify a [user profile](https://docs.aws.amazon.com/cli/v1/userguide/cli-configure-files.html#cli-configure-files-format)
with a proper reference to `AWS_PROFILE` in the file `AWS_CONFIG_FILE`.

For more information please see either
[GDAL vsis3](https://gdal.org/en/latest/user/virtual_file_systems.html#vsis3-aws-s3-files) or
[AWS CLI](https://docs.aws.amazon.com/cli/v1/userguide/cli-configure-files.html) documentation. 

Additionally, if the `s3` authentication method is referred to through `auth:refs`, you should disable signing requests,
e.g. through setting `AWS_NO_SIGN_REQUEST` to `NO`. Otherwise it should be `YES`.

## Contributing

All contributions are subject to the
[STAC Specification Code of Conduct](https://github.com/radiantearth/stac-spec/blob/master/CODE_OF_CONDUCT.md).
For contributions, please follow the
[STAC specification contributing guide](https://github.com/radiantearth/stac-spec/blob/master/CONTRIBUTING.md) Instructions
for running tests are copied here for convenience.

### Running tests

The same checks that run as checks on PR's are part of the repository and can be run locally to verify that changes are valid.
To run tests locally, you'll need `npm`, which is a standard part of any [node.js installation](https://nodejs.org/en/download/).

First you'll need to install everything with npm once. Just navigate to the root of this repository and on
your command line run:

```bash
npm install
```

Then to check markdown formatting and test the examples against the JSON schema, you can run:

```bash
npm test
```

This will spit out the same texts that you see online, and you can then go and fix your markdown or examples.

If the tests reveal formatting problems with the examples, you can fix them with:

```bash
npm run format-examples
```
