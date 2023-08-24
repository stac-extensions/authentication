# Secure Assets Extension Specification

- **Title:** Secure Assets
- **Identifier:** <https://stac-extensions.github.io/secure-assets/v1.0.0/schema.json>
- **Field Name Prefix:** -
- **Scope:** Asset
- **Extension [Maturity Classification](https://github.com/radiantearth/stac-spec/tree/master/extensions/README.md#extension-maturity):** Proposal
- **Owner**: @jamesfisher-gis

The Secure Assets extension to the [STAC](https://github.com/radiantearth/stac-spec) specification provides a way to specify what authentication
scheme is needed to access an asset in secured storage. The aim of the Secure Assets extension is to provide a standard field for clients to
recognise secure assets and access them via the specified authentication scheme.

The Secure Assets extension aligns with the [stac-asset](https://github.com/stac-utils/stac-asset) library, which has support for several
[authentication clients](https://github.com/stac-utils/stac-asset#clients). A `signed_url` scheme can be specified that would a user-defined
API to generate signed URLs. See the [Signed URL](#url-signing) section for an example Lambda function.

- Examples:
  - [Item example](examples/item.json): Shows the basic usage of the extension in a STAC Item
  - [Collection example](examples/collection.json): Shows the basic usage of the extension in a STAC Collection
- [JSON Schema](json-schema/schema.json)
- [Changelog](./CHANGELOG.md)

## Fields

The fields in the table below can be used in these parts of STAC documents:

- [ ] Catalogs
- [ ] Collections
- [ ] Item Properties (incl. Summaries in Collections)
- [x] Assets (for both Collections and Items, incl. Item Asset Definitions in Collections)
- [ ] Links

| Field Name | Type                                                    | Description                                            |
| ---------- | ------------------------------------------------------- | ------------------------------------------------------ |
| security   | Map<string, [SecureAsset Object](#secure-asset-object)> | Object that desribes the authenticated scheme and href |

### Additional Field Information

#### security

An Asset property used to specify a keyword that defines the authentication scheme of the Asset and provide a description of how to authenticate.

### Secure Asset Object

An Asset with the Secure Assets extension will have a "security" object with the following fields

| Field Name  | Type   | Description                                                                                                                                                            |
| ----------- | ------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| scheme      | string | **REQUIRED**. The authentication scheme used to access the data (`http` \| `s3` \| `planetary_computer` \| `earthdata` \| `signed_url`). |
| description | string | Additional instructions for authentication                                                                                                                           |

### Schemes

The available authentication schemes align with relevant clients included in the [stac-asset](https://github.com/stac-utils/stac-asset) library.

| Name                      | Description                                                                                                              |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| `http`              | Simple HTTP client without any authentication                                                                            |
| `s3`                | Simple S3 client                                                                                                         |
| `planetary_computer` | Signs URLs with the [Planetary Computer Authentication API](https://planetarycomputer.microsoft.com/docs/reference/sas/) |
| `earthdata`         | Uses a token-based authentication to download data, from *some* Earthdata providers, e.g. DAACs                          |
| `signed_url`         | Signs URLs with a user-defined authentication API                                                                      |

### URL Signing

The `signed_url` scheme indicates that authentication will be handled by an API, which generates and returns a signed URL. For example,
a signed URL for an asset in AWS S3 can be generated with the following Lambda function code.

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

The authentication API can be called clientside base on an AWS S3 href (`https://<bucket>.s3.<region>.amazonaws.com/<key>`) with the following code
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
