# AWS Serverless URL Shortener

Independent AWS learning project using Lambda, API Gateway and DynamoDB, with a static frontend historically hosted through S3 and CloudFront with TLS.

**Status:** no longer deployed. This repository contains the application files, not a reproducible infrastructure stack. The frontend was AI-generated; my focus was AWS integration, hosting and troubleshooting.

## Repository contents

| File | Purpose |
| --- | --- |
| [ShortenerFunction.py](ShortenerFunction.py) | Python Lambda handler for creating mappings and redirects |
| [index.html](index.html) | HTML/CSS/JavaScript frontend |
| [README.md](README.md) | Architecture, contract and deployment caveats |

## Request flow

The frontend submits JSON containing `long_url` to API Gateway. Lambda creates a six-character code and writes a `short_code` / `long_url` mapping to DynamoDB. A GET request looks up the code and returns an HTTP 302 redirect, or 404 when the code is unknown.

Historical hosting also used S3, CloudFront, ACM and a custom API domain. Those cloud settings, IAM policies, CORS configuration and monitoring settings are not captured as infrastructure code in this repository.

## Current handler contract

- Expects an API Gateway event with `requestContext.http.method` (HTTP API-style payload).
- Uses a DynamoDB table named `URLShortner`, keyed by the string attribute `short_code`.
- POST expects a JSON body containing `long_url`.
- GET expects `pathParameters.short_code`.
- The current POST response key is `Your shortened URL short_code`.

## Known integration gaps

**The checked-in frontend and handler do not currently agree on the response key:** the frontend reads `data[""] || data.short_url`, but the handler returns the key above. Align that contract and test it before redeployment. An API mapping might change responses externally, but no such configuration is committed here.

The frontend endpoint and returned API URL are hard-coded historical values. Replace them with your own configuration; they are not live demo links.

Other limitations:

- No conditional write or retry to prevent a short-code collision overwriting an existing mapping.
- No server-side URL validation, abuse controls or comprehensive malformed-input handling.
- No committed automated tests or infrastructure-as-code configuration.
- CORS must be configured consistently for the frontend origin and API responses.
- No measured latency, availability or user-volume claim is made.

## Reproduction checklist

1. Use an isolated AWS environment and review the Python handler before deployment.
2. Create the DynamoDB table with the key described above. Configure a Lambda role scoped to the table's required read/write operations.
3. Package the handler, configure compatible API Gateway routes/event format, and align the frontend response contract.
4. Replace historical endpoints, configure CORS and TLS, and host the frontend using your own S3/CloudFront setup.
5. Test POST, redirect, missing code, invalid JSON and invalid URL cases. Add collision tests before broader use.
6. Capture the infrastructure in Terraform and automate these tests as future improvements.

## Cost and cleanup

AWS charges depend on requests, storage and distribution usage. Set a budget before deploying. Remove only the dedicated lab resources after reviewing dependencies; export any mappings you want to retain before deleting the table. There is no automated teardown in this repository.
