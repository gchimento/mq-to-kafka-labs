# MQ to Kafka labs: configurando Streaming Queues y Kafka Connect
Las colas y tópicos de IBM MQ son una fuente valiosa de datos y eventos para procesar. En este ejercicio, vamos a ver cómo los mensajes de MQ pueden publicarse en tópicos de Kafka, desde donde se pueden usar como fuente de eventos para otras aplicaciones.

## Lab tasks
### Configurar Streaming Queues
1. Iniciar sesión en Cloud Pak for Integration, reemplazando XX por el número asignado para los ejercicios: https://cp4i-navigator-pn-tools.apps.itz-nxsqr6.hub01-lb.techzone.ibm.com/instances

Username:
```bash
studentXX
```
Password:
```bash
studentXX-Password
```

2. Abrir consola de MQ
![image](https://github.com/user-attachments/assets/75dfc10a-2ae4-4778-95c4-5712dba4bbf3)

3. Seleccionar **Manage QMGRDEMO**
![image](https://github.com/user-attachments/assets/aa2bd921-6068-49d0-ae13-059b4ad53609)

4. Reemplazando con el número asignado, crear dos queues locales con el formato _STUDENTXX_ y _STUDENTXX.COPY_ desde la pestaña **Queues**
![image](https://github.com/user-attachments/assets/01ec3a75-6bd6-4cbf-9547-0f35c8ed5f17)
![image](https://github.com/user-attachments/assets/37b1b6ba-8cdc-4e41-bb57-537d8922a613)
![image](https://github.com/user-attachments/assets/07dd2e7a-3d1e-473a-a910-796dabdc1636)

5. En _STUDENTXX_, indicar a _STUDENTXX.COPY_ como su Streaming Queue y guardar los cambios.
![image](https://github.com/user-attachments/assets/30926cc0-8dcd-4056-bd0f-8637d6725cf3)
![image](https://github.com/user-attachments/assets/de0187d2-65ee-4b72-bdef-9bba8d220782)
![image](https://github.com/user-attachments/assets/706f96f9-5a45-410c-b2b6-7f12d430fec4)

En este punto habremos configurado a _STUDENTXX.COPY_ para ser Streaming Queue de _STUDENTXX_, haciendo que reciba una copia de los mensajes que lleguen a la queue original.

### Crear Tópico de Kafka

1. Volvemos al navigator de Cloud Pak for Integration y para ingresar a nuestra instancia de Event Streams: https://cp4i-navigator-pn-tools.apps.itz-nxsqr6.hub01-lb.techzone.ibm.com/instances

![image](https://github.com/user-attachments/assets/5a28481e-8dcf-4bd5-bfc6-104b07a2e5a3)
![image](https://github.com/user-attachments/assets/a92b483f-68e9-42c4-9aac-7b7a0545d679)

2. En la pestaña **Topics** seleccionamos **Create topic** para crear el tópico destino al cual van a llegar los mensajes desde la queue _STUDENTXX.COPY_. Para el nombre utilizamos el formato _MQ.STUDENTXX_ reemplazando con el número asignado.

![image](https://github.com/user-attachments/assets/250aed1a-3826-4768-a8c7-681bc9bfb5e3)
![image](https://github.com/user-attachments/assets/340bf74d-3cda-4144-bcfc-5290be861574)
![image](https://github.com/user-attachments/assets/3ee72932-0684-40b4-a3c3-5db720e031be)
En esta opción seleccionamos una única réplica:
![image](https://github.com/user-attachments/assets/4e9fab04-9d71-4cf7-8a04-4d3b0548f456)

3. Vemos que nuestro tópico fue creado.
![image](https://github.com/user-attachments/assets/00625768-63b9-41e0-bd10-23649fb90615)

En este punto sólo queda configurar el Kafka Connector para que los mensajes de la streaming queue empiecen a ser consumidos y replicados en nuestro tópico de Kafka.

### Configurar Kafka Connector
1. Iniciar sesión en la consola de OpenShift usando **kube_admin**: https://console-openshift-console.apps.itz-nxsqr6.hub01-lb.techzone.ibm.com/
![image](https://github.com/user-attachments/assets/2339c91f-c33b-4372-bc0c-ca5ba986e2b1)

Username:
```bash
kubeadmin
```
Password:
```bash
sSfV4-Yfci5-cRM8D-LFzrv
```

2. Utilizando el símbolo **(+)** de la esquina superior derecha vamos a importar el templato del Kafka Connector en formato YAML.
![image](https://github.com/user-attachments/assets/0690e985-1fe6-476f-866a-e19d0b7f2b2d)

**IMPORTANTE!:** reemplazar el número de student según corresponda en los campos _metadata.name_, _spec.config.topic_, y _spec.config.mq.queue_
```yaml
apiVersion: eventstreams.ibm.com/v1beta2
kind: KafkaConnector
metadata:
  name: studentxx-mq-to-es-connector
  namespace: tools
  labels:
    eventstreams.ibm.com/cluster: jgr-connect-cluster
spec:
  class: com.ibm.eventstreams.connect.mqsource.MQSourceConnector
  tasksMax: 1
  config:
    # the Kafka topic to produce to
    topic: MQ.STUDENTXX

    # the MQ queue to get messages from
    mq.queue: STUDENTXX.COPY

    # connection details for the queue manager
    mq.queue.manager: QMGRDEMO
    mq.connection.name.list: qmgr-demo-ibm-mq(1414)
    mq.channel.name: KAFKA.SVRCONN

    # format of the messages to transfer
    mq.message.body.jms: true
    mq.record.builder: com.ibm.eventstreams.connect.mqsource.builders.JsonRecordBuilder
    key.converter: org.apache.kafka.connect.storage.StringConverter
    value.converter: org.apache.kafka.connect.json.JsonConverter

    # whether to send the schema with the messages
    key.converter.schemas.enable: false
    value.converter.schemas.enable: false
```

3. Ir a **Operators -> Installed Operators** y seleccionar el operador de Event Streams. En la pestaña **Kafka Connector**, verificar que el Status del conector sea **Ready**.
![image](https://github.com/user-attachments/assets/40b7efe1-f6a3-4c40-bd15-01d7da6ec513)
![image](https://github.com/user-attachments/assets/3e13e20d-edf6-4aee-b8e0-e78c28b6ff21)

Ya tenemos todo configurado para empezar a enviar mensajes a MQ y recibirlos también en Event Streams.

### Envío de mensajes

El siguiente paso es verificar que los mensajes de MQ aparezcan en el tópico de Kafka como flujo de eventos.

1. Comprobamos que el tópico _MQ.STUDENTXX_ está vacío.
![image](https://github.com/user-attachments/assets/60121e7a-3b4c-4abe-8433-0b94b9286f48)

2. Desde la consola de MQ, enviamos el siguiente mensaje a la queue original _STUDENTXX_:
```json
{
  "transaction_id": "a1b2c3d4-e5f6-7890-abcd-1234567890ef",
  "customer": {
    "id": "98f7a6b5-c4d3-2e1f-0a9b-87654321fedc",
    "name": "Tomás Herrera"
  },
  "account": {
    "number": "123456789012",
    "type": "Cuenta Corriente",
    "balance": 12500.75
  },
  "debitcard": {
    "number": "4567123412341234",
    "expiry": "11/26"
  },
  "transaction": {
    "type": "Transferencia",
    "amount": "700000",
    "currency": "ARS",
    "description": "Pago alquiler junio"
  },
  "transaction_time": "2025-06-06 14:32:47.583"
}
```
![image](https://github.com/user-attachments/assets/e0be621f-e25f-415f-9680-798952e23f28)

3. Confirmamos que el mensaje está disponible en el tópico _MQ.STUDENTXX_ de Event Streams.
![image](https://github.com/user-attachments/assets/ea3f9c10-50c0-4b66-8fee-1febf5409e1d)

4. También podemos ver que sigue estando disponible en la cola de MQ.
![image](https://github.com/user-attachments/assets/0c1378c4-27fe-4d51-a26f-c87d871bc375)

### Transformación de mensajes

Podemos aplicar transformaciones a los mensajes que llegan desde MQ, para darles el formato requerido por la aplicación consumidora, para ofuscar datos sensibles y más. Para esto, tenemos que modificar el Kafka Connector que en los pasos anteriores.

1. Volver a la consola de OpenShift: https://console-openshift-console.apps.itz-nxsqr6.hub01-lb.techzone.ibm.com/
  
2. Ir a **Operators -> Installed Operators** y buscar el operador de Event Streams. En la pestaña **Kafka Connector**, ingresar a _studentxx-mq-to-es-connector_ y abrir la pestaña **YAML**.
![image](https://github.com/user-attachments/assets/3e13e20d-edf6-4aee-b8e0-e78c28b6ff21)
![image](https://github.com/user-attachments/assets/79b6d9dd-b1bf-44e3-8175-18f34634a743)

3. Bajo _spec:_, agregar el siguiente código utilizado para aplanar el mensaje JSON y guardar los cambios.


```yaml
transforms: flatten

transforms.flatten.type: org.apache.kafka.connect.transforms.Flatten$Value
transforms.flatten.delimiter: _
```

**Nota:** si nos dice que el objeto fue modificado, click en **Reload**, volver a agregar las transformaciones y guardar.
![image](https://github.com/user-attachments/assets/93c2461a-aed2-4f47-9b48-b26f36ca3609)

4. Enviar un nuevo mensaje a la cola de MQ y ver las transformaciones en Event Streams. El mensaje fue aplanado y ya no contiene estructuras anidadas.
**Nota:** los cambios pueden tardar un momento en aplicarse. Si al enviar el mensaje vemos que no fue transformado, esperar 30 segundos y reenviarlo.

```json
{
  "transaction_id": "f3e2d1c0-b9a8-4567-8901-abcdef123456",
  "customer": {
    "id": "ab12cd34-ef56-7890-ab12-cd34ef567890",
    "name": "Lucía Benítez"
  },
  "account": {
    "number": "987654321098",
    "type": "Caja de Ahorro",
    "balance": 84200.50
  },
  "debitcard": {
    "number": "5123987654321098",
    "expiry": "08/27"
  },
  "transaction": {
    "type": "Pago",
    "amount": "125000",
    "currency": "ARS",
    "description": "Compra electrodomésticos"
  },
  "transaction_time": "2025-06-07 09:18:22.417"
}
```

![image](https://github.com/user-attachments/assets/22d5ecbf-aac5-4a21-8d69-07de9edc6672)

```json
{
  "transaction_id": "f3e2d1c0-b9a8-4567-8901-abcdef123456",
  "transaction_time": "2025-06-07 09:18:22.417",
  "account_number": "987654321098",
  "account_balance": 84200.5,
  "account_type": "Caja de Ahorro",
  "debitcard_number": "5123987654321098",
  "debitcard_expiry": "08/27",
  "transaction_amount": "125000",
  "transaction_description": "Compra electrodomésticos",
  "transaction_currency": "ARS",
  "transaction_type": "Pago",
  "customer_name": "Lucía Benítez",
  "customer_id": "ab12cd34-ef56-7890-ab12-cd34ef567890"
}
```

El mensaje ya está listo para ser consumido por la aplicación que lee los mensajes desde Kafka. Vamos a ver algunas transformaciones adicionales que podemos hacer.

5. Volvemos a nuestro conector y eliminamos la líneas que agregamos en el paso anterior: _transforms_, _transforms.flatten.type_ y _transforms.flatten.delimiter_. Agregamos las nuevas transformaciones en el nivel de _spec_.

```yaml
transforms: flatten, redact, drop, origin, casts

transforms.flatten.type: org.apache.kafka.connect.transforms.Flatten$Value
transforms.flatten.delimiter: _

transforms.redact.type: org.apache.kafka.connect.transforms.MaskField$Value
transforms.redact.fields: debitcard_number
transforms.redact.replacement: XXXXXXXXXXXXXXXX

transforms.drop.type: org.apache.kafka.connect.transforms.ReplaceField$Value
transforms.drop.blacklist: customer_name

transforms.origin.type: org.apache.kafka.connect.transforms.InsertField$Value
transforms.origin.static.field: origin
transforms.origin.static.value: mq-connector

transforms.casts.type: org.apache.kafka.connect.transforms.Cast$Value
transforms.casts.spec: transaction_amount:int32
```

![image](https://github.com/user-attachments/assets/30f061d0-473e-4e81-825e-24d814f61279)

6. Enviamos un nuevo mensaje a MQ y vemos el resultado:

```json
{
  "transaction_id": "9c8b7a6d-5e4f-3210-ba98-76543210fedc",
  "customer": {
    "id": "cd34ef56-7890-ab12-cd34-ef5678901234",
    "name": "Federico Álvarez"
  },
  "account": {
    "number": "112233445566",
    "type": "Cuenta Corriente",
    "balance": 30500.00
  },
  "debitcard": {
    "number": "4012888888881881",
    "expiry": "03/28"
  },
  "transaction": {
    "type": "Extracción",
    "amount": "50000",
    "currency": "ARS",
    "description": "Retiro en cajero automático"
  },
  "transaction_time": "2025-06-08 17:45:11.209"
}
```

![image](https://github.com/user-attachments/assets/3580c799-67b2-4fc5-a6a8-aa54e803c512)

```json
{
  "transaction_id": "9c8b7a6d-5e4f-3210-ba98-76543210fedc",
  "debitcard_number": "XXXXXXXXXXXXXXXX",
  "account_number": "112233445566",
  "account_type": "Cuenta Corriente",
  "account_balance": 30500,
  "transaction_amount": 50000,
  "origin": "mq-connector",
  "debitcard_expiry": "03/28",
  "transaction_description": "Retiro en cajero automático",
  "transaction_type": "Extracción",
  "transaction_time": "2025-06-08 17:45:11.209",
  "customer_id": "cd34ef56-7890-ab12-cd34-ef5678901234",
  "transaction_currency": "ARS"
}
```

Notamos que:
- El conector insteró la propiedad propiedad _origin_ en el mensaje.
- El campo _debitcard_number_ fue ofuscado con caracteres X, ocultando el número real de la tarjeta por razones de seguridad.
- El campo _customer_name_ fue eliminado del mensaje para proteger la identidad del cliente.
- El campo _transaction_amount_, originalmente un string, fue convertido a un entero para facilitar su procesamiento en sistemas posteriores.
- Al igual que antes, la estructura del mensaje fue aplanada.
