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

4. Crear dos queues locales con el formato _STUDENTXX_ y _STUDENTXX.COPY_ desde la pestaña **Queues**
![image](https://github.com/user-attachments/assets/01ec3a75-6bd6-4cbf-9547-0f35c8ed5f17)
![image](https://github.com/user-attachments/assets/37b1b6ba-8cdc-4e41-bb57-537d8922a613)
![image](https://github.com/user-attachments/assets/07dd2e7a-3d1e-473a-a910-796dabdc1636)

5. En _STUDENTXX_, configurar a _STUDENTXX.COPY_ como su Streaming Queue y guardar los cambios.
![image](https://github.com/user-attachments/assets/30926cc0-8dcd-4056-bd0f-8637d6725cf3)
![image](https://github.com/user-attachments/assets/de0187d2-65ee-4b72-bdef-9bba8d220782)
![image](https://github.com/user-attachments/assets/706f96f9-5a45-410c-b2b6-7f12d430fec4)

En este punto habremos configurado a _STUDENTXX.COPY_ para ser Streaming Queue de _STUDENTXX_, haciendo que reciba una copia de los mensajes que lleguen a la queue original.

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

2. Ir a **Operators -> Installed Operators** y seleccionar el operador de Event Streams.
![image](https://github.com/user-attachments/assets/40b7efe1-f6a3-4c40-bd15-01d7da6ec513)
