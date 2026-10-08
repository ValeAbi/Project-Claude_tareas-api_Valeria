# CLAUDE.md

Orientación para Claude Code al trabajar en este repositorio.

## Stack

- **Java 17** con Maven 3.8+
- **JUnit 5** para pruebas
- Persistencia en memoria (sin base de datos)

## Comandos

```bash
mvn clean compile           # Compilar
mvn test                    # Ejecutar todas las pruebas
mvn test -Dtest=ClaseTest   # Ejecutar una clase de prueba específica
mvn -q exec:java            # Ejecutar la demo de consola (App.java)
```

No hay Maven wrapper: usar `mvn` del sistema.

## Convenciones

### Código
- Todo el código, comentarios y nombres en **español** (clases, métodos, variables, Javadoc)
- Arquitectura en tres capas: dominio (`Tarea`, `Prioridad`), repositorio (`TareaRepositorio`), servicio (`TareaServicio`)
- No agregar dependencias nuevas sin preguntarme antes

### Pruebas
- **JUnit 5** únicamente
- Nombres de pruebas descriptivos en español que explican el comportamiento esperado
- **Una prueba por comportamiento**: cada `@Test` verifica un único caso o escenario
- `@BeforeEach` para instancias frescas en cada prueba
- Usar fechas fijas en constantes para pruebas determinísticas

## Regla de Oro

**Cualquier cambio que hagas debe dejar `mvn test` en verde.** Si un test falla después de tu cambio, debes arreglarlo antes de considerar la tarea completa. Los tests deshabilitados con `@Disabled` pueden ignorarse temporalmente.
