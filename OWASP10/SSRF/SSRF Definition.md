_________
#ssrf

- En un ataque SSRF, el atacante manipula al **servidor** para que haga solicitudes HTTP hacia otros sistemas, incluyendo servicios internos de la red que no están expuestos al público.

_______

-  "Es como usar la solicitud HTML del servidor (víctima) para que haga una solicitud por nosotros (a recursos internos o externos)."

____

- Aceptan URLs como parametros de entrada para recuperar datos
     Mostrar imagenes o importar contenido desde otro sitio

____

- Fijarse en que parametros estan enviando informacion o fijarse en parametros suceptibles de enviar/solicitar informacion

___________

***La clave es entender que un SSRF ocurre cuando **el servidor actúa como un cliente**. Es decir, cuando le pides al servidor que él mismo vaya a buscar algo a otra parte.***

### 1. Parámetros que aceptan URLs o Direcciones IP

Cualquier campo de entrada que le diga al servidor "ve allí y tráeme esto" es sospechoso. Busca parámetros con nombres como:

- `?url=` / `?uri=`
- `?dest=` / `?destination=`
- `?path=`
- `?redirect=` / `?callback=`
- `?request=`
- `?api_url=`
- `?image_url=` (Muy común en importación de fotos de perfil).

### 2. Funcionalidades "Proxy" o de Importación

Si la web tiene herramientas que interactúan con otros sitios, ahí hay un vector. Ejemplos clásicos:

- **Webhooks:** "Configura una URL para avisarte cuando pase X evento".
- **Importar desde URL:** Subir un archivo usando un link en lugar de subirlo desde tu PC.
- **Generadores de PDF/Capturas de pantalla:** Le das una URL y el servidor la renderiza para generarte un documento.
- **Lectores de RSS:** El servidor tiene que ir a buscar el XML del feed.
### 3. Respuestas de Tiempo (Time-based Analysis)

Esta es la señal más sutil pero efectiva.

- Si envías una URL a un servidor interno que **existe** (como `http://10.0.0.1`), la respuesta del servidor suele ser rápida (aunque sea un error 403 o 401).
- Si envías una URL a una IP que **no existe**, el servidor se quedará pensando hasta que el "timeout" expire.
- **Conclusión:** Si los tiempos de respuesta varían según la IP que pongas, el servidor está intentando conectar.

### 4. Diferencias en el Status Code o el Banner

A veces el servidor no te devuelve el contenido, pero te da pistas en el error:

- **200 OK:** El servidor llegó al destino.
- **500 Internal Server Error:** El servidor llegó, pero el servicio (ej. base de datos interna) no supo qué hacer con la petición.
- **404 Not Found:** El servidor llegó a la IP, pero no encontró el archivo/ruta específica.
- **Banner Grabbing:** Si el error dice algo como `Server: Apache/2.2.8 (Ubuntu) PHP/5.2.4`, y la web principal usa Nginx, acabas de descubrir un servicio interno diferente.

### El "Truco del Espejo" (External Interaction)

La forma más rápida de confirmar un SSRF (si es que la máquina tiene salida a internet) es usar un **DNS/HTTP Collaborator** (como el de Burp o `interact.sh`):

1. Pones tu URL única en el parámetro sospechoso: `?url=http://tu-id.burpcollaborator.net`.
    
2. Si recibes una visita en tu servidor de control, **el SSRF está confirmado**. El servidor "mordió el anzuelo" y salió a buscarte.
    


______________


Un atacante podría realizar consultas a servicios internos de la empresa para:

- Robar datos sensibles como credenciales de usuarios o archivos de sistema.
- Hacer peticiones a servicios internos para manejar el panel de administración, escanear puertos y servicios dentro de la infraestructura interna y conectarse al servidor de correos para enviar correos sin autorización.
- Escalar privilegios dentro del sistema y ejecutar código de forma remota dentro del servidor.

## Cómo funciona un SSRF

Cuando una aplicación hace una llamada a una URL (por ejemplo a una API), un atacante puede podría editar la consulta para que esa llamada se haga a otra dirección. 

Así, podría acceder a un panel de administración sin autorización, ver archivos dentro del sistema con información sensible o incluso escanear puertos dentro de la red interna de ese servidor.




[Server-Side Request Forgery (SSRF) | Jackie0x17](https://j4ckie0x17.gitbook.io/notes-pentesting/pentesting-web/server-side-request-forgery-ssrf)