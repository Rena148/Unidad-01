# Reto Unidad 01 — Ejecución en Puerto 8081 mediante Configuración Externa

## 1. Objetivo del Reto
Ejecutar la aplicación Spring Boot `backend-academico` en el puerto `8081` utilizando exclusivamente mecanismos de **configuración externa**, sin modificar la clase principal Java (`BackendAcademicoApplication.java`) ni recompilar el código fuente.

---

## 2. Salida de Versiones del Entorno

### Java Runtime / JDK (JDK 25 LTS)
```text
openjdk version "25.0.4.1" 2026-08-18 LTS
OpenJDK Runtime Environment Temurin-25.0.4.1+1 (build 25.0.4.1+1-LTS)
OpenJDK 64-Bit Server VM Temurin-25.0.4.1+1 (build 25.0.4.1+1-LTS, mixed mode, sharing)
javac 25.0.4.1
```

### Maven Wrapper
```text
Apache Maven 3.9.11 (3e54c93a704957b63ee3494413a2b544fd3d825b)
Maven home: C:\Users\Administrator\.m2\wrapper\dists\apache-maven-3.9.11\03d7e36a140982eea48e22c1dcac01d8862b2550b2939e09a0809bbc5182a5bc
Java version: 25.0.4.1, vendor: Eclipse Adoptium, runtime: C:\Users\Administrator\.gemini\antigravity\scratch\tools\jdk-25.0.4.1+1
Default locale: en_US, platform encoding: UTF-8
OS name: "windows 11", version: "10.0", arch: "amd64", family: "windows"
```

---

## 3. Resultado de las Pruebas Unitarias (`mvnw test`)

```text
[INFO] -------------------------------------------------------
[INFO]  T E S T S
[INFO] -------------------------------------------------------
[INFO] Running mx.edu.backendacademico.BackendAcademicoApplicationTests
...
2026-09-25T23:45:41.371-06:00  INFO 20000 --- [backend-academico] [           main] m.e.b.BackendAcademicoApplicationTests   : Starting BackendAcademicoApplicationTests using Java 25.0.4.1 with PID 20000
2026-09-25T23:45:41.794-06:00  INFO 20000 --- [backend-academico] [           main] m.e.b.BackendAcademicoApplicationTests   : Started BackendAcademicoApplicationTests in 0.549 seconds (process running for 1.103)
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 1.065 s -- in mx.edu.backendacademico.BackendAcademicoApplicationTests
[INFO] 
[INFO] Results:
[INFO] 
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```

---

## 4. Ejecución con Configuración Externa

### Comando ejecutado
```powershell
.\mvnw.cmd spring-boot:run '-Dspring-boot.run.arguments=--server.port=8081'
```
*(Alternativa equivalente mediante variable de entorno del sistema operativo: `$env:SERVER_PORT="8081"; .\mvnw.cmd spring-boot:run`)*

### Evidencia de arranque en Tomcat (Puerto 8081)
```text
2026-09-25T23:47:17.658-06:00  INFO 18348 --- [backend-academico] [           main] o.s.boot.tomcat.TomcatWebServer          : Tomcat initialized with port 8081 (http)
2026-09-25T23:47:17.861-06:00  INFO 18348 --- [backend-academico] [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2026-09-25T23:47:17.862-06:00  INFO 18348 --- [backend-academico] [           main] o.apache.catalina.core.StandardEngine    : Starting Servlet engine: [Apache Tomcat/11.0.22]
2026-09-25T23:47:17.886-06:00  INFO 18348 --- [backend-academico] [           main] b.w.c.s.WebApplicationContextInitializer : Root WebApplicationContext: initialization completed in 325 ms
2026-09-25T23:47:18.026-06:00  INFO 18348 --- [backend-academico] [           main] o.s.boot.tomcat.TomcatWebServer          : Tomcat started on port 8081 (http) with context path '/'
2026-09-25T23:47:18.028-06:00  INFO 18348 --- [backend-academico] [           main] m.e.b.BackendAcademicoApplication        : Started BackendAcademicoApplication in 0.673 seconds (process running for 0.832)
```

### Comprobación HTTP (`curl -i http://localhost:8081/`)
```http
HTTP/1.1 404 
Content-Type: application/json
Transfer-Encoding: chunked
Date: Sat, 26 Sep 2026 05:47:41 GMT

{"timestamp":"2026-09-26T05:47:41.874Z","status":404,"error":"Not Found","path":"/"}
```

---

## 5. Explicación Técnica Razonada: ¿Por qué cambiar el puerto no exige recompilar la lógica Java?

1. **Separación de Responsabilidades (Build-Time vs. Run-Time):**
   La compilación Java (`javac`) traduce código estático a bytecode independiente del entorno de despliegue. `BackendAcademicoApplication.java` no tiene valores numéricos de red codificados en duro (*hardcoded*); únicamente invoca `SpringApplication.run(BackendAcademicoApplication.class, args)`.

2. **Mecanismo de Configuración Externalizada de Spring Boot:**
   Spring Boot implementa un modelo jerárquico de fuentes de propiedades (`PropertySource`). Al iniciar, la infraestructura de Spring construye un `ConfigurableEnvironment` e incorpora los argumentos pasados por línea de comandos (`SimpleCommandLinePropertySource`) con la máxima prioridad, por encima de las variables de entorno del sistema operativo y de los archivos `application.properties` o `application.yml`.

3. **Inyección Dinámica en la Fábrica de Servidores Embebidos:**
   El servidor web embebido (Apache Tomcat) no se configura estáticamente durante la compilación. En su lugar, durante el ciclo de vida de arranque del `ApplicationContext`, la clase autoconfigurada `ServletWebServerFactoryConfiguration` crea un bean `TomcatServletWebServerFactory`, el cual resuelve la propiedad `server.port` evaluando el `Environment` en tiempo de ejecución. 

4. **Principio de Portabilidad de Doce Factores (12-Factor App - Config):**
   Este desacoplamiento cumple con el principio de aplicaciones modernas nativas de la nube: el mismo artefacto inmutable (el archivo JAR generado en la fase de empaquetado) puede desplegarse en desarrollo, pruebas locales o contenedores de producción en cualquier puerto o host simplemente suministrando variables de entorno o argumentos CLI, garantizando la reproducibilidad y evitando reconstrucciones innecesarias.
