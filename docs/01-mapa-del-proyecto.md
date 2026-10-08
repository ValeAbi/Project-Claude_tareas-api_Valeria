# Mapa del Proyecto: Tareas API

## 1. Descripción del Proyecto

Este proyecto es un gestor de tareas pendientes desarrollado en Java 17 con Maven, diseñado como repositorio de práctica para la mentoría técnica de Generation México (cohorte CH70 Java). Implementa un sistema completo de gestión de tareas con persistencia en memoria, pruebas unitarias con JUnit 5, y una arquitectura en tres capas (dominio, repositorio, servicio).

## 2. Tabla de Clases

| Nombre de Clase | Responsabilidad | Dependencias |
|-----------------|-----------------|--------------|
| `Tarea` | Entidad de dominio que representa una tarea con id, título, descripción, prioridad, fecha límite y estado de completada | `Prioridad`, `LocalDate` |
| `Prioridad` | Enumeración que define tres niveles de prioridad (BAJA, MEDIA, ALTA) cuyo orden es significativo para comparaciones | Ninguna |
| `TareaNoEncontradaException` | Excepción de tipo RuntimeException que se lanza cuando se busca una tarea por un id inexistente | Ninguna |
| `TareaRepositorio` | Capa de persistencia en memoria usando LinkedHashMap, gestiona almacenamiento y asignación de IDs autoincrementales | `Tarea`, `LinkedHashMap`, `ArrayList` |
| `TareaServicio` | Capa de lógica de negocio que expone casos de uso (crear, completar, eliminar, listar, generar reportes) | `TareaRepositorio`, `Tarea`, `Prioridad`, `TareaNoEncontradaException`, `LocalDate`, `ChronoUnit` |
| `App` | Clase principal con método main que ejecuta una demostración de consola creando tareas y mostrando un reporte | `TareaServicio`, `TareaRepositorio`, `Prioridad`, `LocalDate` |

## 3. Compilación y Pruebas

### Compilar el proyecto:
```bash
mvn clean compile
```

### Ejecutar todas las pruebas:
```bash
mvn test
```

### Ejecutar una clase de prueba específica:
```bash
mvn test -Dtest=TareaServicioTest
mvn test -Dtest=TareaRepositorioTest
```

### Ejecutar la aplicación de demostración:
```bash
mvn -q exec:java
```

### Empaquetar el proyecto:
```bash
mvn package
```

**Nota:** El proyecto no incluye Maven wrapper, por lo que se debe usar el comando `mvn` del sistema.

## 4. Tres Cosas Sospechosas o Incompletas

### 1. Búsqueda de títulos sensible a mayúsculas/minúsculas
**Archivo:** `TareaRepositorio.java`  
**Línea:** 44  
**Descripción:** El método `buscarPorTitulo()` usa `t.getTitulo().contains(texto)` que es sensible a mayúsculas/minúsculas, pero el Javadoc del método (líneas 38-40) promete explícitamente que la búsqueda debe ser "sin distinguir mayúsculas de minúsculas". Por ejemplo, buscar "informe" no encontrará "INFORME". Esto causa que el test `buscarPorTituloIgnoraMayusculas` esté deshabilitado con `@Disabled`.

### 2. Validación faltante de tarea nula al completar
**Archivo:** `TareaServicio.java`  
**Línea:** 26  
**Descripción:** El método `completar(int id)` llama a `repositorio.buscar(id)` que puede retornar `null` si el id no existe, pero luego invoca directamente `tarea.marcarCompletada()` sin verificar si `tarea` es null. Esto provocaría un `NullPointerException` en lugar de lanzar la `TareaNoEncontradaException` prometida en el Javadoc del método (línea 23). Debería usar el método privado `obtener(id)` que sí maneja correctamente este caso.

### 3. Filtro de prioridad excluye el valor exacto solicitado
**Archivo:** `TareaServicio.java`  
**Línea:** 51  
**Descripción:** El método `listarPorPrioridadMinima(Prioridad minima)` usa la condición `t.getPrioridad().ordinal() > minima.ordinal()` con operador mayor que (`>`), pero el Javadoc (líneas 45-47) indica que debe incluir tareas con prioridad "igual o mayor". Por ejemplo, si se solicita `MEDIA`, debería retornar tareas MEDIA y ALTA, pero actualmente solo retorna ALTA. El operador correcto sería mayor o igual (`>=`).
