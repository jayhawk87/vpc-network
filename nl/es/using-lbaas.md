---



copyright:
  years: 2018, 2019
lastupdated: "2019-02-20"


---

<!-- Common attributes used in the template are defined as follows: -->
{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:note: .note}
{:screen: .screen}
{:tip: .tip}
{:note: .note}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# (Beta) Utilización de equilibradores de carga en la VPC de IBM Cloud
{: #--beta-using-load-balancers-in-ibm-cloud-vpc}

El servicio de equilibrador de carga distribuye el tráfico entre varias instancias de servidor de la misma región para el {{site.data.keyword.cloud}} VPC.

**Este servicio ahora dispone de una versión de release beta. Los recursos utilizados durante el período beta no estarán disponibles después del cierre de la versión beta y no se admitirán. Indique sus comentarios y preguntas al representante de ventas de IBM Cloud.**

## Equilibrador de carga público
{: #public-load-balancer}

A la instancia de servicio del equilibrador de carga se le ha asignado un nombre de dominio completo y accesible públicamente (FQDN), que debe utilizar para acceder a las aplicaciones alojadas detrás del servicio de equilibrador de carga de la VPC de {{site.data.keyword.cloud}}. Es posible registrar este nombre de dominio con una o varias direcciones IP públicas.

Con el tiempo, dichas direcciones IP públicas y el número de direcciones IP públicas pueden cambiar en función de las actividades de mantenimiento y escalado, que son transparentes para los clientes. Las instancias de servicio de fondo (VSI) que alojan la aplicación deben ejecutarse en la misma región y en el mismo VPC.

## Escuchas frontales y agrupaciones de fondo
Puede definir hasta 10 escuchas frontales (puertos de aplicación) y correlacionarlas con las respectivas agrupaciones de fondo en los servidores de aplicaciones de fondo. El nombre de dominio completo (FQDN) asignado a la instancia de servicio del equilibrador de carga y a los puertos de escucha frontales están expuestos a Internet público. Las solicitudes de usuario entrantes se reciben en dichos puertos.

Los protocolos de escucha frontales soportados son:
* HTTP
* HTTPS
* TCP

Los protocolos de agrupación de fondo soportados son:
* HTTP
* TCP

El tráfico HTTPS entrante finaliza en el equilibrador de carga para permitir la comunicación HTTP en texto sin formato con el servidor de fondo.

Puede adjuntar hasta 50 instancias de servidor a una agrupación de fondo. Cada instancia de servidor adjunta debe tener un puerto configurado. El puerto puede ser o no el mismo que el puerto de escucha frontal.

El rango de puertos de 56500 a 56520 está reservado para fines de gestión; estos puertos no se pueden utilizar como puertos de escucha frontales.
{: note}

## Métodos de equilibrio de carga
{: #load-balancing-methods}

Los siguientes tres métodos de equilibrio de carga están disponibles para distribuir el tráfico entre los servidores de aplicaciones back-end:

* **Round-robin:** round-robin es el método de equilibrio de carga predeterminado. Con este método, el equilibrador de carga reenvía las conexiones entrantes del cliente de forma rotativa a los servidores back-end. En consecuencia, todos los servidores back-end reciben aproximadamente el mismo número de conexiones de cliente.

* **Round-robin ponderado:** con este método, el equilibrador de carga reenvía las conexiones entrantes del cliente a los servidores back-end en proporción al peso asignado a estos servidores. Cada servidor tiene asignado un peso predeterminado de 50, valor que se puede personalizar del 0 al 100.

A modo de ejemplo, si tres servidores de aplicaciones A, B y C tienen ponderaciones personalizadas de 60, 60 y 30 respectivamente, los servidores A y B reciben un número igual de conexiones, mientras que el servidor C recibe la mitad de conexiones.

* **Menos conexiones** con este método, la instancia de servidor que sirve el menor número de conexiones en un momento dado recibe la siguiente conexión de cliente.

**Características adicionales de estos métodos:**

* Si vuelve a establecer el peso en '0', no se reenviarán nuevas conexiones a dicho servidor, pero el tráfico existente continuará fluyendo siempre que esté activo. La utilización de un peso de '0' puede ayudar a desactivar un servidor y eliminarlo de la rotación de servicio.
* Los valores de ponderación de servidor solo se aplican cuando se utiliza el método 'round-robin ponderado'. Se pasan por alto con los métodos de equilibrio de carga round-robin y menos conexiones.

## Comprobaciones de estado
{: #health-checks}

Las definiciones de comprobación de estado son obligatorias para las agrupaciones de fondo.

El equilibrador de carga lleva a cabo comprobaciones de estado periódicas para supervisar el estado de los puertos de fondo y reenvía el tráfico de cliente en consecuencia. Si un puerto de servidor de fondo tiene mal estado, no se le reenviarán nuevas conexiones. El equilibrador de carga continúa supervisando el estado de los puertos con mal estado y reanuda su uso cuando vuelven a tener un buen estado, lo que significa que pasan correctamente dos intentos de comprobación de estado consecutivos.

Las comprobaciones de estado de los puertos HTTP y TCP se llevan a cabo de la forma siguiente:

* **HTTP:** se envía una solicitud `HTTP GET` con un URL especificado previamente al puerto del servidor back-end. Se marca que el puerto del servidor se encuentra en buen estado tras haber recibido una respuesta `200 OK`. La vía de acceso del estado de `GET` predeterminada es "/" mediante la IU, y se puede personalizar.

* **TCP:** el equilibrador de carga intenta abrir una conexión con el servidor back-end en un puerto TCP especificado. Se marca que el puerto del servidor se encuentra en buen estado si la conexión finaliza correctamente y la conexión se cierra.

El intervalo de comprobación de estado predeterminado es 5 segundos, el tiempo de espera excedido predeterminado de una solicitud de comprobación de estado es 2 segundos y el número predeterminado de reintentos es 2.
{: note}

## Descarga de SSL y autorizaciones necesarias

Para todas las conexiones HTTPS de entrada, el servicio de equilibrador de carga finaliza la conexión SSL y establece una comunicación HTTP en texto sin formato con la instancia de servidor de fondo. Con esta técnica, el reconocimiento SSL intensivo de CPU y las tareas de cifrado o descifrado se desplazan de las instancias de servidor de fondo, permitiéndoles utilizar todos los ciclos de CPU para procesar el tráfico de aplicaciones.

Se necesita un certificado SSL para que el equilibrador de carga pueda realizar tareas de descarga de SSL. Puede gestionar los certificados SSL a través de [IBM Certificate Manager ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")](/docs/services/certificate-manager?topic=certificate-manager-gettingstarted){: new_window}.

Si necesita acceso del equilibrador de carga al certificado SSL en la instancia del gestor de certificados, cree una autorización que proporcione a la instancia de servicio equilibrador de carga acceso a la instancia del gestor de certificados que contiene el certificado SSL. Puede gestionar dicha autorización a través de [autorizaciones de identidad y acceso ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")](/docs/services/iam/#/authorizations){: new_window}. Asegúrese de elegir **Servicio de infraestructura** como servicio de origen, **Equilibrador de carga para VPC** como tipo de recurso y **Gestor de certificados** como servicio de destino y de asignar el rol de acceso de servicio **Escritor**. Para crear un equilibrador de carga debe otorgar la autorización **Todas las instancias de recurso** a la instancia de recurso de origen. La instancia de servicio de destino puede ser **Todas las instancias** o la instancia de recurso de su gestor de certificados específico.

Si se elimina la autorización necesaria, es posible que se produzcan errores en el equilibrador de carga.
{: note}

## Identity and Access Management (IAM)

Puede configurar políticas de acceso para la instancia del **Equilibrador de carga para VPC**. Para gestionar sus políticas de acceso de usuario, visite [Autorizaciones de identidad y acceso ![Icono de enlace externo](../icons/launch-glyph.svg "Icono de enlace externo")](/docs/services/iam/#/authorizations){: new_window}.

### Configuración de políticas de acceso de grupo de recursos para los usuarios

Para crear un equilibrador de carga necesitará acceso a un grupo de recursos. El usuario que está creando un equilibrador de carga debe tener el acceso adecuado al grupo de recursos proporcionado o al grupo de recursos predeterminado si no se proporciona ninguno.

1. Vaya a **Gestionar > Cuenta > Usuarios**. Verá una lista de los usuarios con acceso a su cuenta de IBM Cloud.
2. Seleccione el nombre del usuario al que desea asignar una política de acceso. Si no se muestra el usuario, pulse **Invitar usuarios** para añadir el usuario a la cuenta de IBM Cloud.
3. Seleccione **Asignar acceso**.
4. Seleccione **Asignar acceso dentro de un grupo de recursos**.
5. En la lista desplegable **Grupo de recursos**, seleccione el grupo de recursos deseado.
6. En la lista desplegable **Asignar acceso a un grupo de recursos**, seleccione el acceso deseado.
7. En la lista desplegable **Servicios**, seleccione los servicios deseados.
9. Seleccione **Asignar** para asignar la política de acceso del grupo de recursos al usuario.

### Configuración de políticas de acceso de recursos para los usuarios

| Rol de acceso a la plataforma | Acción de equilibrador de carga |
|-------------|-----|
| Administrador | Crear/Visualizar/Editar/Suprimir equilibrador de carga |
| Editor | Crear/Visualizar/Editar/Suprimir equilibrador de carga |
| Visor | Ver equilibrador de carga |

1. Vaya a **Gestionar > Cuenta > Usuarios**. Verá una lista de los usuarios con acceso a su cuenta de IBM Cloud.
2. Seleccione el nombre del usuario al que desea asignar una política de acceso. Si no se muestra el usuario, pulse **Invitar usuarios** para añadir el usuario a la cuenta de IBM Cloud.
3. Seleccione **Asignar acceso**.
4. Seleccione **Asignar acceso a recursos**.
5. En la lista desplegable **Servicios**, seleccione **Servicios de infraestructura**.
6. En la lista desplegable **Tipo de recurso**, seleccione **Equilibrador de carga para VPC**.
7. En la lista desplegable **ID de equilibrador de carga**, seleccione un ID de instancia de equilibrador de carga o utilice el valor predeterminado **Todos los equilibradores de carga**.
8. Asigne un rol de acceso de plataforma al usuario.
9. Seleccione **Asignar** para asignar la política de acceso al usuario.

## API disponibles

Para realizar llamadas de API debe utilizar algún tipo de cliente REST. Por ejemplo, puede utilizar el mandato `curl` para recuperar todos los equilibradores de carga existentes:

```
curl -X GET $rias_endpoint/v1/load_balancers?version=2019-01-01 -H "Authorization: $iam_token"
```
{: pre}

En la sección siguiente se proporcionan detalles sobre las API que puede utilizar para el equilibrador de carga en el entorno VPC. Para ver la especificación completa, revise la [Consulta de API de RIAS](https://{DomainName}/apidocs/rias#retrieves-all-load-balancers).

| Descripción | API |
|-------------|-----|
| Crea y suministra un equilibrador de carga | POST /load_balancers |
| Recupera todos los equilibradores de carga | GET /load_balancers |
| Recupera un equilibrador de carga | GET /load_balancers/{id} |
| Suprime un equilibrador de carga | DELETE /load_balancers/{id} |
| Actualiza un equilibrador de carga | PATCH /load_balancers/{id} |
| Crea una escucha | POST /load_balancers/{id}/listeners |
| Recupera todas las escuchas del equilibrador de carga | GET /load_balancers/{id}/listeners |
| Recupera una escucha | GET /load_balancers/{id}/listeners/{listener_id} |
| Suprime una escucha | DELETE /load_balancers/{id}/listeners/{listener_id} |
| Actualiza una escucha | PATCH /load_balancers/{id}/listeners/{listener_id} |
| Crea una agrupación | POST /load_balancers/{id}/pools |
| Recupera todas las agrupaciones del equilibrador de carga | GET /load_balancers/{id}/pools |
| Recupera una agrupación | GET /load_balancers/{id}/pools/{pool_id} |
| Suprime una agrupación | DELETE /load_balancers/{id}/pools/{pool_id} |
| Actualiza una agrupación | PATCH /load_balancers/{id}/pools/{pool_id} |
| Crea un miembro | POST /load_balancers/{id}/pools/{pool_id}/members |
| Recupera todos los miembros de la agrupación | GET /load_balancers/{id}/pools/{pool_id}/members |
| Recupera un miembro |GET /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| Suprime un miembro de la agrupación | DELETE /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| Actualiza un miembro | PATCH /load_balancers/{id}/pools/{pool_id}/members/{member_id} |
| Actualiza miembros de la agrupación | PUT /load_balancers/{id}/pools/{pool_id}/members |

## Ejemplo de equilibrador de carga

En el ejemplo siguiente, utilizará la API para crear un equilibrado de carga delante de 2 instancias de servidor de VPC (`192.168.100.5` y `192.168.100.6`) que ejecutan una aplicación web que está a la escucha en el puerto `80`. El equilibrador de carga tiene una escucha frontal que permite el acceso a la aplicación web de forma segura mediante HTTPS. A continuación, puede utilizar la API para obtener detalles de la instancia del equilibrador de carga una vez creada y para suprimir la misma instancia.

### Pasos de ejemplo

En los pasos siguientes del ejemplo se omiten los pasos de requisito previo de utilizar la [interfaz de usuario de IBM Cloud](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console), la [CLI](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli) o la [API de RIAS](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis) para suministrar una VPC, subredes e instancias. 


Los pasos de ejemplo del equilibrador de carga también se pueden ejecutar utilizando la [CLI](/docs/cli/reference/ibmcloud?topic=infrastructure-service-cli-vpc-reference#vpc-reference).
{: note}

#### Paso 1. Crear un equilibrador de carga con las instancias de escucha, agrupación y servidor adjunto (miembros)

```bash
curl -H "Authorization: $iam_token" -X POST
$rias_endpoint/v1/load_balancers?version=2019-01-01 \
    -d '{
        "name": "example-balancer",
        "is_public": true,
        "listeners": [
            {
                "certificate_instance": {
                    "crn": "crn:v1:bluemix:public:cloudcerts:us-south:a/123456:b8877ea4-b8eg-467e-912a-da1eb7f031cg:certificate:43219c4c97d013fb2a95b21dddde1234"
                },
                "port": 443,
                "protocol": "https",
                "default_pool": {
                    "name": "example-pool"
                }
            }
        ],
        "pools": [
            {
                "algorithm": "round_robin",
                "health_monitor": {
                    "delay": 5,
                    "max_retries": 2,
                    "timeout": 2,
                    "type": "http",
                    "url_path": "/"
                },
                "name": "example-pool",
                "protocol": "http",
                "session_persistence": {
                    "cookie_name": "string",
                    "type": "source_ip"
                },
                "members": [
                    {
                        "port": 80,
                        "target": {
                            "address": "192.168.100.5"
                        },
                        "weight": 50
                    },
                    {
                        "port": 80,
                        "target": {
                            "address": "192.168.100.6"
                        },
                        "weight": 50
                    }
                ]
            }
        ],
        "subnets": [
            {
                "id": "7ec87131-1c7e-4990-b4f0-a26f2e61f98e"
            }
        ]
        }'
```
{: codeblock}

Salida de ejemplo:
```
{
    "created_at": "2018-07-12T23:17:07.5985381Z",
    "crn": "crn:v1:bluemix:public:is:us-south:a/123456::load-balancer:dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
    "hostname": "ac34687d-7ce5-413b-aae4-90a2a88b7872.lb.appdomain.cloud",
    "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
    "id": "dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
    "is_public": true,
    "listeners": [
        {
            "id": "70294e14-4e61-11e8-bcf4-0242ac110004",
            "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/listeners/70294e14-4e61-11e8-bcf4-0242ac110004"
        }
    ],
    "name": "example-balancer",
    "operating_status": "offline",
    "pools": [
        {
            "id": "70294e14-4e61-11e8-bcf4-0242ac110004",
            "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/pools/70294e14-4e61-11e8-bcf4-0242ac110004",
            "name": "example-pool"
        }
    ],
    "provisioning_status": "create_pending",
    "resource_group": {
        "id": "56969d60-43e9-465c-883c-b9f7363e78e8"
    },
    "subnets": [
        {
            "id": "7ec86020-1c6e-4889-b3f0-a15f2e50f87e",
            "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/7ec86020-1c6e-4889-b3f0-a15f2e50f87e",
            "name": "example-subnet"
        }
    ]
}
```
{: screen}

Guarde el ID del equilibrador de carga para utilizarlo en los pasos siguientes, por ejemplo en la variable `lbid`.

```
lbid=dd754295-e9e0-4c9d-bf6c-58fbc59e5727
```

#### Paso 2. Obtener un equilibrador de carga

```
curl -H "Authorization: $iam_token" -X GET $rias_endpoint/v1/load_balancers/$lbid?version=2019-01-01
```
{: pre}

El suministro del equilibrador de carga tarda un rato. Cuando esté listo, se establecerá en el estado `en línea` y `activo`, tal como verá en la siguiente salida de ejemplo:

Salida de ejemplo:

```bash
{
  "id": "dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
  "crn": "crn:v1:bluemix:public:is:us-south:a/123456::load-balancer:dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
  "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727",
  "name": "example-balancer",
  "created_at": "2018-07-13T22:22:24.489Z",
  "hostname": "dd754295-e9e0-4c9d-bf6c-58fbc59e5727.lb.appdomain.cloud",
  "is_public": true,
  "listeners": [
    {
      "id": "70294e14-4e61-11e8-bcf4-0242ac110004",
      "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/listeners/70294e14-4e61-11e8-bcf4-0242ac110004"
    }
  ],
  "operating_status": "online",
  "pools": [
    {
      "id": "70294e14-4e61-11e8-bcf4-0242ac110004",
      "href": "https://us-south.iaas.cloud.ibm.com/v1/load_balancers/dd754295-e9e0-4c9d-bf6c-58fbc59e5727/pools/70294e14-4e61-11e8-bcf4-0242ac110004",
      "name": "example-pool"
    }
  ],
  "private_ips": [
    {
      "address": "192.168.10.5"
    },
    {
      "address": "192.168.10.6"
    }
  ],
  "provisioning_status": "active",
  "public_ips": [
    {
        "address": "169.11.111.115"
    },
    {
        "address": "169.11.111.116"
    }
  ],
  "resource_group": {
    "id": "56969d60-43e9-465c-883c-b9f7363e78e8"
  },
  "subnets": [
    {
      "id": "7ec86020-1c6e-4889-b3f0-a15f2e50f87e",
      "href": "https://us-south.iaas.cloud.ibm.com/v1/subnets/7ec86020-1c6e-4889-b3f0-a15f2e50f87e",
      "name": "example-subnet"
    }
  ]
}
```
{: screen}

#### Paso 3. Suprimir un equilibrador de carga

```bash
curl -H "Authorization: $iam_token" -X DELETE $rias_endpoint/v1/load_balancers/$lbid?version=2019-01-01
```
{: pre}

## Preguntas más frecuentes

Esta sección contiene respuestas a algunas de las preguntas más frecuentes sobre el servicio **Equilibrador de carga para VPC**.

### ¿Puedo utilizar un nombre de DNS distinto para mi equilibrador de carga?

El nombre de DNS asignado automáticamente para el equilibrador de carga no se puede personalizar. Sin embargo, puede añadir un registro CNAME (nombre canónico) que indique su nombre de DNS preferido en el nombre de DNS del equilibrador de carga asignado automáticamente. Por ejemplo, si el ID del equilibrador de carga es `dd754295-e9e0-4c9d-bf6c-58fbc59e5727`, el nombre de DNS del equilibrador de carga asignado automáticamente es `dd754295-e9e0-4c9d-bf6c-58fbc59e5727.lb.appdomain.cloud`. Su nombre de DNS preferido es `www.myapp.com`. Puede añadir un registro de CNAME (mediante el proveedor de DNS que utiliza para gestionar `myapp.com`) que indique `www.myapp.com` en el nombre de DNS del equilibrador de carga `dd754295-e9e0-4c9d-bf6c-58fbc59e5727.lb.appdomain.cloud`.

### ¿Cuál es el número máximo de escuchas frontales que puedo definir con mi equilibrador de carga?

10.

### ¿Cuál es el número máximo de instancias de servidor que puedo conectar a mi agrupación de fondo?

50.

### ¿Puedo crear un equilibrador de carga privado de uso interno al que solo tengan acceso los clientes internos?  

No, no en este momento.

### ¿Es el equilibrador de carga escalable horizontalmente?  

No, no en este momento.

### ¿Qué debo hacer si estoy utilizando las ACL o los grupos de seguridad en las subredes que se utilizan para desplegar el equilibrador de carga?

Deberá asegurarse de que las reglas de grupo de seguridad o de ACL adecuadas estén en vigor para permitir el tráfico de entrada a puertos de escucha y de gestión configurados (los puertos del 56500 al 56520). También se debe permitir el tráfico entre el equilibrador de carga y las instancias de fondo.

### ¿Por qué recibo el mensaje de error: `no se encuentra la instancia de certificado`?

* Es posible que el CRN de la instancia de certificado no sea válido.
* Es posible que no disponga de autorización de servicio a servicio. Consulte la sección **Descarga de SSL** de este documento.

### ¿Por qué recibo el código `401 Unauthorized Error`?

Compruebe las políticas de acceso siguientes de su usuario:
* Política de acceso del tipo de recurso del equilibrador de carga
* Política de acceso del grupo de recursos

### ¿Por qué mi balanceador de carga está en el estado `maintenance_pending`?

El equilibrador de carga tendrá el estado `maintenance_pending` durante diversas actividades de mantenimiento como las siguientes:
* Cualquier actualización continua para solucionar vulnerabilidades y aplicar parches de seguridad
* Actividades de recuperación

### ¿Por qué tengo que elegir varias subredes durante el suministro?

Los dispositivos del equilibrador de carga se despliegan en las subredes que ha seleccionado. Se recomienda elegir subredes de distintas zonas para obtener una mayor disponibilidad y redundancia.
