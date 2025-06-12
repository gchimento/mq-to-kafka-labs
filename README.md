# MQ to Kafka labs: configurando Streaming Queues y Kafka Connect
Lorem ipsum

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

| Nº | Nombre         |
|----|----------------|
| 1  | Juan Pérez     |
| 2  | María Gómez    |
| 3  | Lucas Fernández|
| 4  | Ana Torres     |
| 5  | Diego Ramírez  |
| 6  | Sofía Méndez   |
| 7  | Martín Rivas   |

2. Abrir consola de MQ
![image](https://github.com/user-attachments/assets/75dfc10a-2ae4-4778-95c4-5712dba4bbf3)

3. Seleccionar **Manage QMGRDEMO**
![image](https://github.com/user-attachments/assets/aa2bd921-6068-49d0-ae13-059b4ad53609)

4. Reemplazando con el número asignado, crear dos queues locales con el formato _STUDENTXX_ y _STUDENTXX.COPY_ desde la pestaña **Queues**
![image](https://github.com/user-attachments/assets/01ec3a75-6bd6-4cbf-9547-0f35c8ed5f17)
![image](https://github.com/user-attachments/assets/37b1b6ba-8cdc-4e41-bb57-537d8922a613)
![image](https://github.com/user-attachments/assets/07dd2e7a-3d1e-473a-a910-796dabdc1636)

5. En _STUDENTXX_, configurar a _STUDENTXX.COPY_ como su Streaming Queue y guardar los cambios.
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

99. Ir a **Operators -> Installed Operators** y seleccionar el operador de Event Streams.
![image](https://github.com/user-attachments/assets/40b7efe1-f6a3-4c40-bd15-01d7da6ec513)
