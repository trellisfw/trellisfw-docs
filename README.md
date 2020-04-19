# trellis-docs
Documentation for the Trellis framework for data.

The Trellis Framework builds on the [Open Ag Data Alliance API standards](https://github.com/oada/oada-docs).
The core of this framework is an API spec for creating data **connections** between
platforms. Any platform or app can be made "Trellis conformant" or "OADA conformant" by supporting
that API.  Trellis adds a [document signature process](signatures.md) for document integrity to the OADA 
standard, as well as a method auditably obscuring shared data to preserve privacy known as [Mask & Link](https://github.com/trellisfw/trellisfw-masklink).

## Useful links for Trellis:
| Item | Link |
| --- | --- |
|  Data Models   |  https://github.com/OADA/oada-formats/tree/master/formats/application/vnd/trellis |
|  OADA API Core |  https://github.com/OADA/oada-docs |
|  Signatures    |  https://github.com/trellisfw/trellisfw-signatures |
|  Signatures Demo | https://trellisfw.github.io/trellisfw-audit-sign-verify |
| Mask&Link | https://github.com/trellisfw/trellisfw-masklink |

## Getting Started

To start using Trellis, download and install the [OADA Reference Implementation](https://github.com/oada/oada-srvc-docker):
```bash
git clone git@github.com:OADA/oada-srvc-docker.git
cd oada-srvc-docker
./oada --install-self
oada up -d
```
Please refer to the [OADA Documentation](https//github.com/oada/oada-docs).

## Apps

### Conductor 
Conductor lets you monitor and drop in PDF documents for scraping, analysis, and sharing via automatic smart rules.  

* Code: https://github.com/trellisfw/conductor
* Live: https://trellisfw.github.com/conductor

In order to run Conductor yourself, you should have the following service modules running against your Trellis installation:
* [ainz](https://github.com/trellisfw/ainz): main "smart rules" orchestration service
* [abalonemail](https://github.com/trellisfw/abalonemail): service for sending emails in response to rules via sendgrid
* [sendgrid-ingest](https://github.com/trellisfw/sendgrid-ingest): receives emails from sendgrid and places them in the document queue.
* [oada-backups](https://github.com/oada/oada-backups): Maintains rolling nightly backups of internal database
* [oada-ensure](https://github.com/oada/oada-ensure): Ensures doubly-linked virtual documents
* [trellis-masker](https://github.com/trellisfw/trellis-masker): Service to automatically mask particular key names in any virtual document's JSON
* [trellis-signer](https://github.com/trellisfw/trellis-signer): Service to sign incoming JSON as transcriptions

For your deployment, you should have your own domains and tokens.  You can easily supply your production domains and tokens to these services using the `z_tokens` method: create and enable `z_tokens/docker-compose.yml` which looks like this:
```docker-compose
version: '3'

services:
  ##########################################
  # Overrides for oada-core services:
  ##########################################
  admin:
    volumes:
      - ./services-available/z_tokens:/code/z_tokens

  ###############################################
  # Add tokens for all services that need them:
  ###############################################
  abalonemail:
    environment:
      - token=yourtoken
      - domain=https://yourdomain
      - emailKey=youremailkeyforsendgrid
      - from=your@from.address

  ainz:
    environment:
      - token=yourtoken
      - domain=https://yourdomain

  sendgrid-ingest:
    environment:
      - token=yourtoken
      - domain=https://yourdomain
      - blacklist=abalmos@gmail.com,abalmos@purdue.edu

  trellis-signer:
    volumes:
      - ./services-available/z_tokens/private_key.jwk:/private_key.jwk
    environment:
      - token=yourtoken
      - domain=https://yourdomain
      - privateJWK=/private_key.jwk

  trellis-masker:
    volumes:
      - ./services-available/z_tokens/private_key.jwk:/private_key.jwk
    environment:
      - token=yourtoken
      - domain=https://yourdomain
      - privateJWK=/private_key.jwk

  oada-ensure:
    environment:
      - token=yourtoken
      - domain=https://yourdomain
```

### Reagan
If you have a "masked" piece of data and have permission from the data owner to verify it, you can pass that
masked object (or a URL to something that is masked), and Reagan will `Trust, but Verify` for you.  Just add either
a `trellis-mask` or `masked-resource-url` query parameter and watch it work it's magic!

* Code: https://github.com/trellisfw/reagan
* Live: https://trellisfw.github.com/reagan

## Libraries
Useful javascript libraries for Trellis:
* [Trellis Mask&Link](https://github.com/trellisfw-masklink): library for creating and validating masks.
* [Trellis Signatures](https://github.com/trellisfw-signatures): library for creating and validating Trellis document signatures.
* [Trellis Formats](https://github.com/oada/oada-formats): Schemas for various Trellis document types.

## Services
* [ainz](https://github.com/trellisfw/ainz): main "smart rules" orchestration service
* [abalonemail](https://github.com/trellisfw/abalonemail): service for sending emails in response to rules via sendgrid
* [sendgrid-ingest](https://github.com/trellisfw/sendgrid-ingest): receives emails from sendgrid and places them in the document queue.
* [oada-backups](https://github.com/oada/oada-backups): Maintains rolling nightly backups of internal database
* [oada-ensure](https://github.com/oada/oada-ensure): Ensures doubly-linked virtual documents
* [trellis-masker](https://github.com/trellisfw/trellis-masker): Service to automatically mask particular key names in any virtual document's JSON
* [trellis-signer](https://github.com/trellisfw/trellis-signer): Service to sign incoming JSON as transcriptions
* [unfisk](https://github.com/trellisfw/unfisk): Service to simplify PUT's to OADA: you can PUT an object to a parent resource, and this service will automatically make it into a resource and link appropriately.
