# Sistema de Gestión de Servicios de Salud - IPS

Proyecto académico para la gestión de servicios de una IPS.

## Tecnologías
- Java 17
- Spring Boot 3.2.0
- Maven 3.9.9
- PostgreSQL 16
- Docker
- Docker Compose
- Angular 15

## Arquitectura
- backend: API REST con Spring Boot.
- db: base de datos PostgreSQL.

Ambos servicios se comunican mediante la red Docker ips-network.
El backend utiliza el nombre del servicio db para conectarse a PostgreSQL.

## Configuración
Crear un archivo .env tomando como referencia .env.example.
Configurar las credenciales antes de iniciar.

## Ejecutar el proyecto

Construir e iniciar:
sudo docker compose up --build -d

Ver contenedores:
sudo docker compose ps

Ver registros:
sudo docker compose logs -f backend

Detener:
sudo docker compose down

## Prueba de comunicación

POST:
http://localhost:8082/api/servicio/guardarServicioDto2

GET:
http://localhost:8082/api/servicio/listaServicioDto2

La aplicación permite guardar y consultar servicios utilizando PostgreSQL.

## Repositorios
Frontend:
https://github.com/Brainlaer/ServiciosSaludFrontend

Backend:
https://github.com/Brainlaer/ServiciosSaludBackend

## Integración continua con Jenkins

La guía de instalación, configuración de credenciales, jobs, webhooks y evidencias está en [Jenkins-Entrega.md](Jenkins-Entrega.md).

El `Jenkinsfile` construye las imágenes del backend y del frontend y ejecuta una prueba de comunicación por el nombre del servicio. Para que el pipeline pueda construir ambas imágenes, Jenkins clona el repositorio del frontend en la carpeta `frontend`.
