# Secure Assets Extension Specification

- **Title:** Secure Assets
- **Identifier:** <https://stac-extensions.github.io/secure-assets/v1.0.0/schema.json>
- **Field Name Prefix:** security
- **Scope:** Asset
- **Extension [Maturity Classification](https://github.com/radiantearth/stac-spec/tree/master/extensions/README.md#extension-maturity):** Proposal
- **Owner**: @jamesfisher-gis

The Secure Assets extension to the [STAC](https://github.com/radiantearth/stac-spec) specification provides a way to specify the authentication 
schemes, flows, and scopes required to access secured assets. The aim of the Secure Assets extension is to provide a standard set of fields to describe authentication that align with the [OpenAPI security spec](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.0.3.md#security-scheme-object)

The Secure Assets extension also includes support for other [authentication schemes](https://github.com/stac-utils/stac-asset#clients) specified in [stac-asset](https://github.com/stac-utils/stac-asset) library. A `signedUrl` scheme type can be specified that describes authentication via signed URLs returned from a user-defined 
API. See the [Signed URL](#url-signing) section for a Lambda function example.

- Examples:
  - [Item example](examples/item.json): Shows the basic usage of the extension in a STAC Item
  - [Collection example](examples/collection.json): Shows the basic usage of the extension in a STAC Collection
- [JSON Schema](json-schema/schema.json)
- [Changelog](./CHANGELOG.md)

## Fields

The fields in the table below can be used in these parts of STAC documents:

- [ ] Catalogs
- [x] Collections
- [x] Item Properties (incl. Summaries in Collections)
- [x] Assets (for both Collections and Items, incl. Item Asset Definitions in Collections)
- [ ] Links

| Field Name | Type                                                    | Description                                            |
| ---------- | ------------------------------------------------------- | ------------------------------------------------------ |
| security:schemes   | Map<string, [SecureAssetSchemeObject](#secure-asset-scheme-object)> | A property that contains all of the [scheme definitions](#secure-asset-scheme-object) used by Assets in the STAC Item or Collection. |
| security:refs   |   [string]    |   An Asset property that specifies which schemes in `security:schemes` may be used to access an Asset.   |

### Scheme Types

The available authentication schemes align with relevant clients included in the [stac-asset](https://github.com/stac-utils/stac-asset) library and in the [OpenAPI spec](https://swagger.io/docs/specification/authentication/).

| Name                      | Description                                                                                                              |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| `http`                    | Simple HTTP without any authentication                                                                                   |
| `s3`                      | Simple S3 authentication                                                                                                 |
| `planetaryComputer`       | Signs URLs with the [Planetary Computer Authentication API](https://planetarycomputer.microsoft.com/docs/reference/sas/) |
| `earthdata`               | Uses a token-based authentication to download data, from *some* Earthdata providers, e.g. DAACs                          |
| `signedUrl`               | Signs URLs with a user-defined authentication API                                                                        |
| `oauth2`                  | [Open Authentication 2.0](https://swagger.io/docs/specification/authentication/oauth2/) configuration                                                                                                                                          |
| `apiKey`                  | Description of [API key](https://swagger.io/docs/specification/authentication/api-keys/) authentication included in request headers, query parameters, or cookies.                                                                                                                                               |
| `openIdConnect`           | Description of [OpenID Connect Discovery](https://swagger.io/docs/specification/authentication/openid-connect-discovery/) authentication     

### Secure Asset Scheme Object

The Secure Asset Scheme aligns with the [OpenAPI security spec](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.0.3.md#security-scheme-object) for support of OAuth2.0, API Key, and OpenID authentication. All the [authentication clients](https://github.com/stac-utils/stac-asset#clients) included in the [stac-asset](https://github.com/stac-utils/stac-asset) library can be described, as well as a custom signed URL authentication scheme.

| Field Name  | Type   | Description                                                                                                                                         |
| ----------- | ------ | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| type      | string | **REQUIRED**. The authentication scheme type used to access the data (`http` \| `s3` \| `planetaryComputer` \| `earthdata` \| `signedUrl` \| `oauth2` \| `apiKey` \| `openIdConnect`).              |
| description | string | Additional instructions for authentication                                                                                                          |
| name | string | Required for `type: apiKey`. The name of the header, query, or cookie parameter to be used.                                                                 |
| in | string | Required for `type: apiKey`. The location of the API key (`query` \| `header` \| `cookie`).                                                                  |
| scheme | string | Required for `type: http`. The name of the HTTP Authorization scheme to be used in the [Authorization header as defined in RFC7235](https://tools.ietf.org/html/rfc7235#section-5.1).  The values used SHOULD be registered in the [IANA Authentication Scheme registry](https://www.iana.org/assignments/http-authschemes/http-authschemes.xhtml). (`basic` \| `bearer`)                                                                   |
| flows | Map<string, [SecureAssetFlowsObject](#secure-asset-flows-object)> | Required for `type: oauth2` and `type: signedUrl`. Scenarios an API client performs to get an access token from the authorization server (`authorizationCode` \| `implicit` \| `password ` \| `clientCredentials` \| `signedUrl`)  |
| openIdConnectUrl | string | Required for `type: openIdConnectUrl`. OpenId Connect URL to discover OAuth2 configuration values. This MUST be in the form of a URL.          |


### Secure Asset Flow Object

[OpenAPI OAuth Flow Object](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.0.3.md#oauth-flows-object). Allows configuration of the supported OAuth Flows.

Configuration details for a supported OAuth Flow

##### Fixed Fields
Field Name | Type | Description
---|:---:|---
authorizationUrl | `string` | Required for `oauth2` (`"implicit"`, `"authorizationCode"`). The authorization URL to be used for this flow. This MUST be in the form of a URL.  |
tokenUrl | `string` | Required for `oauth2` (`"password"`, `"clientCredentials"`, `"authorizationCode"`). The token URL to be used for this flow. This MUST be in the form of a URL.  |
authorizationApi | `string` | Optional for `signedUrl`. The signed URL API endpoint to be used for this flow. If not enferred from the client environment, this must be defined in the authentication flow.  |
refreshUrl | `string` | Optional for `oauth2`. The URL to be used for obtaining refresh tokens. This MUST be in the form of a URL.  |
scopes | Map[`string`, `string`] | Required for `oauth2`. The available scopes for the authentication scheme. A map between the scope name and a short description for it. The map MAY be empty. |


### URL Signing

The `signedUrl` scheme indicates that authentication will be handled by an API which generates and returns a signed URL. A signed URL authentication scheme can be defined with 
```json
{
  "security:schemes": {
    "signedUrl": {
      "type": "signedUrl",
      "description": "Requires an authentication API",
      "flows": {
        "signedUrl": {
            "authorizationApi": "https://example.com/signedUrl/authorize"
        }
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

The authentication API can be called on the client side based on an AWS S3 href (`https://<bucket>.s3.<region>.amazonaws.com/<key>`) with the following code 
snippet.

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
