# DGII-eCF

## Project Overview
DGII-eCF provides utilities to sign, validate and send electronic fiscal documents in the Dominican Republic. The library helps Node.js applications manage electronic invoices using DGII services.

## Features
- Certificate reader and signer (`P12Reader`, `Signature`).
- API client to authenticate and submit invoices (`ECF`).
- Helpers for custom authentication flows (`CustomAuthentication`).
- Utilities for QR codes, XML transformation and validation.

## Installation
Install the package using npm:

```bash
npm install dgii-ecf
```

## Usage Examples

### Certificate Loading
```ts
import path from 'path';
import { P12Reader } from 'dgii-ecf';

const reader = new P12Reader('your_password');
const certs = reader.getKeyFromFile(path.resolve(__dirname, 'certificate.p12'));
```

### Authentication
```ts
import { ECF } from 'dgii-ecf';

const ecf = new ECF(certs);
const tokenData = await ecf.authenticate();
```

### Sending Invoices
```ts
import fs from 'fs';
import { Signature, ECF } from 'dgii-ecf';

const xml = fs.readFileSync('invoice.xml', 'utf8');
const signature = new Signature(certs.key!, certs.cert!);
const signedXml = signature.signXml(xml, 'ECF');
const response = await ecf.sendElectronicDocument(signedXml, 'RNCeNCF.xml');
```

### Validate XML
```ts
import { validateXMLCertificate } from 'dgii-ecf';

const result = validateXMLCertificate(signedXml);
if (result.isValid) {
  console.log('Certificate Subject:', result.cert?.subject);
}
```

### Custom Authentication
```ts
import { CustomAuthentication } from 'dgii-ecf';

const customAuth = new CustomAuthentication(certs);
const seed = customAuth.generateSeed();
const token = await customAuth.verifySignedSeed(seed);
```

## Environment Configuration
Use `ENVIRONMENT` to select the target DGII environment:
- `ENVIRONMENT.DEV` - Development (TesteCF)
- `ENVIRONMENT.CERT` - Certification (CerteCF)
- `ENVIRONMENT.PROD` - Production (eCF)

```ts
import { ECF, ENVIRONMENT } from 'dgii-ecf';
const ecf = new ECF(certs, ENVIRONMENT.PROD);
```

## Testing
Run the unit and integration tests:

```bash
npm run test
```

## Docker Usage
Build the image:
```bash
docker build -t dgii-ecf .
```
Run the container:
```bash
docker run --rm dgii-ecf
```

## ERP Integration
For a detailed guide on how to embed **dgii-ecf** into your ERP system, see [docs/ERP_Integration_Guide.en.md](docs/ERP_Integration_Guide.en.md).


## Links & Documentation
- [DGII Official Documentation](https://dgii.gov.do/cicloContribuyente/facturacion/comprobantesFiscalesElectronicosE-CF/Paginas/default.aspx)
- [DGII Technical Docs](https://dgii.gov.do/cicloContribuyente/facturacion/comprobantesFiscalesElectronicosE-CF/Paginas/documentacionSobreE-CF.aspx)
