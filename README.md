# Lab 04 — Microservicios con JWT, H2, Swagger, OpenFeign y Docker

## Descripción

Solución compuesta por dos microservicios:

- **auth-service**: autenticación de usuarios, validación de contraseñas cifradas con BCrypt y generación de tokens JWT.
- **consulta-service**: consulta de estudiantes protegida por token JWT validado mediante OpenFeign.

---

## Estructura del proyecto

```
Lab4/
├── auth-service/
│   ├── Dockerfile
│   ├── pom.xml
│   └── src/
├── consulta-service/
│   ├── Dockerfile
│   ├── pom.xml
│   └── src/
├── demo-academico/
│   └── src/
└── docker-compose.yml
```

---

## Cómo ejecutar los servicios localmente

### Requisitos previos

- Java 21
- Maven 3.9+
- Docker Desktop en ejecución

### Compilar auth-service

```bash
cd auth-service
mvn clean package -DskipTests
```

### Compilar consulta-service

```bash
cd consulta-service
mvn clean package -DskipTests
```

### Ejecutar auth-service

```bash
cd auth-service
mvn spring-boot:run
```

### Ejecutar consulta-service (en otra terminal)

```bash
cd consulta-service
mvn spring-boot:run
```

---

## Cómo levantar la solución con Docker Compose

Desde la carpeta raíz `Lab4`:

```bash
docker compose -f docker-compose.yml up -d --build
```

Para detener y eliminar los contenedores:

```bash
docker compose -f docker-compose.yml down --rmi all
```

---

## URLs de Swagger y H2 Console

| Servicio | Swagger | H2 Console |
|---|---|---|
| auth-service | http://localhost:8081/swagger-ui.html | http://localhost:8081/h2-console |
| consulta-service | http://localhost:8082/swagger-ui.html | http://localhost:8082/h2-console |

### Configuración H2 Console

| Campo | Valor |
|---|---|
| JDBC URL | `jdbc:h2:mem:authdb` (auth) / `jdbc:h2:mem:demoacademico` (consulta) |
| User Name | `sa` |
| Password | _(vacío)_ |

---

## Usuarios de prueba

| Usuario | Contraseña | Rol |
|---|---|---|
| admin | Admin2026* | ADMIN |
| docente | Docente2026* | DOCENTE |
| estudiante1 | Estu2026* | ESTUDIANTE |

---

## Flujo de uso del botón Authorize en Swagger

1. Abrir Swagger de **auth-service**: http://localhost:8081/swagger-ui.html
2. Ejecutar **POST /auth/login** con un usuario válido
3. Copiar únicamente el valor del campo `token` de la respuesta (empieza con `eyJ...`)
4. Abrir Swagger de **consulta-service**: http://localhost:8082/swagger-ui.html
5. Hacer clic en el botón **Authorize** 🔒 (arriba a la derecha)
6. Pegar el token **sin** escribir la palabra `Bearer`
7. Hacer clic en **Authorize** y luego en **Close**
8. Ejecutar **GET /api/estudiantes** — Swagger enviará automáticamente el header `Authorization: Bearer <token>`

> **Importante:** No escribir `Bearer` antes del token en el campo Authorize. Swagger lo agrega automáticamente.

---

## Flujo de autenticación entre microservicios

```
Cliente
   │
   │ POST /auth/login
   ▼
auth-service
   │ busca usuario en H2
   │ valida contraseña con BCrypt
   │ genera JWT
   ▼
Cliente
   │
   │ GET /api/estudiantes
   │ Authorization: Bearer <token>
   ▼
consulta-service
   │ intercepta la petición (JwtAuthenticationFilter)
   │ llama a auth-service via OpenFeign
   ▼
auth-service
   │ valida el token
   ▼
consulta-service
   │ permite acceso al recurso
   ▼
Cliente recibe lista de estudiantes
```

---

## Tecnologías utilizadas

- Spring Boot 3.5.14
- Spring Security
- Spring Data JPA
- H2 Database (embebida)
- JWT (jjwt 0.11.5)
- BCrypt
- OpenFeign
- Springdoc OpenAPI (Swagger)
- JavaFaker
- Docker & Docker Compose
- Java 21 / Maven 3.9
