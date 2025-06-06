# DGII-eCF

## Descripción del Proyecto
DGII-eCF ofrece utilidades para firmar, validar y enviar documentos fiscales electrónicos en la República Dominicana. La librería facilita a las aplicaciones Node.js manejar e-CF usando los servicios de la DGII.

## Funcionalidades
- Lector de certificados y firma digital (`P12Reader`, `Signature`).
- Cliente API para autenticación y envío de facturas (`ECF`).
- Ayudantes para flujos de autenticación personalizados (`CustomAuthentication`).
- Utilidades para códigos QR, transformación y validación de XML.

## Instalación
Instale el paquete usando npm:

```bash
npm install dgii-ecf
```

## Ejemplos de Uso

### Carga del Certificado
```ts
import path from 'path';
import { P12Reader } from 'dgii-ecf';

const reader = new P12Reader('your_password');
const certs = reader.getKeyFromFile(path.resolve(__dirname, 'certificate.p12'));
```

### Autenticación
```ts
import { ECF } from 'dgii-ecf';

const ecf = new ECF(certs);
const tokenData = await ecf.authenticate();
```

### Envío de Facturas
```ts
import fs from 'fs';
import { Signature, ECF } from 'dgii-ecf';

const xml = fs.readFileSync('invoice.xml', 'utf8');
const signature = new Signature(certs.key!, certs.cert!);
const signedXml = signature.signXml(xml, 'ECF');
const response = await ecf.sendElectronicDocument(signedXml, 'RNCeNCF.xml');
```

### Validar XML
```ts
import { validateXMLCertificate } from 'dgii-ecf';

const result = validateXMLCertificate(signedXml);
if (result.isValid) {
  console.log('Certificate Subject:', result.cert?.subject);
}
```

### Autenticación Personalizada
```ts
import { CustomAuthentication } from 'dgii-ecf';

const customAuth = new CustomAuthentication(certs);
const seed = customAuth.generateSeed();
const token = await customAuth.verifySignedSeed(seed);
```

## Configuración de Entorno
Utilice `ENVIRONMENT` para seleccionar el entorno DGII:
- `ENVIRONMENT.DEV` - Desarrollo (TesteCF)
- `ENVIRONMENT.CERT` - Certificación (CerteCF)
- `ENVIRONMENT.PROD` - Producción (eCF)

```ts
import { ECF, ENVIRONMENT } from 'dgii-ecf';
const ecf = new ECF(certs, ENVIRONMENT.DEV);
```

## Pruebas
Ejecute las pruebas unitarias e integradas:

```bash
npm run test
```

## Uso con Docker
Construir la imagen:
```bash
docker build -t dgii-ecf .
```
Ejecutar el contenedor:
```bash
docker run --rm dgii-ecf
```


## Enlaces y Documentación
- [Documentación Oficial DGII](https://dgii.gov.do/cicloContribuyente/facturacion/comprobantesFiscalesElectronicosE-CF/Paginas/default.aspx)
- [Documentación Técnica DGII](https://dgii.gov.do/cicloContribuyente/facturacion/comprobantesFiscalesElectronicosE-CF/Paginas/documentacionSobreE-CF.aspx)
