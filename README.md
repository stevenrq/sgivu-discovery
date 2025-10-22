# 🧩 SGIVU - Servicio de Descubrimiento

## 📘 Descripción

Servicio de descubrimiento (Eureka Server) que centraliza el registro y la ubicación dinámica de todos los
microservicios del ecosistema SGIVU. Facilita el balanceo de carga, la alta disponibilidad y elimina la necesidad de
configurar endpoints estáticos entre servicios.

## 🧱 Arquitectura y Rol

* Tipo: Microservicio Spring Boot 3 / Spring Cloud Netflix Eureka Server.
* Interactúa con: `sgivu-config`, `sgivu-gateway`, el resto de microservicios clientes de Eureka (`sgivu-auth`,
  `sgivu-user`, etc.).
* Provee UI de administración en `http://localhost:8761/` y endpoints REST de Eureka para el registro de instancias (
  `/eureka/apps/**`).
* Registra las instancias de cada microservicio en Eureka y expone su catálogo para consumidores internos.
* Obtiene configuración centralizada desde Config Server (`configserver:http://sgivu-config:8888`) con el perfil `prod`.
* No persiste datos; toda la información de registro se mantiene en memoria y se reconstruye dinámicamente.

## ⚙️ Tecnologías

* **Lenguaje:** Java 21 (Amazon Corretto en Docker)
* **Framework:** Spring Boot 3.5.6, Spring Cloud 2025.0.0, Netflix Eureka Server
* **Seguridad:** Sin autenticación integrada aún; operación en red privada detrás de `sgivu-gateway`
* **Persistencia:** No aplica (estado en memoria)
* **Infraestructura:** Docker, AWS (EC2)

## 🚀 Ejecución Local

1. Clona este repositorio y sitúate en `sgivu-discovery`.
2. Asegúrate de tener disponible `sgivu-config` en `http://localhost:8888` o ajusta `SPRING_CONFIG_IMPORT`.
3. Exporta variables si deseas sobrescribir configuración:

   ```bash
   export SPRING_PROFILES_ACTIVE=prod
   export SPRING_CONFIG_IMPORT=configserver:http://localhost:8888
   ```

4. Ejecuta el servicio:

   ```bash
   ./mvnw spring-boot:run
   ```

5. Accede al dashboard de Eureka en `http://localhost:8761`.

## 🔗 Endpoints Principales

```
GET /
GET /eureka/apps
GET /eureka/apps/{applicationName}
```

* `GET /`: Panel web de Eureka con el estado de los servicios.
* `GET /eureka/apps`: Catálogo completo de instancias registradas (consumido por clientes internos).
* `GET /eureka/apps/{applicationName}`: Detalle de instancias para un servicio específico.

## 🔐 Seguridad

Actualmente se ejecuta sin OAuth2 ya que opera en una red interna protegida por `sgivu-gateway`. Está planificada la
integración con `sgivu-auth` para endurecer el acceso a los endpoints REST (`/eureka/**`) mediante JWT y roles de
servicio.

## 🧩 Dependencias

* `sgivu-config` (configuración centralizada)
* `sgivu-discovery` (este servicio, como registro de servicios)
* `sgivu-gateway` (enrutamiento y protección perimetral)
* Microservicios clientes que se registran en Eureka

## 🧮 Dockerización

* Contenedor: `sgivu-discovery`
* Puerto expuesto: `8761`
* Ejemplo de ejecución:

  ```bash
  ./mvnw clean package -DskipTests
  docker build -t sgivu-discovery .
  docker run --rm -p 8761:8761 \
    -e SPRING_CONFIG_IMPORT=configserver:http://host.docker.internal:8888 \
    sgivu-discovery
  ```

## ☁️ Despliegue en AWS

* Desplegar en una instancia EC2 dentro de la VPC privada.
* Configurar variables de entorno `SPRING_CONFIG_IMPORT` apuntando al Config Server gestionado.
* Abrir el puerto 8761 solo para la subred interna o para el balanceador de carga.
* Registrar `sgivu-config` y `sgivu-gateway` en la misma VPC para garantizar conectividad.

## 📊 Monitoreo

* Integrable con Micrometer + Prometheus a través de Actuator (`management.endpoints.web.exposure.include=*` en
  configuraciones futuras).
* Puede enviar trazas a Zipkin al habilitar `spring.zipkin.baseUrl` desde Config Server.
* Supervisión mínima mediante `GET /actuator/health` (tras añadir dependencia `spring-boot-starter-actuator`).

## ✨ Autor

* **Steven Ricardo Quiñones**
