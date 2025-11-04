# nginx-tls-gateway

## Descripción

Este proyecto implementa un **gateway de terminación TLS** utilizando NGINX como reverse proxy. Su función principal es recibir tráfico HTTPS cifrado, descifrarlo, y reenviarlo a un backend (WAF/API Gateway) a través de una red interna.

## ¿Qué hace este proyecto?

El `nginx-tls-gateway` actúa como punto de entrada seguro para las peticiones HTTPS:

1. **Terminación TLS/SSL**: Recibe conexiones HTTPS en el puerto 443 y maneja el cifrado/descifrado de las comunicaciones usando certificados SSL/TLS.

2. **Proxy reverso**: Después de descifrar el tráfico, lo reenvía al servicio backend (`api-gateway:8888`) que actúa como WAF (Web Application Firewall).

3. **Seguridad**: Implementa protocolos TLS modernos (TLSv1.2 y TLSv1.3) y cifrados seguros para proteger las comunicaciones.

## Arquitectura

```
Cliente (Internet)
    ↓ HTTPS (443)
[nginx-tls-gateway]
    ↓ HTTP (8888)
[api-gateway/WAF]
    ↓
[Servicios Backend]
```

## Componentes

### Docker Compose

- **Servicio**: `tls-terminator`
- **Imagen**: nginx:latest
- **Puerto expuesto**: 443 (HTTPS)
- **Redes**:
  - `orchestration_net`: Red de orquestación para comunicación con el API Gateway
  - `frontend_net`: Red frontend para servicios públicos
- **Volúmenes**:
  - `nginx.conf`: Configuración de NGINX
  - `certs/`: Certificados SSL (fullchain.pem y privkey.pem)

### Configuración NGINX

- Utiliza el módulo **stream** para manejar tráfico TCP/TLS
- Define un upstream hacia `api-gateway:8888`
- Configuración SSL con certificados personalizados
- Protocolos seguros: TLSv1.2 y TLSv1.3
- Cifrados seguros: HIGH:!aNULL:!MD5

## Requisitos Previos

- Docker y Docker Compose instalados
- Certificados SSL válidos en el directorio `certs/`:
  - `fullchain.pem`: Certificado completo
  - `privkey.pem`: Clave privada
- Redes Docker externas creadas:
  - `orchestration_net`
  - `frontend_net`

## Instalación

### 1. Crear las redes Docker (si no existen)

```bash
docker network create orchestration_net
docker network create frontend_net
```

### 2. Colocar los certificados SSL

Coloca tus certificados SSL en el directorio `certs/`:

```
certs/
  ├── fullchain.pem
  └── privkey.pem
```

**Nota**: Si necesitas generar certificados de prueba, puedes usar OpenSSL:

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout certs/privkey.pem \
  -out certs/fullchain.pem \
  -subj "/CN=localhost"
```

### 3. Iniciar el servicio

```bash
docker-compose up -d
```

## Verificación

### Comprobar que el contenedor está ejecutándose

```bash
docker-compose ps
```

### Ver los logs

```bash
docker-compose logs -f tls-terminator
```

### Probar la conexión HTTPS

```bash
curl -k https://localhost
```

## Configuración

### Cambiar el backend

Para modificar el servidor backend, edita el archivo `nginx.conf`:

```nginx
upstream waf_backend {
    server tu-servicio:puerto;
}
```

### Ajustar protocolos SSL

Puedes modificar los protocolos y cifrados permitidos en `nginx.conf`:

```nginx
ssl_protocols TLSv1.2 TLSv1.3;
ssl_ciphers HIGH:!aNULL:!MD5;
```

## Detener el servicio

```bash
docker-compose down
```

## Seguridad

- ✅ Terminación TLS/SSL centralizada
- ✅ Protocolos TLS modernos (1.2 y 1.3)
- ✅ Cifrados seguros configurados
- ✅ Certificados montados como volúmenes de solo lectura
- ⚠️ Asegúrate de usar certificados válidos en producción
- ⚠️ Mantén los certificados actualizados

## Troubleshooting

### El contenedor no inicia

```bash
# Verificar logs
docker-compose logs tls-terminator

# Verificar que los certificados existen
ls -l certs/
```

### Error de red externa no encontrada

```bash
# Crear las redes manualmente
docker network create orchestration_net
docker network create frontend_net
```

### Error de certificados

Verifica que los archivos de certificados:

- Existen en el directorio `certs/`
- Tienen los permisos correctos
- Son certificados válidos

