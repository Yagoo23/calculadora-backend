# Requisitos Previos
Antes de comenzar, asegúrate de tener instalados los siguientes componentes:
- **Docker**: Para la contenedorización de aplicaciones.
- **Git**: Para el control de versiones y colaboración.
- **Java 17+**: Como lenguaje de programación principal.
- **Maven**: Para la gestión de dependencias y construcción del proyecto.
# Estructura del Proyecto
```
.
├── .github/
│   └── workflows/
│       └── build-and-test.yml   # Configuración de GitHub Actions
├── src/                         # Código fuente de la aplicación
├── pom.xml                      # Archivo de configuración Maven
├── Dockerfile                   # Dockerfile para la aplicación
└── README.md                    # Este archivo
```
# Dockerfile
El Dockerfile está diseñado en dos etapas para optimizar el tamaño de la imagen final. Aquí está el contenido:
```
# Etapa 1: Construcción del proyecto
FROM maven:3.9.4-eclipse-temurin-17 AS build

# Configurar el directorio de trabajo en el contenedor
WORKDIR /app

# Copiar archivos necesarios para resolver dependencias
COPY pom.xml ./
RUN mvn dependency:go-offline

# Copiar el resto del proyecto y construir
COPY src ./src
RUN mvn clean package -DskipTests

# Etapa 2: Generación de la imagen final
FROM openjdk:17-jdk-slim

# Configurar el directorio de trabajo en el contenedor final
WORKDIR /app

# Copiar el artefacto generado desde la etapa anterior
COPY --from=build /app/target/*.jar app.jar

# Exponer el puerto de la aplicación
EXPOSE 8080

# Comando para ejecutar la aplicación
CMD ["java", "-jar", "app.jar"]
```

# Construcción y Ejecución con Docker
Paso 1: Construir la Imagen
Ejecuta el siguiente comando para construir la imagen Docker:
```
bash
docker build -t mi-proyecto .
```

Paso 2: Ejecutar el Contenedor
Corre el contenedor con:
```
bash
docker run -p 8080:8080 mi-proyecto
```

Paso 3: Subir a DockerHub (Opcional)
Si quieres subir la imagen a DockerHub, autentícate y realiza el push:
```
bash
docker tag mi-proyecto usuario/mi-proyecto
docker push usuario/mi-proyecto
```

# Configuración de GitHub Actions
El pipeline automatiza la compilación, pruebas, creación del artefacto .jar y generación de la imagen Docker.
Aquí está el archivo .github/workflows/build-and-test.yml:
```
text
name: Build, Test, and Package with Docker

on:
  push:
    branches:
      - main
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v3

    - name: Set up JDK 17
      uses: actions/setup-java@v3
      with:
        distribution: 'temurin'
        java-version: '17'

    - name: Cache Maven dependencies
      uses: actions/cache@v3
      with:
        path: ~/.m2
        key: ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}
        restore-keys: |
          ${{ runner.os }}-maven-

    - name: Install dependencies and run tests
      run: mvn clean test

    - name: Build Docker image
      run: docker build -t usuario/mi-proyecto .

    - name: Log in to DockerHub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}

    - name: Push Docker image
      run: docker push usuario/mi-proyecto
```
# Instrucciones para Configurar el Pipeline
Configurar Secrets de DockerHub:
En GitHub, ve a Settings > Secrets and Variables > Actions.
Agrega:
```
DOCKER_USERNAME: Tu usuario de DockerHub.
DOCKER_PASSWORD: Tu contraseña de DockerHub.
```
# Branch Principal:
El pipeline se ejecuta automáticamente en la rama main o al crear un Pull Request.
