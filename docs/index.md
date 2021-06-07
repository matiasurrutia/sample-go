# ms-p2p-transfers

## API: Transferencia

/ms/p2p



### Crear intención P2P

![picture](images/p2p-generar-intencion-flow-v2.png)


### Transferencia Diferida - Crear Intenci&oacute;n

El envío de dinero diferido se da cuando el emisor es un usuario de TodoPago y el destinatario no tiene cuenta en TodoPago, se puede enviar el dinero a través de un número de celular y/o un mail.

**1) POST /accounts/:id/intentions**

Al ser un envío de dinero diferido se validan solo los datos de la cuenta emisora.

|Validaciones |                                                                           |
| ------------|---------------------------------------------------------------------------|
|DENOMINATION1|No debe ser nulo                                                           |
|STATUS       |CTA_HABILITADA                                                             |
|TYPE         |PROFESIONAL / PARTICULAR                                                   |
|USER_TYPE    |ADMIN                                                                      |
|AMOUNT       |>=$(monto mínimo configurable) y >= al saldo disponible. Admite 2 decimales|
|DESCRIPTION  |Texto 25 caracteres                                                        |
|Límites      |Control de límites configurados por equipo de Riesgos a través de panel    |

También, existe un Feature Flag para todas las integraciones con COELSA, en el caso que se encuentre en ON se agrega una validación:

- El emisor debe tener CVU (Clave Virtual Uniforme), caso contrario no puede enviar dinero hasta que tenga su CVU dada de alta.

##### `request`

```json
{
    "concept": "remera xl",
    "amount": 200.00,
    "recipient_validator": {
        "strategy": "PHONE",
        "value": "1130202192"
    },
    "link": {
        "title": "¡Juan te envió $200!",
        "description": "Seguí este link para recibir el dinero",
        "image": "https://img.todopago.com.ar/logo",
        "analytics_info": {
          	"google_play_analytics": {
            	"utm_source": "string",
                "utm_medium": "string",
                "utm_campaign": "string",
                "utm_term": "string",
                "utm_content": "string",
                "gclid": "string"
          	},
            "itunes_connect_analytics": {
                "at": "string",
                "ct": "string",
                "mt": "string",
                "pt": "string"
            }
        }
    }
}
```

##### `response - 200 (OK)`

```json
{
    "status": "PENDING",
    "valid_until": "2021-01-11T14:33:00",
    "link": "https://todopagowallet.page.link/LS8yyBSYgc4w3Ze29",
    "hash": "MPsIeXf3ND3UgnxyNS1Mcg"
}
```
#### Tabla de Errores:
|Status|Code	                                           |Tittle                                                                                      |
|------|---------------------------------------------------|--------------------------------------------------------------------------------------------|
|422   |EXISTING_ACCOUNT_EXCEPTION                         |There is already an account created for EMAIL with value $$email                            |
|422   |EXISTING_ACCOUNT_EXCEPTION                         |There is already an account created for PHONE with value $$phone                            |
|404   |P2P_ACCOUNT_NOT_EXIST                              |No se encontró la cuenta.                                                                   |
|409   |P2P_ACCOUNT_WITHOUT_NAME                           |Configure su nombre y apellido para enviar dinero.                                          |
|409   |"P2P_INVALID_ACCOUNT_STATE                         |El estado de la cuenta no permite enviar dinero.                                            |
|409   |P2P_INVALID_ACCOUNT_TYPE                           |El tipo de cuenta no permite enviar dinero.                                                 |
|409   |P2P_OTP_SMS_REQUIRED                               |Se requiere validación de línea telefónica por SMS antes de enviar dinero.                  |
|409   |P2P_BIOMETRICS_REQUIRED                            |Se requiere validación de reconocimiento facial antes de enviar dinero.                     |
|409   |P2P_INVALID_SOURCE_CVU                             |Tenés que tener tu Clave Virtual Uniforme (CVU) para completar la operación.                |
|409   |AMOUNT_LESS_THAN_MINIMUN                           |El monto de dinero a enviar debe ser mayor o igual a $$amountMinimun                        |
|409   |INVALID_CONCEPT                                    |La cantidad de caracteres debe ser menor o igual a 25 y no pueden ser caracteres especiales.|
|409   |LIMITS_ERROR                                       |¡Te notificamos que llegaste al límite de envío de dinero permitido!                        |
|409   |INSUFFICIENT_AVAILABLE_BALANCE                     |El saldo disponible de la cuenta virtual $$cuentaVirtual es insuficiente.                   |


**2) PUT /public/v1/accounts/:accountId/intentions/approve**

##### `url params`

|name       | description      |
| ----------| -----------------|
|accountId  | id de la cuenta  |


### Transferencia Directa - Crear Intenci&oacute;n


##### `Headers`

|Key            |Value                |
|---------------|---------------------|
|Context-Type:  |application/json     |
|Authorization: |bearer {access_token}|

#### POST /accounts/:id/movements

##### `url params`

|name    | description     |
| -------| --------------- |
| id     | id de la cuenta |

##### `request`

```json
{
    "concept": "remera xl",
    "amount": 200.00,
    "target_account": "10814"
}
```
|Name	       |Description                            |Type   |Required|
| -------------| --------------------------------------|-------|--------|
|concept       | Concepto de la transacci&oacute;n     |String |NO      |
|amount        | Monto que se va a enviar              |Number |SI      |
|target_account|ID de la cuenta destino                |String |SI      |


##### `response - 200 (OK)`

```json
{
    "status": "APPROVED",
    "valid_until": "2021-01-11T14:53:40"
}
```
#### Tabla de Errores:
|Status|Code	                                           |Tittle                                                                                      |
|------|---------------------------------------------------|--------------------------------------------------------------------------------------------|
|404   |P2P_ACCOUNT_NOT_EXIST                              |La cuenta no existe.                                                                        |
|409   |P2P_ACCOUNT_WITHOUT_NAME                           |Configure su nombre y apellido para enviar dinero.                                          |
|409   |"P2P_INVALID_ACCOUNT_STATE                         |El estado de la cuenta no permite enviar dinero.                                            |
|409   |P2P_INVALID_ACCOUNT_TYPE                           |El tipo de cuenta no permite enviar dinero.                                                 |
|409   |P2P_OTP_SMS_REQUIRED                               |Se requiere validación de línea telefónica por SMS antes de enviar dinero.                  |
|409   |P2P_BIOMETRICS_REQUIRED                            |Se requiere validación de reconocimiento facial antes de enviar dinero.                     |
|409   |P2P_INVALID_SOURCE_ACCOUNT_STATE_PROCESS_INTENTION |La cuenta que envió el dinero no existe o se encuentra en un estado inválido.               |
|409   |P2P_INVALID_TARGET_ACCOUNT_STATE_PROCESS_INTENTION |El estado de la cuenta no permite recibir dinero.                                           |
|409   |P2P_INVALID_TARGET_ACCOUNT_TYPE_PROCESS_INTENTION  |El tipo de cuenta no permite recibir dinero.                                                |
|409   |P2P_INVALID_TARGET_CVU                             |El destinatario tiene que tener su Clave Virtual Uniforme (CVU) para completar la operación.|
|409   |P2P_INVALID_SOURCE_CVU                             |Tenés que tener tu Clave Virtual Uniforme (CVU) para completar la operación.                |
|409   |MOVEMENT_EQUALS_ACCOUNTS                           |No podés enviarte dinero a tu propia cuenta.                                                |
|409   |AMOUNT_LESS_THAN_MINIMUN                           |El monto de dinero a enviar debe ser mayor o igual a $$amountMinimun                        |
|409   |INVALID_CONCEPT                                    |La cantidad de caracteres debe ser menor o igual a 25 y no pueden ser caracteres especiales.|
|409   |LIMITS_ERROR                                       |¡Te notificamos que llegaste al límite de envío de dinero permitido!                        |
|409   |INSUFFICIENT_AVAILABLE_BALANCE                     |El saldo disponible de la cuenta virtual 453514 es insuficiente.                            |

### Obtener datos de intención
Estados posibles que pueden tener las intenciones:

|id |Status   |
|---|---------|
|1  |PENDING  |
|2  |APPROVED |
|3  |CANCELLED|
|4  |EXPIRED  |


#### GET /intentions/:hash

##### `response - 200`

```json
{
    "hash": "MPsIeXf3ND3UgnxyNS1Mcg",
    "source_account": {
        "first_name": "Franco",
        "last_name": "Martinez",
        "email": "fscarpello_loadtest@prismamp.com"
    },
    "target_account": {
        "first_name": "Matias",
        "last_name": "Menendez",
        "email": "matias@prismamp.com"
    },
    "status": "PENDING",
    "concept": "remera xl",
    "amount": 5000.0,
    "expiration_date": "2020-07-31T17:55:19",
    "link": "http://portal.integration.todopago.com.ar/app/intentions/p2p?hash=6300",
    "short_link": "https://todopagowallet.page.link/5Rg6gzepD7smBu8y8",
    "payment_source": "Saldo Virtual"
}
```

##### `response - 4XX`

```json
{
    "errors": [
        {
            "status": "404",
            "code": "INTENTION_NOT_FOUND",
            "title": "No existe una intencion con hash {hash}"
        }
    ]
}
```

### Aceptar intención

#### PATCH /intentions/:hash

##### `request`

```json
{
    "target_account": "1231",
    "status": "APPROVED"
}
```

##### `response - 200 (ok)`

```json
{
    "hash": "MPsIeXf3ND3UgnxyNS1Mcg",
    "source_account": {
        "first_name": "Franco",
        "last_name": "Martinez",
        "email": "fscarpello_loadtest@prismamp.com"
    },
    "target_account": {
        "first_name": "Matias",
        "last_name": "Menendez",
        "email": "matias@prismamp.com"
    },
    "status": "APPROVED",
    "concept": "remera xl",
    "amount": 5000.0,
    "valid_until": "2020-07-31T17:55:19.5",
    "long_link": "http://portal.integration.todopago.com.ar/app/intentions/p2p?hash=6300",
    "short_link": "https://todopagowallet.page.link/5Rg6gzepD7smBu8y8"
}
```

##### `response - 4XX`

```json
{
    "errors": [
        {
            "status": "422",
            "code": "INVALID_RESERVE",
            "title": "La reserva está vencida"
        }
    ]
}
```
## transfer-intention-controller

##### `Headers`

|Key            |Value                |
|---------------|---------------------|
|Context-Type:  |application/json     |
|Authorization: |bearer {access_token}|

### Cancelar intención

Las intenciones de envío de dinero las puede cancelar el emisor, es decir quien realiza el envío de dinero, y solo en estado pendiente.

**DELETE /public/v1/accounts/:accountId/intentions/:intentionId**

##### `url params`

|name       | description      |
| ----------| -----------------|
|accountId  | id de la cuenta  |
|intentionId|id de la intención|

El id de la intención se puede consultar a través del hash, que se obtiene al crear la intención, en la tabla p2p_transfers..p2p_intention.

##### `response - 200 (OK)`

```json
{
   {
    "id": "24664",
    "source_account": {
        "id": 3265770,
        "type": "PARTICULAR",
        "first_name": "María",
        "last_name": "Merlo",
        "account_status": "ENABLED",
        "activity": {
            "working_area_id": 5999
        },
        "validations": [],
        "identification": "34929223",
        "identification_type": 21,
        "cuit": "27349292233",
        "cvu": {
            "code": "0001234200000032657708",
            "is_active": true
        },
        "email": "mv@yopmail.com"
    },
    "target_account": null,
    "payment_source": -1,
    "amount": 3.0,
    "concept": "Varios",
    "status": 3,
    "link": "https://portal-tp-qa.prismamp.com/app/intentions/p2p?hash=fNx9Vg4iCuQwt2WjS743Pw",
    "short_link": "https://todopagowallet.page.link/55yZEpmnRYNR4QQf9",
    "creation_date": "2021-02-10T17:46:55.137",
    "update_date": "2021-02-10T17:46:55.137",
    "reserve_id": 13776,
    "valid_until": null,
    "hash": "",
    "recipient": "prueba01@yopmail.com",
    "valid": false
}
}
```

#### Tabla de Errores:
|Status|Code	                                           |Tittle                                                                                      |
|------|---------------------------------------------------|--------------------------------------------------------------------------------------------|
|422   |INVALID_INTENTION_STATUS                           |Invalid intention status for intention $$id                                                 |


### Validar destinatario
Dentro del flujo de envío de dinero directo, al seleccionar un contacto de la agenda/ingresar (mail y/o teléfono), se validan ciertas condiciones para que el receptor pueda recibir el dinero de forma satisfactoria.

|Condiciones      |                                   |
| ----------------| ----------------------------------|
|Estados de Cuenta|CTA_HABILITADA / CTA_INHAB_CASH_OUT|
|Type             |PROFESIONAL / PARTICULAR           |
|USER_TYPE        |ADMIN                              |

También, existe un Feature Flag para todas las integraciones con COELSA, en el caso que se encuentre en ON entonces se agrega una condición:

- El destinatario debe tener CVU (Clave Virtual Uniforme), caso contrario no puede recibir dinero hasta que tenga su CVU.

**POST /public/v1/accounts/validate-target**


|Name	   |Description           |Type   |Required|
| ---------| ---------------------|-------|--------|
|account_id|ID de la cuenta       |Integer|SI      |
|user_id   |ID del tipo de usuario|Integer|SI      |

##### `request`

```json
{
  "account_id": 3265770,
  "user_id": 450964
}
```

##### `response - 200 (OK)`

```json
{
    "valid_account": true
}
```

### Tabla de Errores:

|Status	|Code	                                           |Tittle                                                                                   
| ------| -------------------------------------------------|--------------------------------------------------------------------------------------------|
|400    |VALIDATION_ERROR                                  |Field 'accountId' is invalid                                                                |
|400    |VALIDATION_ERROR                                  |Field 'userId' is invalid                                                                   |
|404    |ACCOUNT_NOT_FOUND                                 |No se encontró la cuenta                                                                    |
|404    |USER_ACCOUNT_NOT_FOUND                            |Usuario 1 no encontrado para la cuenta 3265770                                              |
|409    |P2P_INVALID_TARGET_ACCOUNT_STATE_PROCESS_INTENTION|El estado de la cuenta no permite recibir dinero.                                           |
|409    |P2P_INVALID_TARGET_ACCOUNT_TYPE_PROCESS_INTENTION |El tipo de cuenta no permite recibir dinero.                                                |
|409    |P2P_INVALID_TARGET_CVU                            |El destinatario tiene que tener su Clave Virtual Uniforme (CVU) para completar la operación.|
|409    |P2P_INVALID_ACCOUNT_USER_TYPE                     |El tipo de usuario no puede recibir dinero.                                                 |

