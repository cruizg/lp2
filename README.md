
---

# Guía de Laboratorio: Introducción a JPA con Hibernate y H2

## 1. Objetivo

Bienvenido al laboratorio de JPA. El objetivo de hoy es aprender a conectar una aplicación Java a una base de datos sin escribir SQL manualmente para las operaciones básicas. Usaremos el **Java Persistence API (JPA)**, que es un estándar de Java para mapear objetos a bases de datos relacionales.

**¿Por qué JPA?** Imagina que tienes un objeto `Estudiante` en Java. Sin JPA, para guardarlo en una base de datos, tendrías que escribir una sentencia `INSERT INTO ...`. Para leerlo, un `SELECT ...`. Para actualizarlo, un `UPDATE ...`. JPA hace todo ese trabajo por ti. Tú solo trabajas con tus objetos Java, y JPA (con su implementación, en nuestro caso **Hibernate**) se encarga de "traducir" esas acciones a SQL.

Usaremos **H2 Database**, una base de datos ligera y rápida, ideal para desarrollo y pruebas.

## 2. Configuración del Proyecto

### Paso 2.1: Crear el Proyecto con Maven

Maven es una herramienta que nos ayuda a construir nuestro proyecto y a gestionar las "piezas" de software que necesitamos (dependencias).

Abre una terminal y ejecuta el siguiente comando para crear la estructura del proyecto:

```bash
mvn archetype:generate -DgroupId=com.cibertec -DartifactId=matricula -DarchetypeArtifactId=maven-archetype-quickstart -DarchetypeVersion=1.4 -DinteractiveMode=false
```

Esto crea una carpeta llamada `matricula` con la estructura estándar de un proyecto Java.

### Paso 2.2: Añadir Dependencias en `pom.xml`

El archivo `pom.xml` es el corazón de nuestro proyecto Maven. Aquí le decimos qué librerías externas (dependencias) necesita nuestra aplicación.

Abre el archivo `pom.xml` y reemplaza su contenido con el siguiente código:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" 
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.cibertec</groupId>
    <artifactId>matricula</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>

    <dependencies>
        <!--
            Teoría: ¿Qué son estas dependencias?
            1. hibernate-core: Es la implementación de JPA que usaremos. Es el "motor" que
               convierte nuestros objetos Java en sentencias SQL.
            2. h2: Es nuestra base de datos. Es súper ligera, escrita en Java y perfecta
               para laboratorios como este.
        -->
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-core</artifactId>
            <version>5.6.14.Final</version>
        </dependency>
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <version>2.1.214</version>
        </dependency>
    </dependencies>
    
    <!-- El resto de la configuración de build se puede mantener como está -->
</project>
```

### Paso 2.3: Crear el Archivo de Configuración `persistence.xml`

**Teoría:** El archivo `persistence.xml` es el cerebro de JPA. Aquí le damos las instrucciones clave:
*   **Qué base de datos usar:** Le daremos la URL, usuario y contraseña de nuestra BD H2.
*   **Qué clases son Entidades:** Listaremos las clases Java que queremos que se conviertan en tablas.
*   **Cómo debe comportarse Hibernate:** Le diremos que muestre el SQL que genera y que cree las tablas por nosotros.

Crea la carpeta `src/main/resources/META-INF` y, dentro de ella, el archivo `persistence.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.2"
             xmlns="http://xmlns.jcp.org/xml/ns/persistence"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">

    <persistence-unit name="MatriculaPU" transaction-type="RESOURCE_LOCAL">
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
        
        <!-- Aquí registraremos nuestras clases que serán tablas -->
        
        <properties>
            <!-- Configuración de la conexión a la base de datos H2 -->
            <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
            <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/db/matriculaLabDB"/>
            <property name="javax.persistence.jdbc.user" value="sa"/>
            <property name="javax.persistence.jdbc.password" value=""/>

            <!-- Propiedades de Hibernate -->
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
            <!-- 
                Teoría: hibernate.hbm2ddl.auto
                'create-drop' es perfecto para desarrollo. Significa:
                1. Al iniciar la aplicación, BORRA las tablas viejas (si existen).
                2. CREA las tablas nuevas basadas en nuestras entidades.
                3. Al cerrar la aplicación, BORRA todas las tablas.
                Así, cada ejecución empieza con una base de datos limpia.
            -->
            <property name="hibernate.hbm2ddl.auto" value="create-drop"/>
        </properties>
    </persistence-unit>
</persistence>
```

## 3. Nuestra Primera Entidad y el CRUD

Vamos a empezar con lo más simple: la entidad `Estudiante`. Realizaremos un **CRUD** completo (Crear, Leer, Actualizar y Borrar).

### Paso 3.1: Crear la Entidad `Estudiante`

**Teoría:** Una **Entidad** es una clase Java normal que ha sido "marcada" con la anotación `@Entity`. Esto le dice a JPA: "¡Oye, quiero que esta clase se convierta en una tabla en la base de datos!".
*   `@Id`: Marca cuál de sus atributos será la clave primaria (primary key).
*   `@GeneratedValue`: Le dice a la base de datos que genere el valor del ID automáticamente (autoincremental).
*   `@Column`: Permite configurar detalles de la columna, como que no puede ser nula (`nullable=false`).

Crea el paquete `com.cibertec.matricula.model` y dentro, la clase `Estudiante.java`.

```java
package com.cibertec.matricula.model;

import javax.persistence.*;

@Entity // Marca esta clase como una tabla de base de datos
public class Estudiante {

    @Id // Marca este campo como la clave primaria
    @GeneratedValue(strategy = GenerationType.IDENTITY) // Autoincremental
    private Long id;

    @Column(nullable = false, length = 50)
    private String nombre;

    @Column(nullable = false, length = 50)
    private String apellido;

    // Constructores, Getters y Setters (necesarios para que Hibernate funcione)
    public Estudiante() {}

    public Estudiante(String nombre, String apellido) {
        this.nombre = nombre;
        this.apellido = apellido;
    }

    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getNombre() { return nombre; }
    public void setNombre(String nombre) { this.nombre = nombre; }
    public String getApellido() { return apellido; }
    public void setApellido(String apellido) { this.apellido = apellido; }

    @Override
    public String toString() {
        return "Estudiante{" + "id=" + id + ", nombre='" + nombre + "', apellido='" + apellido + "'}";
    }
}
```

**Importante:** No olvides añadir la entidad a `persistence.xml`:

```xml
<!-- En persistence.xml, dentro de <persistence-unit> -->
<class>com.cibertec.matricula.model.Estudiante</class>
```

### Paso 3.2: Clase de Utilidad para JPA

**Teoría:** Para hablar con la base de datos, JPA nos da un `EntityManager`. Para crear un `EntityManager`, necesitamos un `EntityManagerFactory`. Crear un `EntityManagerFactory` es un proceso "caro" (consume tiempo y recursos), así que lo ideal es crear **uno solo** para toda la aplicación. Esta clase de utilidad se encargará de eso.

Crea el paquete `com.cibertec.matricula.util` y la clase `JPAUtil.java`.

```java
package com.cibertec.matricula.util;

import javax.persistence.EntityManagerFactory;
import javax.persistence.Persistence;

public class JPAUtil {
    private static final String PERSISTENCE_UNIT_NAME = "MatriculaPU";
    private static EntityManagerFactory factory;

    public static EntityManagerFactory getEntityManagerFactory() {
        if (factory == null) {
            factory = Persistence.createEntityManagerFactory(PERSISTENCE_UNIT_NAME);
        }
        return factory;
    }

    public static void shutdown() {
        if (factory != null) {
            factory.close();
        }
    }
}
```

### Paso 3.3: La Clase Principal y el Servidor H2

Vamos a modificar nuestra clase `App.java` para que haga todo:
1.  Inicie el servidor de la base de datos H2.
2.  Nos dé tiempo para ver la base de datos con una pausa.
3.  Ejecute las operaciones CRUD.

Reemplaza el contenido de `App.java` en el paquete `com.cibertec.matricula` con lo siguiente:

```java
package com.cibertec.matricula;

import com.cibertec.matricula.model.Estudiante;
import com.cibertec.matricula.util.JPAUtil;
import org.h2.tools.Server;

import javax.persistence.EntityManager;
import javax.persistence.EntityTransaction;
import java.sql.SQLException;
import java.util.Scanner;

public class App {

    private static Server h2Server;
    private static Server iniciarServidorH2() {
        try {
            Server webServer = Server.createWebServer("-web", "-webAllowOthers", "-webPort", "8082").start();
            System.out.println(" H2 Web Console disponible en: http://localhost:8082");
            System.out.println("Conéctate con:");
            System.out.println("   JDBC URL: jdbc:h2:mem:matriculaDB");
            System.out.println("   Usuario: sa");
            System.out.println("   Contraseña: (dejar vacío)");
            return webServer;
        } catch (java.sql.SQLException e) {
            System.err.println("Error al iniciar el servidor H2");
            e.printStackTrace();
            return null;
        }
    }

    private static void pausar(Scanner scanner) {
        System.out.print("Presiona ENTER para continuar...");
        scanner.nextLine();
    }
    public static void main(String[] args) {
        try {
            // 1. Iniciar el servidor H2 para poder acceder a la consola web
            Server webServer = iniciarServidorH2();
            Scanner scanner = new Scanner(System.in);

            // Obtenemos un EntityManager
            EntityManager em = JPAUtil.getEntityManagerFactory().createEntityManager();
            
            // --- CRUD ---
            // Teoría: Una Transacción es una operación "todo o nada". O se ejecutan
            // todas las operaciones dentro de ella, o no se ejecuta ninguna.
            // Es esencial para mantener la integridad de los datos.
            EntityTransaction tx = em.getTransaction();
            
            // CREATE
            tx.begin();
            System.out.println("\n--- Creando un nuevo estudiante ---");
            Estudiante nuevoEstudiante = new Estudiante("Carlos", "Santana");
            em.persist(nuevoEstudiante); // persist() es como un "INSERT"
            tx.commit();
            System.out.println("Estudiante creado con ID: " + nuevoEstudiante.getId());

            // PAUSA PARA VER LA BD
            pausar();

            // READ
            System.out.println("\n--- Leyendo el estudiante con ID " + nuevoEstudiante.getId() + " ---");
            Estudiante estudianteLeido = em.find(Estudiante.class, nuevoEstudiante.getId()); // find() es como "SELECT ... WHERE ID="
            System.out.println("Estudiante encontrado: " + estudianteLeido);

            // UPDATE
            tx.begin();
            System.out.println("\n--- Actualizando el apellido del estudiante ---");
            estudianteLeido.setApellido("Vives");
            em.merge(estudianteLeido); // merge() es como un "UPDATE"
            tx.commit();
            System.out.println("Estudiante actualizado: " + estudianteLeido);

            // PAUSA PARA VER LA BD
            pausar();

            // DELETE
            tx.begin();
            System.out.println("\n--- Borrando el estudiante con ID " + estudianteLeido.getId() + " ---");
            // Para borrar, el objeto debe estar "gestionado" por el EntityManager,
            // por eso lo volvemos a "adjuntar" con merge() antes de borrar.
            em.remove(em.merge(estudianteLeido)); // remove() es como un "DELETE"
            tx.commit();
            System.out.println("Estudiante borrado.");

            // Verificamos que ya no existe
            Estudiante estudianteBorrado = em.find(Estudiante.class, estudianteLeido.getId());
            System.out.println("¿Estudiante existe? " + (estudianteBorrado == null ? "No" : "Sí"));
            
            em.close();

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // Cierra la conexión a la BD y el servidor H2
            JPAUtil.shutdown();
            stopH2Server();
            System.out.println("\n>>> APLICACIÓN FINALIZADA <<<");
        }
    }

    private static void startH2Server() throws SQLException {
        // Inicia el servidor TCP (para la aplicación) y el servidor Web (para la consola)
        h2Server = Server.createTcpServer("-tcpAllowOthers").start();
        Server.createWebServer("-webAllowOthers").start();
    }

    private static void stopH2Server() {
        if (h2Server != null) {
            h2Server.stop();
        }
    }
    
    
}
```

### ¡A Ejecutar!

1.  Ejecuta la clase `App.java`.
2.  Verás los mensajes en la consola pidiéndote que abras el navegador.
3.  Abre `http://localhost:8082`.
4.  Conéctate con los datos proporcionados.
5.  En la consola H2, escribe `SELECT * FROM ESTUDIANTE;` y presiona `Run`. Verás el registro de "Carlos Santana".
6.  Vuelve a tu IDE y presiona `Enter` en la consola. La aplicación continuará.
7.  Cuando llegue la segunda pausa, refresca la consola H2 y ejecuta de nuevo `SELECT * FROM ESTUDIANTE;`. Ahora verás "Carlos Vives".
8.  Presiona `Enter` de nuevo. La aplicación terminará y borrará el registro. Si revisas la consola H2 una última vez, la tabla estará vacía.

## 4. Agregando Relaciones

Ahora, compliquemos un poco el sistema. Un estudiante puede matricularse en muchos cursos, y un curso puede tener muchos estudiantes. Esto es una relación **Muchos-a-Muchos**. La forma correcta de modelar esto en una base de datos es con una tabla intermedia. En JPA, esta tabla intermedia también será una Entidad.

Nuestro modelo será:
*   `Estudiante` 1 <--- * `Matricula`
*   `Curso` 1 <--- * `Matricula`

### Paso 4.1: Crear Entidad `Curso` y `Matricula`

Crea las siguientes clases en el paquete `com.cibertec.matricula.model`.

**`Curso.java`**
```java
package com.cibertec.matricula.model;

import javax.persistence.*;
import java.util.ArrayList;
import java.util.List;

@Entity
public class Curso {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String nombre;
    
    // Teoría: @OneToMany
    // Un Curso tiene muchas Matrículas.
    // mappedBy = "curso": Le dice a JPA "No crees una columna para esta relación en la tabla CURSO.
    // La relación ya está definida por el campo 'curso' en la clase Matricula".
    // cascade = CascadeType.ALL: Si guardo/actualizo/borro un Curso, haz lo mismo con sus matrículas asociadas.
    @OneToMany(mappedBy = "curso", cascade = CascadeType.ALL)
    private List<Matricula> matriculas = new ArrayList<>();

    public Curso() {}

    public Curso(String nombre) {
        this.nombre = nombre;
    }

    // Getters y Setters...
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getNombre() { return nombre; }
    public void setNombre(String nombre) { this.nombre = nombre; }
    public List<Matricula> getMatriculas() { return matriculas; }
    public void setMatriculas(List<Matricula> matriculas) { this.matriculas = matriculas; }

    @Override
    public String toString() {
        return "Curso{" + "id=" + id + ", nombre='" + nombre + "'}";
    }
}
```

**`Matricula.java`**
```java
package com.cibertec.matricula.model;

import javax.persistence.*;
import java.time.LocalDate;

@Entity
public class Matricula {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    // Teoría: @ManyToOne
    // Muchas matrículas pertenecen a UN solo estudiante. Esta anotación crea la
    // clave foránea (foreign key) en la tabla MATRICULA.
    // @JoinColumn: Especifica el nombre de esa columna de clave foránea.
    @ManyToOne
    @JoinColumn(name = "estudiante_id")
    private Estudiante estudiante;

    @ManyToOne
    @JoinColumn(name = "curso_id")
    private Curso curso;
    
    private LocalDate fechaMatricula;

    public Matricula() {}

    public Matricula(Estudiante estudiante, Curso curso) {
        this.estudiante = estudiante;
        this.curso = curso;
        this.fechaMatricula = LocalDate.now();
    }

    // Getters y Setters...
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public Estudiante getEstudiante() { return estudiante; }
    public void setEstudiante(Estudiante estudiante) { this.estudiante = estudiante; }
    public Curso getCurso() { return curso; }
    public void setCurso(Curso curso) { this.curso = curso; }
    public LocalDate getFechaMatricula() { return fechaMatricula; }
    public void setFechaMatricula(LocalDate fechaMatricula) { this.fechaMatricula = fechaMatricula; }
}
```

### Paso 4.2: Actualizar `Estudiante` y `persistence.xml`

Añade la relación `OneToMany` a la clase `Estudiante.java`.

```java
// Dentro de la clase Estudiante.java

// ... otros campos y métodos ...

@OneToMany(mappedBy = "estudiante", cascade = CascadeType.ALL)
private java.util.List<Matricula> matriculas = new java.util.ArrayList<>();

// ... constructores ...

// Getter y Setter para la nueva lista
public java.util.List<Matricula> getMatriculas() { return matriculas; }
public void setMatriculas(java.util.List<Matricula> matriculas) { this.matriculas = matriculas; }

// ... toString ...
```

Ahora, registra las nuevas entidades en `persistence.xml`:

```xml
<!-- En persistence.xml, dentro de <persistence-unit> -->
<class>com.cibertec.matricula.model.Estudiante</class>
<class>com.cibertec.matricula.model.Curso</class>
<class>com.cibertec.matricula.model.Matricula</class>
```

### Paso 4.3: Probar las Relaciones

Reemplaza el código dentro del bloque `try` del `main` en `App.java` para probar las nuevas relaciones.

```java
// Reemplaza el contenido del bloque try en App.java
try {
    System.out.println(">>> SERVIDOR H2 INICIADO. Revisa la consola en http://localhost:8082 <<<");
    
    EntityManager em = JPAUtil.getEntityManagerFactory().createEntityManager();
    EntityTransaction tx = em.getTransaction();
    
    tx.begin();
    
    // 1. Creamos entidades
    Estudiante estudianteAna = new Estudiante("Ana", "Torres");
    Curso cursoCalculo = new Curso("Cálculo I");
    Curso cursoFisica = new Curso("Física I");

    // Las guardamos para que tengan un ID
    em.persist(estudianteAna);
    em.persist(cursoCalculo);
    em.persist(cursoFisica);

    // 2. Creamos las matrículas (la relación)
    Matricula matricula1 = new Matricula(estudianteAna, cursoCalculo);
    Matricula matricula2 = new Matricula(estudianteAna, cursoFisica);
    
    em.persist(matricula1);
    em.persist(matricula2);
    
    tx.commit();
    System.out.println("Estudiante Ana matriculada en Cálculo y Física.");

    // PAUSA PARA VER LA BD
    pausar();
    
    // 3. Consulta con JPQL (Java Persistence Query Language)
    // Teoría: JPQL es como SQL, pero en lugar de operar sobre tablas, opera sobre
    // nuestras Entidades (objetos Java). Es más seguro y orientado a objetos.
    // "SELECT c FROM Curso c JOIN c.matriculas m WHERE m.estudiante.id = :estId"
    // Traducción: "Selecciona los Cursos 'c' donde la matrícula 'm' de ese curso
    // corresponda al ID de estudiante que te voy a pasar".
    
    System.out.println("\n--- Buscando los cursos de " + estudianteAna.getNombre() + " usando JPQL ---");
    String jpql = "SELECT m.curso FROM Matricula m WHERE m.estudiante.id = :idEstudiante";
    
    java.util.List<Curso> cursosDeAna = em.createQuery(jpql, Curso.class)
        .setParameter("idEstudiante", estudianteAna.getId())
        .getResultList();

    System.out.println("Cursos encontrados:");
    cursosDeAna.forEach(curso -> System.out.println("- " + curso.getNombre()));

    em.close();

} catch (Exception e) {
    e.printStackTrace();
} finally {
    JPAUtil.shutdown();
    stopH2Server();
    System.out.println("\n>>> APLICACIÓN FINALIZADA <<<");
}
```

### ¡Ejecuta y Explora!

1.  Ejecuta la clase `App.java` de nuevo.
2.  Ve a la consola H2. Ahora verás 3 tablas: `ESTUDIANTE`, `CURSO` y `MATRICULA`.
3.  Explora sus contenidos:
    *   `SELECT * FROM ESTUDIANTE;`
    *   `SELECT * FROM CURSO;`
    *   `SELECT * FROM MATRICULA;`
4.  Fíjate cómo la tabla `MATRICULA` tiene las columnas `ESTUDIANTE_ID` y `CURSO_ID`, que son las claves foráneas que conectan todo.
5.  Presiona `Enter` en tu IDE para que la consulta JPQL se ejecute y veas el resultado.

## 5. Conclusión

¡Felicidades! En este laboratorio has logrado:
*   Configurar un proyecto Java con Maven para usar JPA.
*   Mapear una clase Java a una tabla de base de datos usando `@Entity`.
*   Realizar operaciones **CRUD** (Crear, Leer, Actualizar, Borrar) usando el `EntityManager`.
*   Iniciar un servidor de base de datos H2 y **pausar tu programa** para inspeccionar los datos en tiempo real.
*   Modelar una relación **Muchos-a-Muchos** a través de una tabla intermedia.
*   Escribir tu primera consulta en **JPQL** para obtener datos de forma relacional.

Este es el fundamento para construir aplicaciones Java robustas que interactúen con bases de datos de una manera moderna y eficiente.
