# CLAUDE.md

Este archivo proporciona orientación a Claude Code (claude.ai/code) al trabajar con código en este repositorio.

## Descripción General del Proyecto

API de gestión de tareas desarrollada para la cohorte CH70 Java de Generation México. Java 17 + Maven + JUnit 5 con persistencia en memoria.

## Comandos de Compilación

```bash
mvn test                    # Ejecuta la suite completa de pruebas
mvn test -Dtest=ClassName   # Ejecuta una clase de prueba específica
mvn -q exec:java            # Ejecuta la demo de consola (App.java)
```

Nota: No hay Maven wrapper presente - utilizar `mvn` del sistema.

## Arquitectura

Patrón de arquitectura en tres capas:

### Capa de Dominio
- **`Tarea`**: Entidad con propiedades inmutables excepto el flag `completada` y el `id` (asignado por el repositorio)
- **`Prioridad`**: El orden del enum es significativo (BAJA < MEDIA < ALTA) - valores ordinales utilizados para comparaciones
- **`TareaNoEncontradaException`**: RuntimeException lanzada cuando no se encuentra un ID de tarea

### Capa de Repositorio (`TareaRepositorio`)
- Almacenamiento en memoria con `LinkedHashMap<Integer, Tarea>` para preservar el orden de inserción
- Generación de IDs autoincrementales comenzando en 1
- Contrato de asignación de ID: el repositorio invoca `tarea.setId()` cuando `getId() == 0`

### Capa de Servicio (`TareaServicio`)
- Lógica de negocio: filtrado, reportes, cálculos de fechas
- Dependencia `TareaRepositorio` inyectada por constructor
- Interfaz principal para controladores/consumidores CLI (ver `App.java`)

## Problemas Conocidos

### 1. Bug de Búsqueda Case-Sensitive
**Ubicación:** `TareaRepositorio.buscarPorTitulo()` línea 44  
**Problema:** Utiliza `String.contains()` sin normalización  
**Esperado:** Búsqueda case-insensitive según contrato Javadoc  
**Impacto:** Test `buscarPorTituloIgnoraMayusculas` está `@Disabled`

### 2. Error Off-By-One en Filtro de Prioridad
**Ubicación:** `TareaServicio.listarPorPrioridadMinima()` línea 51  
**Problema:** Utiliza `ordinal() >` en lugar de `ordinal() >=`  
**Impacto:** Excluye tareas que coinciden exactamente con el umbral de prioridad mínima

### 3. Cálculo de Fechas Invertido
**Ubicación:** `TareaServicio.diasRestantes()` línea 67  
**Problema:** Argumentos de `ChronoUnit.DAYS.between(fechaLimite, hoy)` invertidos  
**Esperado:** `between(hoy, fechaLimite)` para signo correcto  
**Impacto:** Retorna valores negativos cuando se esperan positivos, y viceversa

## Convenciones de Testing

- `@BeforeEach` setUp crea instancias frescas de servicio y repositorio por cada test
- Constante de fecha fija para aserciones determinísticas: `LocalDate.of(2026, 10, 8)`
- Anotaciones `@Disabled("TODO: ...")` marcan tests fallidos en espera de correcciones de implementación
- Comentarios inline `// TODO:` indican áreas con cobertura de pruebas faltante

## TODOs Pendientes

- `Tarea.java:19` - Agregar validación para título no vacío/no en blanco en el constructor
- `TareaServicioTest.java:39` - Verificar que `completar(999)` lanza `TareaNoEncontradaException`
- `TareaServicioTest.java:102` - Agregar cobertura de pruebas para el método `eliminar(int)`
- `README.md:6` - Completar documentación del README (programado para "Bloque 4")
