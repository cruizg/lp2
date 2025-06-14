# **Guía de Laboratorio: Creando una Aplicación Web Full-Stack con Spring Boot, Thymeleaf,lombok y Supabase**

## 1. Objetivo

¡Felicidades por llegar a la etapa final de nuestro viaje de desarrollo! En este laboratorio, transformaremos nuestra aplicación de consola en una **aplicación web completamente funcional e interactiva**.

Aprenderemos a construir una interfaz de usuario utilizando **Thymeleaf** (el motor de plantillas preferido de Spring) y a darle un aspecto profesional y moderno con **Bootstrap**, aunque seria ideal trabajarlo en tailwindCSS.

Lo más importante es que dejaremos atrás la base de datos temporal en memoria y nos conectaremos a una **base de datos PostgreSQL real en la nube**, gracias al servicio de **Supabase**.

**¿Por qué estamos usando estas tecnologías?**

*   **Spring Boot**: Es el framework que nos permite construir la aplicación de forma rápida y robusta, manejando toda la configuración compleja por nosotros.
*   **Thymeleaf**: Nos permite crear páginas HTML dinámicas. En lugar de tener páginas estáticas, podremos "inyectar" datos desde nuestro código Java (como una lista de estudiantes) directamente en el HTML que ve el usuario.
*   **Bootstrap**: Es un framework de diseño (CSS) que nos da componentes listos para usar (tablas, botones, formularios) para que nuestra aplicación se vea bien sin necesidad de ser expertos en diseño web.
*   **Supabase (PostgreSQL)**: Al usar una base de datos en la nube, nuestros datos se vuelven **persistentes**. La información que guardemos seguirá existiendo incluso si reiniciamos o cerramos la aplicación. Esto simula un entorno de producción real y nos prepara para proyectos más grandes.

Al final de este laboratorio, habrás creado, desde cero, una aplicación web que puede Crear, Leer, Editar y Borrar estudiantes, con todos los datos almacenados de forma segura en la nube.

## 2. Configuración del Proyecto y la Base de Datos en la Nube

### Paso 2.1: Crear Nuestra Base de Datos Gratuita en Supabase

Supabase nos ofrece una base de datos PostgreSQL en su plan gratuito, ideal para nuestros proyectos.

1.  **Crear una cuenta**: Ve a [https://supabase.com/](https://supabase.com/) y regístrate para obtener una cuenta gratuita. Es un proceso rápido.
2.  **Crear un Nuevo Proyecto**:
    *   Una vez dentro de tu panel de control (dashboard), haz clic en **"New Project"**.
    *   Asígnale un nombre a tu proyecto (ej: `matricula-app-cibertec`).
    *   Genera una **contraseña de base de datos segura** y, lo más importante, **¡cópiala y guárdala en un lugar seguro!** La necesitarás en unos minutos.
    *   Elige la región más cercana a tu ubicación para una menor latencia.
    *   Haz clic en **"Create new project"**. La configuración de la base de datos puede tardar un par de minutos.
3.  **Obtener los Datos de Conexión a la Base de Datos**:
    *   Cuando el proyecto esté listo, navega a la configuración del proyecto haciendo clic en el ícono del engranaje (⚙️ **Settings**) en el menú de la izquierda.
    *   En el nuevo menú, selecciona la opción **"Database"**.
    *   Busca la sección **"Connection string"** y asegúrate de que la pestaña **"URI"** esté seleccionada.
    *   Verás una cadena de conexión. De aquí extraeremos los siguientes datos: `postgres://postgres:[YOUR-PASSWORD]@[HOST]:5432/postgres`
        *   **Host**: El valor que está después de `@` y antes de `:5432` (ej: `db.abcdefgh.supabase.co`).
        *   **Port**: `5432`.
        *   **User**: `postgres`.
        *   **Password**: La contraseña que guardaste al crear el proyecto.
        *   **Database Name**: `postgres`.

### Paso 2.2: Generar el Proyecto con Spring Initializr

Usaremos la herramienta oficial de Spring para crear la estructura de nuestro proyecto.

1.  Ve a [https://start.spring.io/](https://start.spring.io/).
2.  Configura los metadatos del proyecto:
    *   **Project**: Maven
    *   **Language**: Java
    *   **Spring Boot**: La versión estable más reciente (ej: 3.2.x).
    *   **Group**: `com.cibertec`
    *   **Artifact**: `matricula-webapp`
    *   **Packaging**: Jar
    *   **Java**: 17 (o la versión que tengas instalada).
3.  En la sección de **Dependencies**, haz clic en "ADD DEPENDENCIES" y añade las siguientes cuatro librerías:
    *   **Spring Web**: Fundamental para crear aplicaciones web, controladores y APIs REST.
    *   **Spring Data JPA**: Para la persistencia de datos y la comunicación con la base de datos.
    *   **Thymeleaf**: Nuestro motor para crear las plantillas HTML dinámicas.
    *   **PostgreSQL Driver**: El "traductor" específico que necesita Java para comunicarse con nuestra base de datos PostgreSQL en Supabase.
4.  Haz clic en el botón **GENERATE**. Se descargará un archivo `.zip`.
5.  Descomprime el archivo y abre la carpeta del proyecto con tu IDE (Visual Studio Code, IntelliJ, etc.).

### Paso 2.3: Conectar Nuestra Aplicación con la Base de Datos

Ahora le diremos a Spring cómo encontrar nuestra base de datos en Supabase. Abre el archivo `src/main/resources/application.properties` y reemplaza todo su contenido con lo siguiente, sustituyendo los valores entre corchetes con tu información real de Supabase.

```properties
# ===================================================================
# CONFIGURACIÓN DE LA BASE DE DATOS (SUPABASE/POSTGRESQL)
# ===================================================================
# Esta es la URL de conexión JDBC para PostgreSQL.
# Reemplaza [HOST] con el host que copiaste de Supabase.
spring.datasource.url=jdbc:postgresql://[HOST]:5432/postgres

# El usuario por defecto en Supabase es 'postgres'.
spring.datasource.username=postgres

# Pega aquí la contraseña que guardaste al crear el proyecto en Supabase.
spring.datasource.password=[TU-CONTRASEÑA-DE-SUPABASE]

# ===================================================================
# CONFIGURACIÓN DE JPA/HIBERNATE
# ===================================================================
# Le dice a Hibernate que "hable" en el dialecto correcto para PostgreSQL.
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect

# Esta es una propiedad muy poderosa. 'update' significa que Hibernate
# comparará tus clases @Entity con las tablas en la base de datos y
# creará o modificará las tablas automáticamente para que coincidan.
spring.jpa.hibernate.ddl-auto=update

# Muestra en la consola de tu IDE el SQL que Hibernate está ejecutando.
# Es extremadamente útil para depurar y entender qué está pasando.
spring.jpa.show-sql=true
# ===================================================================
# OPTIMIZACIÓN DEL POOL DE CONEXIONES (HIKARI CP)
# ¡IMPORTANTE PARA ENTORNOS CON MÚLTIPLES USUARIOS!
# ===================================================================
# Número máximo de conexiones que la aplicación puede abrir. 
# El plan gratuito de Supabase tiene un límite, revisa tu dashboard (suele ser ~60). 
# Un valor entre 20 y 30 es un buen punto de partida para una clase.
spring.datasource.hikari.maximum-pool-size=3

# Número mínimo de conexiones inactivas que Hikari intentará mantener.
# Esto mejora la velocidad de respuesta para nuevas peticiones.
spring.datasource.hikari.minimum-idle=1

# Tiempo máximo (en ms) que la app esperará por una conexión del pool.
# 30 segundos es un valor seguro.
spring.datasource.hikari.connection-timeout=30000

# Tiempo máximo (en ms) que una conexión puede estar inactiva en el pool
# antes de ser retirada. 10 minutos es un buen valor.
spring.datasource.hikari.idle-timeout=600000
```

## 3. Construcción del Backend y la Lógica de Negocio

### Paso 3.1: Definir la Entidad `Estudiante`

Las clases `@Entity` son el corazón de nuestro modelo de datos. Son el plano que JPA usará para crear las tablas en la base de datos.

1.  Crea el paquete `com.cibertec.matriculawebapp.model`.
2.  Dentro, crea la clase `Estudiante.java`.

```java
package com.cibertec.matriculawebapp.model;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;

@Entity // Le dice a JPA que esta clase es una tabla en la BD.
public class Estudiante {

    @Id // Marca este campo como la clave primaria (Primary Key).
    @GeneratedValue(strategy = GenerationType.IDENTITY) // Le dice a la BD que genere el ID automáticamente.
    private Long id;

    private String nombre;
    private String apellido;

    // JPA requiere un constructor vacío.
    public Estudiante() {}

    // Constructor para crear nuevos estudiantes.
    public Estudiante(String nombre, String apellido) {
        this.nombre = nombre;
        this.apellido = apellido;
    }

    // Getters y Setters son necesarios para que Spring y Thymeleaf puedan acceder a los datos.
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getNombre() { return nombre; }
    public void setNombre(String nombre) { this.nombre = nombre; }
    public String getApellido() { return apellido; }
    public void setApellido(String apellido) { this.apellido = apellido; }
}
```

### Paso 3.2: Crear el Repositorio de Datos

**Teoría:** El Repositorio es una interfaz que nos da todos los métodos para interactuar con la base de datos (`save`, `findById`, `findAll`, `delete`) sin que tengamos que escribir una sola línea de SQL. Spring Data JPA lo implementa por nosotros en segundo plano.

1.  Crea el paquete `com.cibertec.matriculawebapp.repository`.
2.  Dentro, crea la interfaz `EstudianteRepository.java`.

```java
package com.cibertec.matriculawebapp.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import com.cibertec.matriculawebapp.model.Estudiante;

// Al extender JpaRepository<Estudiante, Long>, obtenemos todo el CRUD para la entidad Estudiante.
public interface EstudianteRepository extends JpaRepository<Estudiante, Long> {
}
```

### Paso 3.3: Crear el Controlador Web (El Cerebro de la Aplicación)

**Teoría:** El Controlador es el que recibe las peticiones del navegador del usuario. Decide qué hacer, pide datos al Repositorio y finalmente le dice a Thymeleaf qué página HTML debe mostrar.

1.  Crea el paquete `com.cibertec.matriculawebapp.controller`.
2.  Dentro, crea la clase `EstudianteController.java`.

```java
package com.cibertec.matriculawebapp.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;

import com.cibertec.matriculawebapp.model.Estudiante;
import com.cibertec.matriculawebapp.repository.EstudianteRepository;

@Controller // Marca esta clase como un controlador web de Spring.
public class EstudianteController {

    @Autowired // Inyección de dependencias: Spring nos da una instancia del repositorio.
    private EstudianteRepository estudianteRepository;

    // --- READ: Muestra la lista de todos los estudiantes ---
    @GetMapping("/estudiantes")
    public String listarEstudiantes(Model model) {
        // Model es el objeto que usamos para pasar datos del controlador a la vista.
        model.addAttribute("listaEstudiantes", estudianteRepository.findAll());
        return "listado-estudiantes"; // Devuelve el nombre del archivo HTML a renderizar.
    }

    // --- CREATE (Paso 1): Muestra el formulario para un nuevo estudiante ---
    @GetMapping("/estudiantes/nuevo")
    public String mostrarFormularioDeNuevoEstudiante(Model model) {
        Estudiante estudiante = new Estudiante();
        model.addAttribute("estudiante", estudiante);
        return "formulario-estudiante";
    }

    // --- CREATE (Paso 2): Procesa y guarda el estudiante del formulario ---
    @PostMapping("/estudiantes")
    public String guardarEstudiante(@ModelAttribute("estudiante") Estudiante estudiante) {
        // @ModelAttribute vincula los datos del formulario al objeto Estudiante.
        estudianteRepository.save(estudiante);
        return "redirect:/estudiantes"; // Redirige al usuario de vuelta a la lista.
    }

    // --- UPDATE (Paso 1): Muestra el formulario para editar un estudiante existente ---
    @GetMapping("/estudiantes/editar/{id}")
    public String mostrarFormularioDeEditar(@PathVariable Long id, Model model) {
        // @PathVariable obtiene el 'id' de la URL.
        Estudiante estudiante = estudianteRepository.findById(id)
                .orElseThrow(() -> new IllegalArgumentException("Id de estudiante inválido:" + id));
        model.addAttribute("estudiante", estudiante);
        return "formulario-estudiante"; // Reutilizamos el mismo formulario.
    }
    
    // --- UPDATE (Paso 2): Procesa y guarda los cambios del estudiante editado ---
    // NOTA: Este método se fusiona con guardarEstudiante. El save() de JPA es inteligente:
    // si el objeto tiene un ID, hace un UPDATE; si no lo tiene, hace un INSERT.

    // --- DELETE: Elimina un estudiante ---
    @GetMapping("/estudiantes/eliminar/{id}")
    public String eliminarEstudiante(@PathVariable Long id) {
        estudianteRepository.deleteById(id);
        return "redirect:/estudiantes";
    }
}
```

## 4. Creación de la Interfaz de Usuario con Thymeleaf

Spring Boot buscará automáticamente nuestros archivos HTML en la carpeta `src/main/resources/templates`.

### Paso 4.1: Crear la Plantilla para Listar Estudiantes

Este archivo mostrará la tabla con todos los estudiantes y los botones de acción.

Crea el archivo `src/main/resources/templates/listado-estudiantes.html`:

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Sistema de Matrícula</title>
    <!-- Incluimos Bootstrap para los estilos desde un CDN -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <div class="container mt-4">
        <h1 class="mb-4">Sistema de Matrícula de Estudiantes</h1>
        
        <a th:href="@{/estudiantes/nuevo}" class="btn btn-primary mb-3">Registrar Nuevo Estudiante</a>

        <table class="table table-striped table-bordered table-hover">
            <thead class="table-dark">
                <tr>
                    <th>ID</th>
                    <th>Nombre</th>
                    <th>Apellido</th>
                    <th>Acciones</th>
                </tr>
            </thead>
            <tbody>
                <!-- th:each es un bucle que itera sobre la lista que pasamos desde el controlador -->
                <tr th:each="estudiante : ${listaEstudiantes}">
                    <td th:text="${estudiante.id}">ID</td>
                    <td th:text="${estudiante.nombre}">Nombre</td>
                    <td th:text="${estudiante.apellido}">Apellido</td>
                    <td>
                        <!-- Enlace para editar -->
                        <a th:href="@{/estudiantes/editar/{id}(id=${estudiante.id})}" class="btn btn-warning btn-sm">Editar</a>
                        <!-- Enlace para eliminar -->
                        <a th:href="@{/estudiantes/eliminar/{id}(id=${estudiante.id})}" class="btn btn-danger btn-sm">Eliminar</a>
                    </td>
                </tr>
            </tbody>
        </table>
    </div>
</body>
</html>
```

### Paso 4.2: Crear la Plantilla del Formulario (para Crear y Editar)

Reutilizaremos este mismo formulario tanto para registrar nuevos estudiantes como para editar los existentes.

Crea el archivo `src/main/resources/templates/formulario-estudiante.html`:

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Formulario de Estudiante</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <div class="container mt-4">
        <div class="card">
            <div class="card-header">
                <!-- Título dinámico: si el estudiante tiene ID, es una edición; si no, es un registro. -->
                <h3 th:if="${estudiante.id == null}">Registrar Nuevo Estudiante</h3>
                <h3 th:unless="${estudiante.id == null}">Editar Estudiante</h3>
            </div>
            <div class="card-body">
                <!-- th:action envía el formulario a /estudiantes. th:object vincula el form con nuestro objeto Java. -->
                <form th:action="@{/estudiantes}" th:object="${estudiante}" method="POST">
                    
                    <!-- Campo oculto para el ID. Es crucial para que la actualización (UPDATE) funcione. -->
                    <input type="hidden" th:field="*{id}" />

                    <div class="mb-3">
                        <label for="nombre" class="form-label">Nombre del Estudiante:</label>
                        <!-- th:field vincula este input con el atributo 'nombre' del objeto Estudiante. -->
                        <input type="text" class="form-control" id="nombre" th:field="*{nombre}" required>
                    </div>
                    <div class="mb-3">
                        <label for="apellido" class="form-label">Apellido del Estudiante:</label>
                        <input type="text" class="form-control" id="apellido" th:field="*{apellido}" required>
                    </div>
                    
                    <button type="submit" class="btn btn-success">Guardar Cambios</button>
                    <a th:href="@{/estudiantes}" class="btn btn-secondary">Cancelar</a>
                </form>
            </div>
        </div>
    </div>
</body>
</html>
```

## 5. Ejecución y Prueba de la Aplicación Completa

1.  **Inicia la Aplicación**:
    *   En tu IDE, haz clic derecho en la clase `MatriculaWebappApplication.java` y selecciona "Run" o "Debug".
    *   O, desde una terminal en la raíz del proyecto, ejecuta: `mvn spring-boot:run`.
2.  **Abre tu Navegador Web**:
    *   Ve a la dirección [http://localhost:8080/estudiantes](http://localhost:8080/estudiantes).
3.  **Prueba el Flujo CRUD**:
    *   **CREATE**: Haz clic en "Registrar Nuevo Estudiante". Rellena el formulario y guarda. Deberías ver al nuevo estudiante en la lista.
    *   **READ**: La lista que ves es la prueba de la funcionalidad de lectura.
    *   **UPDATE**: Haz clic en el botón "Editar" de cualquier estudiante. Modifica sus datos y guarda. Deberías ver los cambios reflejados en la lista.
    *   **DELETE**: Haz clic en "Eliminar" en la fila de un estudiante. Debería desaparecer de la lista.
4.  **Verifica en la Nube**:
    *   Abre tu panel de Supabase en el navegador.
    *   En el menú de la izquierda, ve al **Table Editor** (ícono de tabla).
    *   Verás una tabla llamada `estudiante` creada por Hibernate. Haz clic en ella.
    *   **¡Ahí están tus datos!** Cada cambio que hiciste en tu aplicación web local se ha guardado permanentemente en tu base de datos en la nube.

## 6. Conclusión

¡Felicidades! Has completado la transición de una simple aplicación de consola a una **aplicación web full-stack, moderna y robusta**.

En este laboratorio has dominado habilidades fundamentales:
*   Configurar y conectar una aplicación Spring Boot a una **base de datos PostgreSQL en la nube** con Supabase.
*   Construir el **backend** con el patrón Controlador-Repositorio.
*   Desarrollar una **interfaz de usuario dinámica y responsiva** con Thymeleaf y Bootstrap.
*   Implementar un **flujo CRUD completo** (Crear, Leer, Editar, Borrar) a través de una interfaz web intuitiva.
*   Entender cómo los datos persisten en un entorno real, desacoplado de tu aplicación local.

Este proyecto representa una base sólida y realista sobre la cual puedes construir aplicaciones web mucho más complejas y con más funcionalidades. ¡El siguiente paso es tuyo
