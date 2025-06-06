# ERP Integration Guide

This document explains how to integrate **dgii-ecf** with any existing ERP system. It assumes your ERP already handles the sales workflow and you want to automate sending electronic fiscal documents (e-CF) to the DGII.

## 1. Environment Preparation

1. Install the library in the Node.js project used by your ERP:

```bash
npm install dgii-ecf
```

2. Make sure you have your digital certificate (.p12) and its password.

## 2. Loading the Certificate

Use `P12Reader` to load the private key and certificate used to sign XML.

```ts
import path from 'path';
import { P12Reader } from 'dgii-ecf';

const reader = new P12Reader('certificate_password');
const certs = reader.getKeyFromFile(path.resolve(__dirname, 'certificate.p12'));
```

## 3. Authentication

Before sending documents, obtain a valid token from DGII.

```ts
import { ECF } from 'dgii-ecf';

const ecf = new ECF(certs);
const tokenData = await ecf.authenticate();
```

Store `tokenData.token` according to your ERP's authentication logic and renew it when needed.

## 4. Signing and Sending Documents

Your ERP should generate the invoice XML according to DGII's standard. Once generated, use `Signature` to sign it and `ECF` to send it.

```ts
import fs from 'fs';
import { Signature } from 'dgii-ecf';

const xml = fs.readFileSync('invoice.xml', 'utf8');
const signature = new Signature(certs.key!, certs.cert!);
const signedXml = signature.signXml(xml, 'ECF');

const response = await ecf.sendElectronicDocument(signedXml, 'RNCeNCF.xml');
```

Record the `trackId` and response status in your ERP for future reference.

## 5. Status Inquiry

To know the status of a previously sent document, use `statusTrackId`.

```ts
const status = await ecf.statusTrackId(trackId);
```

Integrate this information with your ERP's tracking or reporting modules.

## 6. Handling Responses and Errors

- Validate each operation's response and log any error codes in your ERP.
- Implement retries or user notifications on network failures or invalid responses.

## 7. DGII Environments

Configure the environment according to the context (Development, Certification or Production).

```ts
import { ENVIRONMENT } from 'dgii-ecf';
const ecf = new ECF(certs, ENVIRONMENT.PROD);
```

## 8. Best Practices

- Keep integration logic in a separate module within the ERP.
- Protect the private key and limit access to it.
- Automate token renewal and certificate expiration control.

## 9. Basic Flow Example

1. The user registers the invoice in the ERP.
2. The ERP generates the e-CF XML.
3. The XML is signed using `Signature`.
4. The document is sent with `ECF.sendElectronicDocument`.
5. The response and `trackId` are stored.
6. Periodically query the status with `statusTrackId` until confirmation is received.

With this integration, your ERP can manage electronic invoicing automatically.
