
¡Excelente! Ahora que la base funciona, vamos a llevar la guía al siguiente nivel, tal como lo propones.

Incorporaremos **Lombok** para reducir drásticamente el código repetitivo, mencionaremos **TailwindCSS** como una alternativa moderna, y lo más importante, implementaremos la relación **Muchos-a-Muchos** (`Estudiante` <> `Curso`) en la interfaz web, creando **consultas personalizadas** en nuestros repositorios para manejarla.

Esta es la versión definitiva y mejorada de la guía.

---

# **Guía de Laboratorio (Versión Avanzada): Aplicación Full-Stack con Spring, Thymeleaf, Lombok y Supabase**

## 1. Objetivo

¡Felicidades por dominar los fundamentos! En este laboratorio avanzado, elevaremos nuestra aplicación a un nivel profesional. No solo crearemos una interfaz web, sino que también optimizaremos nuestro código y manejaremos relaciones de base de datos complejas.

**Nuevos Objetivos para este Laboratorio:**

*   **Reducir Código con Lombok**: Integraremos la librería Lombok para eliminar automáticamente el código repetitivo (getters, setters, constructores), haciendo nuestras clases de modelo mucho más limpias y legibles.
*   **Manejar Relaciones Complejas (Muchos-a-Muchos)**: Implementaremos la lógica para matricular estudiantes en cursos, gestionando la relación a través de una tabla intermedia y mostrándola en la interfaz web.
*   **Crear Consultas JPA Personalizadas**: Iremos más allá del CRUD básico. Aprenderás a escribir tus propias consultas usando JPQL dentro de los repositorios para obtener exactamente los datos que necesitas.
*   **Diseñar con Frameworks Modernos**: Usaremos **Bootstrap** por su simplicidad, pero también discutiremos **TailwindCSS** como una alternativa moderna y popular para el diseño de interfaces.

Al final, tendrás una aplicación web robusta y optimizada que no solo realiza un CRUD simple, sino que también gestiona relaciones complejas, un pilar fundamental en el desarrollo de software real.

## 2. Configuración del Proyecto y la Base de Datos

### Paso 2.1: Crear la Base de Datos en Supabase

(Este paso no cambia. Si ya tienes tu base de datos del laboratorio anterior, puedes reutilizarla).

1.  **Crea una cuenta gratuita** en [https://supabase.com/](https://supabase.com/).
2.  **Crea un Nuevo Proyecto**, guarda la **contraseña** de la base de datos en un lugar seguro.
3.  **Obtén los Datos de Conexión**: En **Settings > Database**, copia el **Host** de la cadena de conexión URI.

### Paso 2.2: Generar el Proyecto con Lombok

1.  Ve a [https://start.spring.io/](https://start.spring.io/).
2.  Configura los metadatos del proyecto (`com.cibertec`, `matricula-webapp`, etc.).
3.  En la sección de **Dependencies**, añade **CINCO** librerías:
    *   ✅ **Spring Web**
    *   ✅ **Spring Data JPA**
    *   ✅ **Thymeleaf**
    *   ✅ **PostgreSQL Driver**
    *   ✅ **Lombok** (¡La nueva adición!)
4.  Genera, descarga y abre el proyecto.

> **¡IMPORTANTE! Configuración de Lombok en tu IDE**
> Lombok funciona modificando el código durante la compilación. Para que tu IDE (VS Code, IntelliJ) no muestre errores, necesitas instalar un plugin o habilitar una opción:
> *   **Visual Studio Code**: Instala la extensión "Lombok Annotations Support for Java" de `Gabriel Basilio Brito`.
> *   **IntelliJ IDEA**: Ve a `Settings > Plugins`, busca e instala el plugin de Lombok. Luego, ve a `Settings > Build, Execution, Deployment > Compiler > Annotation Processors` y asegúrate de que la casilla "Enable annotation processing" esté marcada.

### Paso 2.3: Conectar y Optimizar la Aplicación

(Este paso es idéntico. Usa la configuración optimizada para el pool de conexiones).

Abre `src/main/resources/application.properties` y configura tu conexión a Supabase y el pool de Hikari.

```properties
# --- CONFIGURACIÓN DE LA BASE DE DATOS (SUPABASE/POSTGRESQL) ---
spring.datasource.url=jdbc:postgresql://[HOST]:5432/postgres
spring.datasource.username=postgres
spring.datasource.password=[TU-CONTRASEÑA-DE-SUPABASE]

# --- CONFIGURACIÓN DE JPA/HIBERNATE ---
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true

# --- OPTIMIZACIÓN DEL POOL DE CONEXIONES (HIKARI CP) ---
spring.datasource.hikari.maximum-pool-size=10
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.connection-timeout=30000
spring.datasource.hikari.idle-timeout=600000
```

## 3. Backend Avanzado con Lombok y Relaciones

### Paso 3.1: Definir las Entidades con Lombok

**Teoría: ¿Qué es Lombok?** Es una librería que se conecta a tu proceso de compilación y escribe automáticamente el código "boilerplate" (repetitivo). Con una simple anotación como `@Data`, Lombok genera todos los getters, setters, `toString()`, `equals()` y `hashCode()`. ¡Adiós a cientos de líneas de código!

Asegúrate de que tus entidades estén en un sub-paquete de tu aplicación principal (ej: `com.cibertec.matriculawebapp.model`).

**`Estudiante.java` (Refactorizado con Lombok)**
```java
package com.cibertec.matriculawebapp.model;

import jakarta.persistence.*;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import java.util.List;

@Data // <-- Genera Getters, Setters, toString, equals, hashCode
@NoArgsConstructor // <-- Genera un constructor sin argumentos
@AllArgsConstructor // <-- Genera un constructor con todos los argumentos
@Entity
public class Estudiante {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String nombre;
    private String apellido;

    @OneToMany(mappedBy = "estudiante")
    private List<Matricula> matriculas;
}
```

**`Curso.java` (Nueva entidad con Lombok)**
```java
package com.cibertec.matriculawebapp.model;

import jakarta.persistence.*;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import java.util.List;

@Data
@NoArgsConstructor
@AllArgsConstructor
@Entity
public class Curso {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String nombre;

    @OneToMany(mappedBy = "curso")
    private List<Matricula> matriculas;
}
```

**`Matricula.java` (La tabla intermedia con Lombok)**
```java
package com.cibertec.matriculawebapp.model;

import jakarta.persistence.*;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import java.time.LocalDate;

@Data
@NoArgsConstructor
@AllArgsConstructor
@Entity
public class Matricula {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne
    @JoinColumn(name = "estudiante_id")
    private Estudiante estudiante;

    @ManyToOne
    @JoinColumn(name = "curso_id")
    private Curso curso;
    
    private LocalDate fechaMatricula;
}
```

### Paso 3.2: Repositorios con Consultas Personalizadas

**Teoría: Consultas con `@Query`**. Aunque JpaRepository nos da métodos como `findByNombre()`, a menudo necesitamos consultas más complejas que involucren varias tablas (JOINs). La anotación `@Query` nos permite escribir consultas en JPQL (Java Persistence Query Language), que es muy similar a SQL pero opera sobre nuestras entidades (`Estudiante`, `Curso`) en lugar de tablas.

Crea los tres repositorios en el paquete `repository`.

**`EstudianteRepository.java`** (Sin cambios por ahora)
```java
package com.cibertec.matriculawebapp.repository;
//...
public interface EstudianteRepository extends JpaRepository<Estudiante, Long> {}
```

**`CursoRepository.java`**
```java
package com.cibertec.matriculawebapp.repository;
//...
public interface CursoRepository extends JpaRepository<Curso, Long> {}
```

**`MatriculaRepository.java` (Con nuestra consulta personalizada)**
```java
package com.cibertec.matriculawebapp.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;

import com.cibertec.matriculawebapp.model.Matricula;
import com.cibertec.matriculawebapp.model.Curso;
import java.util.List;

public interface MatriculaRepository extends JpaRepository<Matricula, Long> {

    /**
     * Consulta JPQL para encontrar todos los cursos en los que un estudiante específico está matriculado.
     * "SELECT m.curso": Selecciona solo el objeto Curso de la matrícula.
     * "FROM Matricula m": Opera sobre la entidad Matricula, a la que llamamos 'm'.
     * "WHERE m.estudiante.id = :idEstudiante": Filtra por el ID del estudiante contenido en la matrícula.
     * ":idEstudiante" es un parámetro que pasaremos al método.
     */
    @Query("SELECT m.curso FROM Matricula m WHERE m.estudiante.id = :idEstudiante")
    List<Curso> findCursosByEstudianteId(@Param("idEstudiante") Long idEstudiante);
}
```

### Paso 3.3: Ampliando los Controladores

Necesitamos un controlador para las matrículas y modificar el de estudiantes para que muestre los detalles.

**`EstudianteController.java` (Modificado)**
Añade un nuevo método para ver los detalles de un estudiante y sus cursos.

```java
// Dentro de EstudianteController.java

// ... (Inyecta también MatriculaRepository)
@Autowired
private MatriculaRepository matriculaRepository;

// ... (Los otros métodos del CRUD se mantienen igual) ...

// --- READ DETAILS: Muestra los detalles de un estudiante y sus cursos matriculados ---
@GetMapping("/estudiantes/detalle/{id}")
public String verDetalleEstudiante(@PathVariable Long id, Model model) {
    Estudiante estudiante = estudianteRepository.findById(id)
            .orElseThrow(() -> new IllegalArgumentException("Id de estudiante inválido:" + id));
    
    // Usamos nuestra consulta personalizada para obtener los cursos
    List<Curso> cursosMatriculados = matriculaRepository.findCursosByEstudianteId(id);
    
    model.addAttribute("estudiante", estudiante);
    model.addAttribute("cursos", cursosMatriculados);
    
    return "detalle-estudiante"; // Necesitaremos crear esta nueva vista
}
```

**`MatriculaController.java` (Nuevo Controlador)**
Crea este nuevo archivo en el paquete `controller`.

```java
package com.cibertec.matriculawebapp.controller;

import com.cibertec.matriculawebapp.model.Curso;
import com.cibertec.matriculawebapp.model.Estudiante;
import com.cibertec.matriculawebapp.model.Matricula;
import com.cibertec.matriculawebapp.repository.CursoRepository;
import com.cibertec.matriculawebapp.repository.EstudianteRepository;
import com.cibertec.matriculawebapp.repository.MatriculaRepository;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.PostMapping;
import java.time.LocalDate;

@Controller
public class MatriculaController {

    @Autowired private MatriculaRepository matriculaRepository;
    @Autowired private EstudianteRepository estudianteRepository;
    @Autowired private CursoRepository cursoRepository;

    // Muestra el formulario para realizar una nueva matrícula
    @GetMapping("/matriculas/nueva")
    public String mostrarFormularioDeMatricula(Model model) {
        // Obtenemos todos los estudiantes y cursos para los dropdowns
        model.addAttribute("estudiantes", estudianteRepository.findAll());
        model.addAttribute("cursos", cursoRepository.findAll());
        model.addAttribute("matricula", new Matricula());
        return "formulario-matricula";
    }

    // Procesa el guardado de la nueva matrícula
    @PostMapping("/matriculas")
    public String guardarMatricula(@ModelAttribute("matricula") Matricula matricula) {
        matricula.setFechaMatricula(LocalDate.now());
        matriculaRepository.save(matricula);
        // Redirigimos a los detalles del estudiante para ver la nueva matrícula
        return "redirect:/estudiantes/detalle/" + matricula.getEstudiante().getId();
    }
}
```

## 4. Interfaz de Usuario Avanzada

### Un apunte sobre TailwindCSS vs Bootstrap

Mientras que **Bootstrap** es genial para prototipar rápido con componentes pre-hechos (`<button class="btn btn-primary">`), **TailwindCSS** es un framework "utility-first". Te da clases de bajo nivel (`<button class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded">`) para que construyas diseños completamente personalizados sin escribir CSS. Para un proyecto real, Tailwind ofrece más flexibilidad, pero requiere un paso de compilación. Para este laboratorio, seguiremos con Bootstrap por su simplicidad (solo un link CDN).

### Paso 4.1: Actualizar la Lista de Estudiantes

Añade un nuevo botón para ver los detalles y otro para ir al formulario de matrícula.

**`listado-estudiantes.html` (Modificado)**
```html
<!-- ... dentro del <tbody> ... -->
<td>
    <!-- Enlace para ver detalles -->
    <a th:href="@{/estudiantes/detalle/{id}(id=${estudiante.id})}" class="btn btn-info btn-sm">Ver</a>
    <!-- Enlace para editar -->
    <a th:href="@{/estudiantes/editar/{id}(id=${estudiante.id})}" class="btn btn-warning btn-sm">Editar</a>
    <!-- Enlace para eliminar -->
    <a th:href="@{/estudiantes/eliminar/{id}(id=${estudiante.id})}" class="btn btn-danger btn-sm">Eliminar</a>
</td>

<!-- ... debajo de la tabla ... -->
<a th:href="@{/matriculas/nueva}" class="btn btn-success mt-3">Nueva Matrícula</a>
```
*También puedes añadir un botón para crear cursos de la misma manera.*

### Paso 4.2: Crear la Vista de Detalle del Estudiante

Crea el archivo `src/main/resources/templates/detalle-estudiante.html`.

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Detalle del Estudiante</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <div class="container mt-4">
        <div class="card">
            <div class="card-header">
                <h3>Detalles de <span th:text="${estudiante.nombre} + ' ' + ${estudiante.apellido}"></span></h3>
            </div>
            <div class="card-body">
                <h4>Cursos Matriculados</h4>
                <div th:if="${cursos.isEmpty()}">
                    <p class="text-muted">Este estudiante no está matriculado en ningún curso.</p>
                </div>
                <ul class="list-group" th:unless="${cursos.isEmpty()}">
                    <li class="list-group-item" th:each="curso : ${cursos}" th:text="${curso.nombre}"></li>
                </ul>
            </div>
            <div class="card-footer">
                <a th:href="@{/estudiantes}" class="btn btn-secondary">Volver a la Lista</a>
                <a th:href="@{/matriculas/nueva}" class="btn btn-primary">Matricular en Nuevo Curso</a>
            </div>
        </div>
    </div>
</body>
</html>
```

### Paso 4.3: Crear el Formulario de Matrícula

Crea el archivo `src/main/resources/templates/formulario-matricula.html`.

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Nueva Matrícula</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
<div class="container mt-4">
    <div class="card">
        <div class="card-header"><h3>Registrar Matrícula</h3></div>
        <div class="card-body">
            <form th:action="@{/matriculas}" th:object="${matricula}" method="POST">
                
                <div class="mb-3">
                    <label for="estudiante" class="form-label">Seleccionar Estudiante:</label>
                    <select class="form-select" id="estudiante" th:field="*{estudiante}" required>
                        <option value="">-- Seleccione un Estudiante --</option>
                        <option th:each="est : ${estudiantes}" th:value="${est.id}" th:text="${est.nombre} + ' ' + ${est.apellido}"></option>
                    </select>
                </div>

                <div class="mb-3">
                    <label for="curso" class="form-label">Seleccionar Curso:</label>
                    <select class="form-select" id="curso" th:field="*{curso}" required>
                        <option value="">-- Seleccione un Curso --</option>
                        <option th:each="cur : ${cursos}" th:value="${cur.id}" th:text="${cur.nombre}"></option>
                    </select>
                </div>

                <button type="submit" class="btn btn-success">Matricular</button>
                <a th:href="@{/estudiantes}" class="btn btn-secondary">Cancelar</a>
            </form>
        </div>
    </div>
</div>
</body>
</html>
```

## 5. Ejecución y Prueba

1.  **Inicia la aplicación**.
2.  **Crea algunos Estudiantes y Cursos**: Ve a `/estudiantes/nuevo` para crear estudiantes. (Para los cursos, puedes insertarlos directamente en Supabase o crear un CRUD simple para ellos como desafío).
3.  **Realiza una Matrícula**: Ve a `/matriculas/nueva`, selecciona un estudiante y un curso, y guarda.
4.  **Verifica**: Serás redirigido a la página de detalles del estudiante. ¡Deberías ver el curso recién matriculado en la lista!

## 6. Conclusión y Próximos Pasos

¡Felicidades! Has construido una aplicación web que refleja muchos de los desafíos del mundo real. Has aprendido a:
*   **Optimizar tu código** radicalmente usando **Lombok**.
*   **Modelar y gestionar relaciones Muchos-a-Muchos** a través de una tabla intermedia.
*   Escribir **consultas JPQL personalizadas** con `@Query` para obtener datos relacionales.
*   **Construir una interfaz de usuario** que maneja estas relaciones complejas, mostrando detalles y permitiendo nuevas asociaciones.

**Desafío Final (Próximos Pasos):**
1.  **CRUD para Cursos**: Crea las vistas y métodos en el controlador para poder añadir, editar y eliminar cursos desde la interfaz web.
2.  **Evitar Matrículas Duplicadas**: Añade lógica en tu `MatriculaController` para comprobar si un estudiante ya está matriculado en un curso antes de guardar una nueva matrícula.
3.  **Página de Detalles del Curso**: Crea una vista que muestre un curso y la lista de todos los estudiantes matriculados en él. ¡Necesitarás una nueva consulta personalizada
