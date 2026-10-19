# bank-postman — Colección de pruebas del Sistema Bancario

Repositorio **dedicado exclusivamente a la colección de Postman** del proyecto, tal como lo exige
la Parte III del enunciado:

> *"...mantener un repositorio de Postman para las pruebas manuales"*

Es un repositorio **independiente** de los microservicios: no contiene código Java, solo los
escenarios de prueba que se usan para verificar funcionalmente la solución.

---

## Contenido

| Archivo | Descripción |
|---|---|
| `Bank-System-Microservices.postman_collection_v2.json` | **Colección principal** (schema v2.1). Es la que se importa y ejecuta. |
| `Bank-System-Microservices.postman_collection.json` | Versión resumida de la colección (mismo contenido, formato ligero). |

**Colección:** `Bank System - Complete Test Scenarios (Partes I, II y III)`
**Total:** 84 peticiones en 9 carpetas.

---

## Estructura de la colección

| Carpeta | Peticiones | Cubre |
|---|---:|---|
| `0. Auth Service` | 2 | Login y registro JWT (Parte III) |
| `1. Customer Service` | 6 | CRUD de clientes, tipos PERSONAL/BUSINESS, perfiles VIP y PYME |
| `2. Credit Service` | 11 | CRUD de créditos personales y empresariales |
| `3. Bank Account Service` | 14 | Cuentas de ahorro, corriente y plazo fijo + reglas de producto |
| `4. Transactions & Transfers` | 9 | Depósitos, retiros y transferencias |
| `5. Credit Payments, Card Consumption & Balances` | 14 | Pagos de crédito, consumos con tarjeta y saldos |
| `6. Debit Cards & Reports` | 11 | Tarjetas de débito y **reportes** |
| `7. Yanki Wallet (Kafka)` | 9 | Monedero Yanki: registro, pagos entre monederos, carga/retiro |
| `8. Update & Delete (CRUD restante)` | 8 | Endpoints `PUT` y `DELETE` de todas las entidades |
| **Total** | **84** | |

Cada carpeta incluye tanto **escenarios de éxito** como **escenarios de error**, para validar las
reglas de negocio (cuentas de ahorro con límite de movimientos, cuenta corriente con comisión,
plazo fijo con un solo movimiento al mes, límites de crédito, etc.).

---

## Cómo importarla

1. Abrir **Postman**.
2. `Import` → `File` → seleccionar
   `Bank-System-Microservices.postman_collection_v2.json`.
3. Aparecerá la colección **“Bank System - Complete Test Scenarios (Partes I, II y III)”**.

También puede importarse desde la línea de comandos:

```bash
newman run Bank-System-Microservices.postman_collection_v2.json
```

---

## Requisitos previos

La colección llama a los microservicios **locales**, por lo que hay que tenerlos corriendo:

| Servicio | URL (variable) | Puerto |
|---|---|---|
| auth-service | `{{auth_url}}` | 8084 |
| customer-service | `{{customer_url}}` | 8081 |
| bank-account-service | `{{account_url}}` | 8082 |
| credit-service | `{{credit_url}}` | 8083 |
| yanki-service | `{{wallet_url}}` | 8085 |

Y la infraestructura: **MongoDB** (27017), **Kafka** (9092), **Redis** (6379) y **Eureka** (8761).

Orden recomendado de arranque (ver `README.md` del proyecto):

```text
MongoDB → Redis → Kafka → config-server → discovery-server →
auth-service → customer-service → bank-account-service → credit-service →
yanki-service → api-gateway
```

---

## Autenticación (JWT)

Todas las peticiones protegidas usan `Authorization: Bearer {{token}}`.

**Ejecutar primero** la carpeta `0. Auth Service` → `POST {{auth_url}}/auth/login`:

```json
{ "username": "admin", "password": "admin123" }
```

El script de pruebas de esa petición guarda el token devuelto en la **variable de colección**
`{{token}}`; el resto de peticiones lo reutilizan automáticamente.

> Los usuarios por defecto los crea el propio `auth-service` al arrancar (`DataInitializer`):
> `admin` / `admin123` con rol `ADMIN`, y `cliente1` / `cliente123` con rol `USER`.

---

## Variables de colección

| Variable | Valor por defecto | Descripción |
|---|---|---|
| `auth_url` | `http://localhost:8084` | Base URL de autenticación |
| `customer_url` | `http://localhost:8081` | Base URL de clientes |
| `account_url` | `http://localhost:8082` | Base URL de cuentas bancarias |
| `credit_url` | `http://localhost:8083` | Base URL de créditos |
| `wallet_url` | `http://localhost:8085` | Base URL del monedero Yanki |
| `token` | *(vacío)* | JWT devuelto por `POST /auth/login` |
| `personalCustomerId` | `cust-pers-01` | Cliente personal (estándar) |
| `businessCustomerId` | `cust-biz-01` | Cliente empresarial |
| `personalVipCustomerId` | `cust-vip-01` | Cliente personal con perfil VIP |
| `savingsAccountId` | `acc-savings-01` | Cuenta de ahorro |
| `targetAccountId` | `acc-savings-02` | Cuenta destino de transferencias |
| `personalCreditId` | `cred-pers-01` | Crédito personal |
| `creditCardId` | `cred-card-01` | Tarjeta de crédito |
| `transactionId` | `tx-001` | Identificador de movimiento |
| `debitCardId` | `card-001` | Identificador de tarjeta de débito |
| `walletId` | `wallet-001` | Identificador de monedero Yanki |
| `walletCellphone` | `999888777` | Celular del monedero origen |
| `targetWalletCellphone` | `999111222` | Celular del monedero destino |
| `creditMovementId` | *(vacío)* | Identificador de movimiento de crédito |

> Los identificadores (`customer_*`, `*AccountId`, `credit*Id`, etc.) se **rellenan
> automáticamente** por los scripts de pruebas de cada petición al crear la entidad, así que
> conviene ejecutar las carpetas **en orden**: primero crear, después operar.

---

## Notas

- La verificación de la colección incluye **aserciones de estado HTTP** en cada petición, de modo
  que un fallo de negocio se reporta como error y no como una llamada correcta.
- La carpeta `7. Yanki Wallet (Kafka)` valida el flujo **orientado a eventos**: `yanki-service`
  no se comunica por REST con el resto de servicios, solo a través de los topics de Kafka.
- La base de datos es **por servicio** (`database per service`): cada microservicio accede
  únicamente a su propia base en MongoDB.
