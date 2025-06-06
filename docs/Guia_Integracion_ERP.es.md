# Guía de Integración con Sistemas ERP

Esta guía describe los pasos generales para integrar **dgii-ecf** dentro de cualquier sistema ERP existente. Se asume que el ERP ya gestiona el ciclo de ventas y se desea automatizar el envío de comprobantes fiscales electrónicos (e-CF) a la DGII.

## 1. Preparación del Entorno

1. Instale la librería en el proyecto Node.js utilizado por su ERP:

```bash
npm install dgii-ecf
```

2. Asegúrese de contar con su certificado digital (.p12) y su contraseña.

## 2. Carga del Certificado

Utilice `P12Reader` para cargar la clave privada y el certificado que se empleará para firmar los XML.

```ts
import path from 'path';
import { P12Reader } from 'dgii-ecf';

const reader = new P12Reader('contraseña_certificado');
const certs = reader.getKeyFromFile(path.resolve(__dirname, 'certificado.p12'));
```

## 3. Autenticación

Antes de enviar documentos, obtenga un token válido frente a la DGII.

```ts
import { ECF } from 'dgii-ecf';

const ecf = new ECF(certs);
const tokenData = await ecf.authenticate();
```

Conserve el `tokenData.token` según la lógica de autenticación de su ERP y renuévelo cuando sea necesario.

## 4. Firma y Envío de Documentos

El ERP debe generar el XML de la factura o documento electrónico siguiendo el estándar de la DGII. Una vez generado, utilice `Signature` para firmarlo y `ECF` para enviarlo.

```ts
import fs from 'fs';
import { Signature } from 'dgii-ecf';

const xml = fs.readFileSync('factura.xml', 'utf8');
const signature = new Signature(certs.key!, certs.cert!);
const signedXml = signature.signXml(xml, 'ECF');

const respuesta = await ecf.sendElectronicDocument(signedXml, 'RNCeNCF.xml');
```

Registre en su ERP el `trackId` y el estado de la respuesta para su posterior consulta.

## 5. Consulta de Estado

Para conocer el estado de un documento previamente enviado, utilice `statusTrackId`.

```ts
const estado = await ecf.statusTrackId(trackId);
```

Integre esta información en los módulos de seguimiento o reportes del ERP.

## 6. Manejo de Respuestas y Errores

- Valide las respuestas de cada operación y registre los códigos de error en el ERP.
- Implemente reintentos o notificación al usuario en caso de fallos de red o respuestas inválidas.

## 7. Entornos de DGII

Configure el entorno adecuadamente según el ambiente (Desarrollo, Certificación o Producción).

```ts
import { ENVIRONMENT } from 'dgii-ecf';
const ecf = new ECF(certs, ENVIRONMENT.PROD);
```

## 8. Buenas Prácticas

- Mantenga la lógica de integración en un módulo independiente dentro del ERP.
- Proteja la clave privada del certificado y limíte su acceso.
- Automatice la renovación del token y el control de vencimiento de certificados.

## 9. Ejemplo de Flujo Básico

1. El usuario registra la factura en el ERP.
2. El ERP genera el XML del e-CF.
3. Se firma el XML con `Signature`.
4. Se envía el documento con `ECF.sendElectronicDocument`.
5. Se almacena la respuesta y el `trackId`.
6. Se consulta periódicamente el estado con `statusTrackId` hasta recibir confirmación.

Con esta integración, su ERP podrá gestionar la emisión electrónica de comprobantes de forma automática.

