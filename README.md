# chiquillo-post1-u1

Post-contenido — Refactorización SOLID y análisis de patrones GoF en Spring

## Análisis de Violaciones SOLID

| Principio | Método/Sección afectada | Descripción de la violación |
|-----------|-------------------------|-----------------------------|
| SRP | `calculateTotal` + `applyDiscount` + `saveOrder` + `sendEmail` + `printReport` | La clase actúa como un *God Object* al centralizar múltiples responsabilidades incompatibles entre sí. Mezcla lógica de negocio (cálculos de impuestos y descuentos), persistencia de datos (almacenamiento simulado en lista), notificación externa (envío de correos) y la capa de presentación (impresión en consola). Según el principio de responsabilidad única, una clase debe tener un único motivo para cambiar; cualquier cambio en el formato del reporte, en las notificaciones o en los cálculos obliga a alterar esta misma clase. |
| OCP | `applyDiscount` (if/else sobre `customerType`) | El método viola el principio Abierto/Cerrado al utilizar condicionales rígidos para determinar el descuento según el tipo de cliente. Si en el futuro se requiere agregar un nuevo tipo de cliente, es obligatorio modificar directamente el código fuente de esta clase en lugar de extender su comportamiento mediante abstracciones, elevando el riesgo de introducir regresiones. |
| DIP | Toda la clase (dependencias internas sin abstracciones) | La clase viola la Inversión de Dependencias al acoplarse directamente a implementaciones concretas en lugar de depender de abstracciones. Instancia internamente la colección de almacenamiento y realiza salidas directas a consola en lugar de recibir interfaces inyectadas por constructor, lo que imposibilita sustituir componentes y dificulta la creación de pruebas unitarias. |
