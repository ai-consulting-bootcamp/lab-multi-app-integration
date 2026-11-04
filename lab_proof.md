## Escenario
El equipo comercial de una consultoría de IA recibe por Telegram solicitudes de cotización de leads o clientes interesados en sus servicios. El workflow registra automáticamente en Airtable las solicitudes que contienen cinco campos con las etiquetas en el orden establecido y valores no vacíos. Esto evita copiar los datos manualmente y reduce el riesgo de dejar solicitudes sin registrar.

## Comportamiento técnico
Cuando llega un mensaje por Telegram, el workflow comprueba el número de líneas, el orden de las etiquetas y que sus valores no estén vacíos; si cumple el formato, transforma los datos y crea un registro en Airtable; de lo contrario, envía un aviso por Telegram y no registra la solicitud.

## Workflow export
[Ver workflow](workflow/telegram_airtable_solicitudes.json)

## Payload de origen

Payload de una ejecución real. Los datos personales se han anonimizado.

```json
1 item
  
  
"message": 
{ 
"message_id": 
9,
"from": 
{
"id": 
nros,
"is_bot": 
false,
"first_name": 
"San",
"last_name": 
"Regon",
"language_code": 
"es"
},
"chat": 
{
"id": 
nro,
"first_name": 
"San",
 "last_name": 
"Regon",  
"type": 
"private"
},
"date": 
1793798389,
"text": 
"Empresa:CCC\nContacto:Alberto Salazar\nEmail:asalazar@example.com\nCelular:+42565434555\nSolicitud: implementacion",
"entities": 
[ 
{  
"offset": 
37,  
"length": 
26, 
"type": 
"url" 
} 
] 
} 
}
]
```
## Registro de destino

Registro creado en Airtable en la misma ejecución del payload de origen. Chat ID anonimizado.

```json
[
{ 
"id": 
"rec6QEeQ6rHcaKk1k", 
"createdTime": 
"2026-11-04T13:19:51.000Z", 
"fields": 
{ 
"Contacto": 
"Alberto Salazar",
"Solicitud": 
"implementacion", 
"Chat ID": 
"nro",
"Empresa": 
"CCC",
"Celular": 
"+42565434555", 
"Timestamp": 
"2026-11-04T13:19:49.000Z", 
"Email": 
"asalazar@example.com" 
} 
}
]
```

## Tabla de mapeo

| Origen en Telegram | Destino en Airtable | Transformación | Motivo |
|---|---|---|---|
| `message.text`, línea 1 (`[0]`) | Empresa | Separar por `\n`, seleccionar la primera línea, quitar `Empresa:` y aplicar `trim()`. | Identificar la empresa que solicita la cotización. |
| `message.text`, línea 2 (`[1]`) | Contacto | Seleccionar la segunda línea, quitar `Contacto:` y aplicar `trim()`. | Identificar a la persona de contacto. |
| `message.text`, línea 3 (`[2]`) | Email | Seleccionar la tercera línea, quitar `Email:` y aplicar `trim()`. | Conservar el correo proporcionado para contactar. |
| `message.text`, línea 4 (`[3]`) | Celular | Seleccionar la cuarta línea, quitar `Celular:` y aplicar `trim()`. Mantener como texto para conservar el `+`. | Conservar el teléfono proporcionado para contactar. |
| `message.text`, línea 5 (`[4]`) | Solicitud | Seleccionar la quinta línea, quitar `Solicitud:` y aplicar `trim()`. | Conocer la necesidad indicada por el cliente. |
| `message.chat.id` | Chat ID | Convertir el identificador a texto mediante `String()`. | Relacionar el registro con el chat de origen. |
| `message.date` | Timestamp | Convertir segundos Unix a fecha ISO mediante `DateTime.fromSeconds(...).toISO()`. | Conservar la fecha y hora de envío del mensaje. |

Limitación: la extracción depende del orden de las cinco líneas. El If comprueba las etiquetas, su orden y los valores no vacíos antes de permitir el registro; no verifica que el email o el celular sean válidos.

## Riesgo del mapeo y detección
Si el mensaje incumple las condiciones del If, se rechaza. Y ese cambio se detecta porque algún campo está vacío o en otro orden. Ello se evidencia porque no se registra nada en Airtable y Telegram devuelve mensaje indicando que ingrese la información en forma correcta. 
El campo más frágil es message.text, porque el wf extrae los datos según la posición de sus líneas
Lo que no estamos validando, es que la información ingresada cumpla con formatos reales y se detectan soluciones duplicadas. Hemos probado el rechazo por email vacío; todavía no hemos demostrado con una prueba el rechazo por etiquetas desordenadas.

## Verificación
| Prueba | Resultado observado | 
|---|---|
|Solicitud válida:[evidence/solicitud completa.png](evidence/solicitud%20completa.png)) |Pasa por True y crea un registro con los datos esperados en Airtable.|
|Solicitud con email vacío:[evidence/email vacio.png](evidence/email%20vacio.png)) |Pasa por False, envía un aviso por Telegram y no crea un registro en Airtable.|
