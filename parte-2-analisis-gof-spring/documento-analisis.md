# Análisis de Patrones de Diseño GoF en Spring Framework

**Nombre:** _[Karen Yelitza Chiquillo Quintero]_
**Código:** _[02240131060]_
**Curso:** Patrones de Diseño de Software — Sexto Semestre
**Unidad:** Unidad 1 — Fundamentos de Patrones de Diseño y Buenas Prácticas
**Fecha:** _[18/08/2026]_

---

## 1. Introducción

El valor práctico de los patrones de diseño orientados a objetos se comprende mejor cuando se examina su funcionamiento dentro de proyectos de software a escala industrial, donde las decisiones arquitectónicas responden a requerimientos reales de desacoplamiento, mantenibilidad y extensibilidad. Aunque el catálogo clásico formulado por Gamma et al. (1994) suele estudiarse a partir de abstracciones teóricas, su propósito fundamental radica en resolver problemas estructurales recurrentes. Con base en esta premisa, este trabajo analiza el código fuente oficial de Spring Framework con el fin de evidenciar cómo las soluciones del catálogo Gang of Four (GoF) sustentan la infraestructura de un framework ampliamente adoptado en entornos de producción.

Spring Framework representa un escenario de análisis idóneo, ya que su contenedor de Inversión de Control (IoC) y su arquitectura modular operan mediante la integración continua de patrones creacionales, estructurales y de comportamiento. En este documento se abordan tres implementaciones concretas: **Factory Method** (Creacional), localizado en la interfaz `BeanFactory` del módulo `spring-beans` para abstraer la instanciación y el ciclo de vida de los objetos; **Facade** (Estructural), a través de `JdbcTemplate` en `spring-jdbc`, el cual simplifica la interacción y la gestión de recursos de la API JDBC nativa; y **Observer** (Comportamiento), mediante el contrato `ApplicationListener` en `spring-context`, que permite la suscripción y el despacho desacoplado de eventos. Cada patrón se evalúa a partir de evidencia directa de código, explicando la necesidad arquitectónica que resuelve y su vínculo directo con los principios SOLID (DIP, SRP y OCP), bajo el marco conceptual de la Unidad 1 (Universidad UDES, 2026).

---

## 2. Análisis de Patrón 1: Factory Method

### 2.1 Patrón y categoría

El análisis inicia con **Factory Method**, clasificado dentro de la categoría de patrones **creacionales** en el catálogo clásico de GoF. La intención central de este patrón radica en abstraer la creación de objetos mediante la definición de una interfaz en una superclase, delegando a las implementaciones concretas la responsabilidad de decidir qué clase específica instanciar y bajo qué mecanismo construirla (Gamma et al., 1994). Como destaca Refactoring Guru (s.f.), esta indirección permite que el código cliente opere exclusivamente a través de contratos abstractos, eliminando la dependencia directa hacia los constructores y desacoplando la lógica de negocio del proceso de instanciación.

### 2.2 Ubicación en Spring Framework

El patrón se manifiesta formalmente en la interfaz `org.springframework.beans.factory.BeanFactory`, ubicada en el módulo `spring-beans`, dentro del repositorio oficial de Spring Framework (Spring Projects, s.f.). `BeanFactory` constituye la raíz de la jerarquía del contenedor de Inversión de Control (IoC) de Spring y actúa como el rol de **Creator** abstracto del patrón: define el contrato `getBean(...)` sin especificar cómo el bean solicitado es efectivamente instanciado, tarea que delega a implementaciones concretas del contenedor como `DefaultListableBeanFactory`.

### 2.3 Problema que resuelve

En una arquitectura orientada a objetos tradicional, el uso directo del operador `new` para instanciar dependencias genera un acoplamiento rígido entre el código cliente y las clases concretas, dispersa la configuración de los objetos a lo largo de la base de código y dificulta sustituir una implementación por otra sin modificar cada punto donde se realiza la instanciación. Spring Boot resuelve este problema centralizando la creación de objetos detrás de un método de fábrica polimórfico: el cliente solicita un bean por nombre o por tipo, y es el contenedor —no el cliente— quien decide si debe crear una nueva instancia, devolver una instancia compartida ya existente (scope Singleton), o aplicar cualquier otra estrategia de ciclo de vida configurada (scope Prototype, por ejemplo). Esta indirección es la que permite que Spring Boot funcione como un contenedor de inyección de dependencias completo, en lugar de una simple biblioteca de utilidades.

### 2.4 Evidencia de código

El siguiente fragmento, extraído del archivo `BeanFactory.java` del repositorio oficial (Spring Projects, s.f.), corresponde a la firma del método de fábrica principal, que constituye el Factory Method clásico de GoF adaptado a Java:

```java
/**
 * Return an instance, which may be shared or independent, of the specified bean.
 * Translates aliases back to the corresponding canonical bean name.
 * Will ask the parent factory if the bean cannot be found in this factory instance.
 */
Object getBean(String name) throws BeansException;
```

Una segunda sobrecarga, ubicada en el mismo archivo, incorpora seguridad de tipos mediante genéricos, evitando que el cliente deba realizar conversiones explícitas (*castings*) sobre el objeto retornado:

```java
/**
 * Behaves the same as getBean(String), but provides a measure of type
 * safety by throwing a BeanNotOfRequiredTypeException if the bean is not
 * of the required type.
 */
<T> T getBean(String name, Class<T> requiredType) throws BeansException;
```

Ambos métodos comparten la característica esencial del patrón: el cliente invoca una operación de fábrica y recibe un objeto ya construido, sin conocer si dicho objeto proviene de una instancia reutilizada o de una recién creada, ni qué mecanismo interno de resolución de dependencias fue aplicado por el contenedor.

### 2.5 Principio SOLID asociado

`BeanFactory` refuerza principalmente el **Dependency Inversion Principle (DIP)**: los módulos de alto nivel de una aplicación Spring Boot, los servicios de negocio no dependen de clases concretas ni invocan constructores directamente, sino que dependen de la abstracción `BeanFactory` (o, más comúnmente, de la inyección automática de dependencias que el contenedor construye sobre ella). De forma secundaria, el patrón también respalda el **Single Responsibility Principle (SRP)**, dado que la responsabilidad de instanciar, configurar e inyectar dependencias queda delegada por completo al contenedor, liberando a las clases del dominio de tareas de infraestructura que no les corresponden. Esta doble conexión coincide con lo señalado en la Guía de Contenido de la Unidad 1, que identifica a Factory Method como uno de los patrones que implementan directamente el principio DIP (Universidad UDES, 2026).

---

## 3. Análisis de Patrón 2: Facade

### 3.1 Patrón y categoría

El segundo patrón identificado es **Facade**, perteneciente a la categoría **Estructural** del catálogo GoF. Facade proporciona una interfaz unificada y simplificada sobre un conjunto de interfaces de un subsistema, ocultando su complejidad interna y facilitando su uso desde el código cliente (Gamma et al., 1994). Refactoring Guru (s.f.) describe este patrón como una solución que "provee una interfaz simplificada a una biblioteca, un framework o cualquier otro conjunto complejo de clases", sin eliminar la funcionalidad subyacente, sino encapsulándola detrás de operaciones de alto nivel.

### 3.2 Ubicación en Spring Framework

Dentro de Spring Framework, esta solución estructural se localiza de forma concreta en la clase `org.springframework.jdbc.core.JdbcTemplate`, perteneciente al módulo `spring-jdbc` (Spring Projects, s.f.). Esta clase extiende `JdbcAccessor` e implementa la interfaz `org.springframework.jdbc.core.JdbcOperations`, la cual define el contrato abstracto de la fachada. A través de esta estructura, `JdbcTemplate` actúa como un punto de acceso unificado que encapsula la interacción con el subsistema nativo de Java Database Connectivity (`java.sql`), consolidándose en el ecosistema del framework como el mecanismo estándar para gestionar operaciones sobre bases de datos relacionales sin exponer los detalles de bajo nivel al código cliente (Spring, s.f.).

### 3.3 Problema que resuelve

El API nativo de JDBC exige al desarrollador gestionar manualmente un flujo de trabajo repetitivo y propenso a errores: abrir y cerrar la conexión, crear y tipar el `PreparedStatement`, iterar el `ResultSet` fila por fila, y capturar la excepción comprobada `SQLException` en cada punto de fallo. Omitir el cierre de recursos en un bloque `finally` provoca fugas de memoria y agotamiento del *pool* de conexiones, un defecto común en aplicaciones legacy construidas directamente sobre JDBC. `JdbcTemplate` resuelve este problema absorbiendo toda esa complejidad operativa detrás de métodos declarativos como `query`, `update` y `execute`: el desarrollador únicamente define la sentencia SQL y, opcionalmente, cómo transformar cada fila del resultado en un objeto de dominio, mientras la fachada se encarga de adquirir la conexión, ejecutar la sentencia, liberar los recursos y traducir las excepciones comprobadas de JDBC a excepciones no comprobadas del ecosistema Spring.

### 3.4 Evidencia de código

El siguiente fragmento, tomado del repositorio oficial de Spring Framework (Spring Projects, s.f.), corresponde al método central donde la fachada absorbe la gestión del ciclo de vida de la conexión JDBC:

```java
private <T> T execute(StatementCallback<T> action, boolean closeResources) throws DataAccessException {
    Connection con = DataSourceUtils.getConnection(obtainDataSource());
    Statement stmt = null;
    try {
        stmt = con.createStatement();
        applyStatementSettings(stmt);
        T result = action.doInStatement(stmt);
        handleWarnings(stmt);
        return result;
    }
    catch (SQLException ex) {
        // traduce la excepción comprobada de JDBC a una excepción de Spring
        throw translateException("StatementCallback", getSql(action), ex);
    }
    finally {
        if (closeResources) {
            JdbcUtils.closeStatement(stmt);
            DataSourceUtils.releaseConnection(con, getDataSource());
        }
    }
}
```

Sobre esta base se construye el método `query`, que representa la abstracción de más alto nivel expuesta al desarrollador, quien solo necesita proporcionar la sentencia SQL y un `RowMapper`:

```java
public <T> List<T> query(String sql, RowMapper<T> rowMapper) throws DataAccessException {
    return result(query(sql, new RowMapperResultSetExtractor<>(rowMapper, 0, this.maxRows)));
}
```

La comparación entre ambos fragmentos evidencia cómo la fachada oculta progresivamente la gestión de conexiones, sentencias y cursores, dejando al cliente una operación de una sola línea.

### 3.5 Principio SOLID asociado

El diseño de `JdbcTemplate` como fachada se articula directamente con tres principios fundamentales de SOLID. En primer lugar, satisface el **Single Responsibility Principle (SRP)** al separar de forma tajante la gestión de infraestructura de bajo nivel —control de conexiones, iteración de cursores, cierre de recursos en bloques de contención y traducción de errores— de la lógica de persistencia y del dominio de la aplicación.

En segundo lugar, la clase promueve el **Dependency Inversion Principle (DIP)**, ya que los componentes de la capa de datos (como DAOs o repositorios) interactúan a través del contrato abstracto `JdbcOperations`, evitando acoplarse a implementaciones concretas de controladores de base de datos. Por último, la arquitectura refuerza el **Open/Closed Principle (OCP)** mediante el desacoplamiento de algoritmos de transformación a través de interfaces funcionales como `RowMapper` y `ResultSetExtractor`; estos mecanismos permiten extender las estrategias de mapeo de resultados sin requerir modificaciones en el código fuente de `JdbcTemplate`. Esta articulación estructural coincide con el marco conceptual desarrollado en la Guía de Contenido, donde los patrones estructurales se analizan como habilitadores esenciales de la cohesión y la extensibilidad en frameworks empresariales (Universidad UDES, 2026).

---

## 4. Análisis de Patrón 3: Observer

### 4.1 Patrón y categoría

El análisis concluye con el patrón **Observer**, clasificado dentro de la categoría de **comportamiento** en el catálogo GoF. Su propósito fundamental radica en estructurar un modelo de comunicación reactivo bajo una relación de dependencia uno-a-muchos, permitiendo que un objeto emisor (el sujeto) propague notificaciones de manera automática a múltiples receptores (los observadores) cuando se produce un cambio de estado o se dispara un evento determinado (Gamma et al., 1994). De acuerdo con Refactoring Guru (s.f.), este esquema opera como un mecanismo de publicación y suscripción donde el sujeto delega el procesamiento de efectos secundarios sin acoplarse a la identidad, cantidad o lógica interna de las clases suscritas.

### 4.2 Ubicación en Spring Framework

El patrón se identifica en la interfaz funcional `org.springframework.context.ApplicationListener`, junto con la clase `org.springframework.context.event.SimpleApplicationEventMulticaster`, ambas del módulo `spring-context` (Spring Projects, s.f.). `ApplicationListener<E>` desempeña el rol de **Observer** del patrón, mientras que `SimpleApplicationEventMulticaster` desempeña el rol de **Subject** concreto, encargado de mantener el registro de observadores y de despachar la notificación correspondiente.

### 4.3 Problema que resuelve

En sistemas empresariales es habitual que un único suceso de negocio, por ejemplo, el registro de una orden o la autenticación de un usuario deba desencadenar múltiples acciones secundarias independientes entre sí: enviar un correo de confirmación, registrar el evento en una bitácora de auditoría, recalcular inventario, entre otras. Si el componente que origina el suceso invoca directamente a cada uno de estos servicios secundarios, el resultado es un acoplamiento alto y una violación de la modularidad, además de que agregar una nueva reacción al suceso obligaría a modificar el componente emisor. La infraestructura de eventos de Spring resuelve este problema permitiendo que el emisor publique un `ApplicationEvent` sin conocer quién lo procesará; los distintos `ApplicationListener` registrados en el contenedor reaccionan de forma autónoma, ya sea de manera síncrona o asíncrona según el `Executor` configurado, sin que el publicador del evento tenga conocimiento de su existencia.

### 4.4 Evidencia de código

El siguiente fragmento corresponde a la declaración de la interfaz `ApplicationListener`, que representa el contrato del observador (Spring Projects, s.f.):

```java
@FunctionalInterface
public interface ApplicationListener<E extends ApplicationEvent> extends EventListener {

    /**
     * Handle an application event.
     */
    void onApplicationEvent(E event);
}
```

El parámetro genérico `<E extends ApplicationEvent>` garantiza seguridad de tipos, permitiendo que cada observador se suscriba exclusivamente al tipo específico de evento que le interesa procesar. El siguiente fragmento, correspondiente a `SimpleApplicationEventMulticaster`, muestra el despacho de la notificación a todos los observadores registrados:

```java
@Override
public void multicastEvent(final ApplicationEvent event, ResolvableType eventType) {
    ResolvableType type = (eventType != null ? eventType : ResolvableType.forInstance(event));
    Executor executor = getTaskExecutor();
    for (ApplicationListener<?> listener : getApplicationListeners(event, type)) {
        if (executor != null) {
            executor.execute(() -> invokeListener(listener, event));
        }
        else {
            invokeListener(listener, event);
        }
    }
}
```

Este método itera sobre la colección de observadores compatibles con el tipo de evento emitido y ejecuta la notificación de forma polimórfica mediante `invokeListener`, lo cual constituye el mecanismo de *multicasting* propio del rol de Subject en el patrón Observer.

### 4.5 Principio SOLID asociado

La infraestructura de eventos de Spring refuerza principalmente el **Open/Closed Principle (OCP)**: es posible agregar nuevos manejadores de un suceso creando nuevos componentes `ApplicationListener`, o anotando métodos con `@EventListener`, sin necesidad de modificar el código que originó el evento. Refuerza también el **Single Responsibility Principle (SRP)**, porque el emisor del evento se enfoca exclusivamente en ejecutar su lógica de negocio y notificar el suceso, delegando los efectos secundarios a escuchadores independientes. Por último, respalda el **Dependency Inversion Principle (DIP)**, dado que tanto el publicador como los observadores interactúan a través de las abstracciones `ApplicationEvent` y `ApplicationListener<E>`, sin depender directamente el uno del otro. Esta relación coincide con lo expuesto en la Guía de Contenido, donde se identifica a Observer como uno de los patrones GoF que implementan directamente el principio OCP en frameworks reales (Universidad UDES, 2026).

---

## 5. Conclusiones

El análisis del código fuente de Spring Framework confirma que los patrones de diseño del catálogo GoF no son construcciones puramente académicas, sino soluciones estructurales que sostienen el funcionamiento de un framework utilizado a nivel industrial. Factory Method permite que el contenedor de Spring Boot controle por completo el ciclo de vida de los beans sin acoplar al cliente a clases concretas; Facade reduce drásticamente el código repetitivo y propenso a fallos que exige el API nativo de JDBC; y Observer permite que múltiples módulos reaccionen a un mismo suceso de negocio sin generar dependencias directas entre ellos. En los tres casos se observa además que el patrón no aparece de forma aislada, sino que refuerza directamente uno o más principios SOLID, particularmente DIP, SRP y OCP, lo que confirma la relación complementaria y no alternativa entre ambos marcos conceptuales. La lección principal para el diseño de software propio es que un patrón bien aplicado no debe buscarse por sí mismo, sino como respuesta a un problema de acoplamiento o de mantenibilidad concreto, tal como ocurre en cada uno de los tres casos documentados en este análisis.

---

## 6. Referencias

Universidad UDES. (2026). *Guía de Contenido — Unidad 1: Fundamentos de Patrones de Diseño y Buenas Prácticas* [Documento de curso]. Ingeniería de Sistemas.

Gamma, E., Helm, R., Johnson, R., & Vlissides, J. (1994). *Design patterns: Elements of reusable object-oriented software*. Addison-Wesley.

Refactoring Guru. (s.f.). *Design patterns*. Recuperado el 17 de agosto de 2026, de https://refactoring.guru/design-patterns

Spring. (s.f.). *Spring Boot reference documentation*. Recuperado el 17 de agosto de 2026, de https://docs.spring.io/spring-boot/reference/

Spring Projects. (s.f.). *spring-framework* [Repositorio de código]. GitHub. Recuperado el 17 de agosto de 2026, de https://github.com/spring-projects/spring-framework