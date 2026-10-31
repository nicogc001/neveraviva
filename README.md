# NeveraViva

**Que no se te ponga nada malo en la nevera.** Haces una foto al ticket de la compra, la IA lee los productos y estima cuándo caduca cada uno, y la app te propone tres platos con lo que caduca antes, ajusta las raciones y te prepara la lista de la compra.

## Qué hace la app

1. **Leer el ticket.** Foto del ticket de supermercado (Mercadona, Carrefour, Lidl, Dia...). La IA convierte cada línea en un producto normalizado con su categoría.
2. **Estimar la caducidad.** A cada producto se le asigna una fecha estimada según su categoría y conservación (nevera, congelador, despensa). El usuario puede corregirla. Siempre se avisa de que es una estimación de IA.
3. **Proponer platos.** Tres tarjetas de plato que aprovechan primero lo que caduca antes, con raciones ajustables y respetando las alergias del usuario.
4. **Lista de la compra.** Lo que falta para los platos elegidos, con enlace para comprarlo online (afiliación con Carrefour vía Awin).

## Modelo de negocio

- **Premium:** 29,99 €/año (unos 21 € netos por suscriptor tras IVA y el 15 % de Apple con el Small Business Program), con muro de pago.
- **Afiliación** con supermercados.
- Crecimiento orgánico, sin publicidad pagada.

## Tecnología

| Pieza | Elección |
|---|---|
| App | Expo + React Native + TypeScript, con Expo Router. iOS primero; Android con el mismo código más adelante |
| Compilación | EAS Build en la nube (no hace falta Mac) y TestFlight para la beta |
| Backend | Supabase en Frankfurt (UE): Postgres, Auth, Storage y Edge Functions |
| IA | Modelo multimodal llamado desde el servidor; la clave nunca va en la app |
| Suscripciones | RevenueCat sobre StoreKit 2 |
| Errores y analítica | Sentry y PostHog (UE), nunca con datos de salud |

Más detalle en [docs/arquitectura.md](docs/arquitectura.md).

## Estructura del repositorio

```
.
├── app/                 App móvil (Expo)
│   ├── src/app/         Pantallas (Expo Router: cada archivo es una ruta)
│   ├── src/features/    Lógica por funcionalidad (tickets, nevera, platos, compra, cuenta, suscripción)
│   ├── src/lib/         Clientes compartidos (Supabase, RevenueCat, analítica)
│   ├── src/components/  Componentes de interfaz reutilizables
│   └── assets/          Iconos, splash e imágenes
├── supabase/            Backend: migraciones y funciones del servidor
├── docs/                Arquitectura y cumplimiento legal
├── .vscode/             Ajustes y extensiones recomendadas para VS Code
└── .github/workflows/   Integración continua (lint y tipos)
```

## Desarrollar en Windows con VS Code

No hace falta Mac. El flujo es:

1. Instalar [Node.js 22 LTS](https://nodejs.org) y [Git](https://git-scm.com).
2. Abrir la carpeta del repo en VS Code y aceptar las extensiones recomendadas.
3. Instalar la app **Expo Go** en el iPhone (App Store).
4. En la terminal de VS Code:

   ```bash
   cd app
   npm install
   copy .env.example .env.local    # en PowerShell o CMD; rellenar con las claves públicas
   npx expo start
   ```

5. Escanear el código QR con la cámara del iPhone (móvil y PC en la misma wifi). La app se recarga sola al guardar.

Cuando necesitemos módulos nativos (compras, cámara avanzada) usaremos una *development build* compilada con EAS en la nube:

```bash
npm install -g eas-cli
eas login
eas build --profile development --platform ios
```

Para eso hace falta la cuenta de Apple Developer (99 €/año).

Comandos útiles (desde `app/`):

```bash
npx expo install <paquete>   # instalar dependencias compatibles con el SDK
npm run lint
npm run typecheck
```

## Reglas del proyecto

- Las claves privadas (IA, *service role* de Supabase) **solo** viven en el servidor, nunca en `app/`.
- Datos y servidores en la UE.
- Las alergias son datos de salud (art. 9 RGPD): consentimiento aparte, tabla separada y cifrada.
- Lo que genera la IA se identifica como tal (art. 50 del Reglamento de IA), y toda fecha de caducidad se muestra como estimación.
- Ramas `feature/<nombre-corto>` y cambios a `main` mediante pull request.

Lista completa en [docs/legal.md](docs/legal.md).
