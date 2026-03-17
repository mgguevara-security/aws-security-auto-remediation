\#   Corrección de un incidente con AWS Config y Lambda


\##  Descripción del Proyecto

Este informe documenta la implementación de un sistema de cumplimiento continuo en AWS. El objetivo es supervisar los Grupos de Seguridad (Security Groups) de Amazon EC2 para detectar configuraciones no deseadas y aplicar una remediación automática mediante funciones Lambda, asegurando que solo los puertos autorizados permanezcan abiertos.



\---



\## 🎯 Objetivos Técnicos

\* \*\*Gestión de Identidades:\*\* Configurar y actualizar roles de IAM para permitir la interacción entre AWS Config, Lambda y EC2.

\* \*\*Inventario de Recursos:\*\* Habilitar AWS Config para el monitoreo continuo de tipos de recursos específicos.

\* \*\*Automatización Defensiva:\*\* Implementar una regla de AWS Config personalizada vinculada a una función Lambda para corregir desvíos de configuración.

\* \*\*Auditoría y Verificación:\*\* Analizar registros en Amazon CloudWatch Logs para validar las acciones de remediación.



\---



\## 🛠️ Stack Tecnológico

\* \*\*Servicios Cloud:\*\* AWS Config, AWS Lambda, Amazon EC2, IAM, CloudWatch Logs.

\* \*\*Lenguajes y SDK:\*\* Python (Boto3).

\* \*\*Infraestructura:\*\* VPC, Security Groups.



\---



\## 🚀 Implementación Detallada



\### Tarea 1: Examinar y Actualizar Roles de IAM

Se analizaron y configuraron los permisos necesarios para la operación:

\* \*\*AwsConfigLambdaSGRole:\*\* Rol para la función Lambda con permisos para revocar reglas de entrada en EC2 y escribir logs.

\* \*\*AwsConfigRole:\*\* Se actualizó asociando la política gestionada `AWS\_ConfigRole` para permitir que el servicio supervise los recursos de la cuenta.



\### Tarea 2: Configuración de AWS Config

Se activó el servicio AWS Config con los siguientes parámetros:

\* \*\*Estrategia de grabación:\*\* Tipos de recursos específicos (`AWS EC2 SecurityGroup`).

\* \*\*Frecuencia:\*\* Grabación continua.

\* \*\*Canal de entrega:\*\* Almacenamiento de resultados en un bucket de Amazon S3.



\### Tarea 3: Simulación del Incidente de Seguridad

Para probar la eficacia del sistema, se modificó el grupo de seguridad \*\*LabSG1\*\* añadiendo reglas de entrada no autorizadas:

\* \*\*Permitidos originalmente:\*\* HTTP (80).

\* \*\*Añadidos manualmente (Incidente):\*\* HTTPS (443), SMTPS (465) e IMAPS (993).

\* \*\*Origen:\*\* 0.0.0.0/0 (Anywhere-IPv4).



\### Tarea 4: Creación de la Regla de AWS Config y Lambda

Se creó una regla personalizada en AWS Config denominada `EC2SecurityGroup`:

\* \*\*Trigger:\*\* "Cuando cambia la configuración".

\* \*\*Ámbito:\*\* Recursos de tipo `AWS EC2 SecurityGroup`.

\* \*\*Lógica:\*\* La regla invoca la función Lambda `awsconfig\_lambda\_security\_group`, pasando parámetros de depuración (`debug: true`).



\### Tarea 5: Revisión de la Remediación Automática

Tras la evaluación inicial, se comprobó que la función Lambda intervino en el grupo de seguridad \*\*LabSG1\*\*:

\* \*\*Estado Final:\*\* Solo permanecieron activos los puertos HTTP y HTTPS.

\* \*\*Resultado:\*\* Las reglas de SMTPS (465) e IMAPS (993) fueron eliminadas automáticamente por la función Lambda al no cumplir con la línea base de seguridad.



\### Tarea 6: Verificación en CloudWatch Logs

Se realizó una auditoría en el grupo de logs `/aws/lambda/awsconfig\_lambda\_security\_group`:

\* \*\*Filtro aplicado:\*\* `revoking for`.

\* \*\*Evidencia:\*\* Se encontraron eventos confirmando la revocación de permisos para los puertos 465 y 993, proporcionando una trazabilidad completa del incidente y su resolución.



\---



\## 🛡️ Lógica de la Solución (Snippet de Código)

La función Lambda utiliza el siguiente bloque de control para definir el cumplimiento:



```python

\# Baseline de seguridad: Solo puertos 80 y 443 permitidos

REQUIRED\_PERMISSIONS = \[

&#x20;   {

&#x20;       'IpProtocol': 'tcp',

&#x20;       'FromPort': 80,

&#x20;       'ToPort': 80,

&#x20;       'IpRanges': \[{'CidrIp': '0.0.0.0/0'}]

&#x20;   },

&#x20;   {

&#x20;       'IpProtocol': 'tcp',

&#x20;       'FromPort': 443,

&#x20;       'ToPort': 443,

&#x20;       'IpRanges': \[{'CidrIp': '0.0.0.0/0'}]

&#x20;   }

]





\---



\## 📈 Conclusiones y Análisis de Seguridad

\* \*\*Seguridad Proactiva:\*\* La solución elimina el factor del error humano en la configuración de perímetros de red.

\* \*\*Visibilidad:\*\* AWS Config proporciona un inventario histórico que es vital para auditorías de cumplimiento.

\* \*\*Eficiencia:\*\* El uso de arquitecturas Serverless (Lambda) permite una respuesta a incidentes inmediata y de bajo costo.

\* \*\*Principio de Menor Privilegio:\*\* Se demostró la importancia de configurar roles de IAM específicos para evitar que los servicios tengan más permisos de los necesarios.



\---

\*\*Informe generado por Marcos Gastón Guevara - Analista en ciberseguridad \*\*

