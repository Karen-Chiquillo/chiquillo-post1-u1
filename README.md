# chiquillo-post1-u1

Post-contenido — Refactorización SOLID y análisis de patrones GoF en Spring

## Descripción
Repositorio del post-contenido de la Unidad 1 de Patrones de Diseño de Software — Sexto Semestre. Contiene dos partes: refactorización SOLID de un God Object ([parte-1-refactorizacion-solid/](parte-1-refactorizacion-solid/)) y análisis de patrones GoF en Spring Framework ([parte-2-analisis-gof-spring/](parte-2-analisis-gof-spring/)).

## Parte 1 — Refactorización SOLID
Proyecto Maven que refactoriza la clase monolítica `OrderProcessor` aplicando los principios de diseño SRP, OCP y DIP mediante la separación en clases cohesivas, la adopción del patrón Strategy para el cálculo de descuentos y la inyección de dependencias por constructor. Ver [parte-1-refactorizacion-solid/](parte-1-refactorizacion-solid/).

### Análisis de Violaciones SOLID

| Principio | Método / Sección afectada | Descripción de la violación |
|---|---|---|
| **SRP** | `calculateTotal` + `applyDiscount` + `saveOrder` + `sendEmail` + `printReport` | La clase centraliza múltiples responsabilidades no cohesivas al mezclar lógica de negocio (impuestos y descuentos), persistencia simulada en memoria, envío de notificaciones externas y presentación en consola. Esto genera múltiples motivos de cambio, obligando a modificar la misma clase ante variaciones en el formato de salida, en las notificaciones o en los cálculos. |
| **OCP** | `applyDiscount` (if/else sobre `customerType`) | El método implementa estructuras condicionales rígidas para calcular descuentos según el tipo de cliente. Agregar un nuevo perfil de cliente exige modificar directamente el código fuente de la clase en lugar de extender su comportamiento mediante abstracciones polimórficas, elevando el riesgo de introducir regresiones. |
| **DIP** | Toda la clase (dependencias internas sin abstracciones) | La clase se acopla directamente a implementaciones concretas en lugar de depender de abstracciones. Instancia internamente la colección de almacenamiento y realiza llamadas directas a la consola (`System.out.println`) sin intermediación de interfaces inyectables, lo que impide sustituir componentes y dificulta la creación de pruebas unitarias. |

### Ejecución del Proyecto
Para compilar el código y ejecutar la clase `Main` que demuestra el funcionamiento de las clases refactorizadas, ejecutar los siguientes comandos desde la terminal:

```bash
cd parte-1-refactorizacion-solid
mvn compile
mvn exec:java -Dexec.mainClass="com.patrones.u1.Main"
```

## Parte 2 — Análisis de Patrones GoF en Spring

| # | Patrón | Categoría | Clase en Spring |
|---|---|---|---|
| 1 | Factory Method | Creacional | `org.springframework.beans.factory.BeanFactory` (`spring-beans`) |
| 2 | Facade | Estructural | `org.springframework.jdbc.core.JdbcTemplate` (`spring-jdbc`) |
| 3 | Observer | Comportamiento | `org.springframework.context.ApplicationListener` (`spring-context`) |

Ver el análisis completo en [parte-2-analisis-gof-spring/documento-analisis.md](parte-2-analisis-gof-spring/documento-analisis.md).  
Los extractos de código fuente y capturas de evidencia se encuentran en [parte-2-analisis-gof-spring/evidencia/evidenciaimage.md](parte-2-analisis-gof-spring/evidencia/evidenciaimage.md).

## Herramientas utilizadas
- Java 17, Apache Maven, VS Code, Git, GitHub
- Código fuente de Spring Framework (investigación)

## Conclusiones
El desarrollo de esta actividad evidenció que los principios SOLID y los patrones de diseño GoF constituyen herramientas complementarias para estructurar software mantenible y desacoplado. En la primera parte, la descomposición de `OrderProcessor` demostró cómo la segregación de responsabilidades y la inyección de dependencias permiten transformar una clase monolítica en componentes modulares y fácilmente extensibles sin alterar código existente. Por su parte, la exploración de Spring Framework comprobó que estos mismos principios sustentan arquitecturas empresariales maduras, donde `BeanFactory` independiza la instanciación de objetos, `JdbcTemplate` encapsula la complejidad de JDBC y `ApplicationListener` facilita una comunicación reactiva no intrusiva. En conclusión, la aplicación rigurosa de patrones no responde a una mera convención estilística, sino a la necesidad técnica de aislar responsabilidades y mitigar el impacto de los cambios en el ciclo de vida del software.