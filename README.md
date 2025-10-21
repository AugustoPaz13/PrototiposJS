# EJERCICIO 4: Análisis OOP del Gestor de Tareas

En este proyecto, refactoricé mi código original, que usaba la sintaxis de `clase` (ES6+), para adaptarlo a la implementación clásica de JavaScript basada en **funciones constructoras y prototipos**.

Para mí, la sintaxis `class` en JavaScript es, en gran medida, "azúcar sintáctico" sobre este modelo de prototipos. Por eso, en este análisis detallo qué características de la Programación Orientada a Objetos (OOP) utilicé para resolver el programa en esta versión "clásica".

## Características de OOP que Utilicé

### 1. Abstracción

Para mí, la abstracción es ocultar la complejidad interna y exponer solo lo esencial.

* **Ejemplo:** Mi objeto `ToDoApp` no sabe *cómo* `TareaRepositorio` guarda las tareas. Simplemente lo hice llamar a `this.repo.add(tarea)` o `this.repo.getAll()`. La implementación interna del repositorio (que en mi caso es un simple array `this.tareas`) está totalmente abstraída.
* **Ejemplo 2:** Mi método `tarea.actualizar(...)` abstrae toda la lógica de validación y las reglas de negocio (como que un espacio " " setea `null` o que una string vacía "" mantiene el valor). El código que llama al método no necesita saber esto, solo pasa los nuevos valores.

### 2. Encapsulamiento

El encapsulamiento es agrupar los datos (propiedades) y los métodos que operan sobre esos datos en una misma unidad (el objeto).

* **Ejemplo:** Mi `function Tarea(...)` define la "plantilla". Cuando creo una instancia con `new Tarea(...)`, el objeto que resulta contiene *tanto sus datos* (`this.titulo`, `this.estado`) como el *comportamiento* asociado (`actualizar`) a través de su prototipo.
* **Ocultamiento de datos:** Aunque JavaScript no tiene modificadores `private` reales, logré el encapsulamiento por convención. Decidí que el array `this.tareas` dentro de `TareaRepositorio` no debe ser modificado directamente desde fuera. En su lugar, expuse métodos públicos como `.add()` y `.getAll()` para interactuar con ese estado interno, protegiendo su integridad.

### 3. Herencia (Prototípica)

Esta es la característica central del modelo que implementé. No usé la herencia "clásica" (de extensión de clases), sino la **herencia prototípica** para compartir comportamiento.

* **Cómo lo implementé:** En lugar de definir `actualizar` dentro de la función constructora `Tarea` (lo que crearía una copia de la función por cada instancia), la definí *una sola vez* en el prototipo:
    ```javascript
    Tarea.prototype.actualizar = function(...) { ... };
    ```
* **Por qué es Herencia:** Cuando creo `const t1 = new Tarea(...)`, `t1` no tiene su *propia* función `actualizar`. Cuando llamo a `t1.actualizar()`, JavaScript busca la propiedad en `t1`, no la encuentra, y entonces "sube" por la **cadena de prototipos** hasta `Tarea.prototype`, donde la encuentra y la ejecuta.
* **Beneficio:** Todas mis instancias de `Tarea` *heredan* y *comparten* la misma definición de método desde el prototipo, lo que me permite ahorrar memoria.

### 4. Modularidad (Separación de Responsabilidades)

Aunque no es un "pilar" exclusivo de la OOP, es un principio clave que apliqué. Dividí el código en "clases" (funciones constructoras) con responsabilidades únicas:

* `ConsoleIO`: Maneja *exclusivamente* la entrada y salida de la consola.
* `Tarea`: Representa mi modelo de dominio (los datos y su lógica de negocio).
* `TareaRepositorio`: Maneja el almacenamiento y acceso a los datos.
* `ToDoApp`: Es el orquestador o "controlador" que une todas las partes y maneja el flujo de mi aplicación.

## Características de OOP que NO Utilicé

### 1. Herencia Clásica (Extensión de Clases)

* **Qué es:** Crear una "clase" hija que herede y extienda a una "clase" padre (ej. `function TareaPrioritaria() { ... }` que herede de `Tarea`).
* **Por qué no la usé:** Mi aplicación es simple y no lo requería. No tenía diferentes *tipos* de tareas (como `TareaSimple`, `TareaConPrioridad`) o diferentes *tipos* de repositorios. Si mi problema hubiera sido más complejo, podría haber usado `Object.create()` o `Hija.prototype = new Padre()` para establecer una cadena de prototipos más profunda.

### 2. Polimorfismo (de Subtipo)

* **Qué es:** La capacidad de tratar objetos de diferentes clases (que comparten un ancestro común) de la misma manera.
* **Por qué no lo usé:** Está directamente relacionado con la falta de Herencia Clásica. Al no tener "subtipos" de `Tarea` o `TareaRepositorio`, no tuve la oportunidad de aplicar este tipo de polimorfismo.
* **Nota (Duck Typing):** Se podría argumentar que usé "Duck Typing" (Si camina como un pato y grazna como un pato, es un pato), que es una forma de polimorfismo. Mi `ToDoApp` espera un objeto `repo` que tenga `.add()` y `.getAll()`. No me *importa* si es un `TareaRepositorio` o cualquier otro objeto, siempre que cumpla con esa "interfaz". En ese sentido, sí apliqué un polimorfismo implícito, pero no el polimorfismo clásico basado en herencia.
