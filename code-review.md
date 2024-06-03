# Code Review

Code Review realizada por Jose Luzón Martín para [Reto: Servicio para gestión de calidad de los anuncios](https://github.com/idealista/code-review-challenge)

## Consideraciones

A pesar de la falta de contexto, se asume que el desarrollo se ha llevado a cabo según convenios dentro de un equipo de desarrollo (vease, versión de springboot, java, etc.), por ello:

1. Se va a tratar de hacer un primer acercamiento desde el punto de vista de respetar el trabajo y tiempo dedicado por un compañero o compañera, centrádome principalmente en aspectos de mejorar del código fuente, entendiendo que se están siguiendo las convenciones del equipo.

2. Puesto que se trata de una entevista laboral, añadiré un apartado de posibles mejoras que, normalmente, no tendrían cabida en un code review, y sí en una discusión a más alto nivel sobre convenios del equipo de desarrollo.

## Review

### Historia de usuario

- Se echa en falta especificación del caso de uso para el primer cálculo de la puntuación de los anuncios.
- Se echa en falta especificación del caso de uso para la actualización de la puntuación del anuncio al modificar el mismo. (por ejemplo, si cambia el tamaño de la descripción)

### README.md

- Falta información de setup, build, despliegue.

### Arquitectura

- Uso de arquitectura hexagonal
  - Capa de aplicación:
    - Ok
  - Capa de dominio:
    - `Ad#isComplete` es lógica de negocio. Podría implementarse con patrón estrategia en capa de aplicación.
  - Capa de Infrastructure.
    - OK
    - Se asume que la implementación de persistencia en memoria es por simplicidad, y que no cabría la posibilidad en entorno real (salvo quizá testing).

### Código fuente

- Validar la construcción de dominio:
  - Todo `Ad` tiene `id`, `typology`...
- Evitar el uso de @Autowired (`AdsController#14`, `AdsServiceImpl#16`), usar en su lugar inyección por constructor.
- Refactorizar `PublicAd` y `QualityAd`
- Comentarios deben ser en inglés.
- Añadir javadoc a clases y métodos públicos.
- Mejoras/Errores en `AdsServiceImpl#calculateScore`:
  - Posible excepción por nulo en `AdsServiceImpl#71`
  - Error en calculo cuando `Ad#isComplete` devuelve `true`, debe sumar 40, y está machacando el valor calculado previamente.
  - `wds` no parece un nombre de variable recomendable
  - Usar notación `ifPresent` en `AdsServiceImpl#86`
  - Usar `Stream::forEach`en `AdsServiceImpl#74`
  - Mejorar el truncado por score < 0 y score > 100
  - Evitar el uso de literales string por constantes finales. (Idealmente, se podría incluso configurar de alguna manera, via BBDD, el conjunto de palabras clave de la descripción y el delta que aporta a la puntuación)
  - Usar fecha UTC en `AdsServiceImpl#133`
- Los mapeos objeto dominio -> objeto api en `AdsServiceImpl#findPublicAds` y `AdsServiceImpl#findQualityAds` deberían hacer en capa api de infraestructura.
- No se ha implementado control de usuarios, ni por tanto de roles, para proteger el acceso a los endpoints únicamente para usuario con rol QA ("/ads/score" y "/ads/quality").
- "/ads/public" devuelve los anuncios en orden creciente de puntuación, cuando el requisito es que sea orden decreciente (mejor a peor).
- Posible excepción por nulo en `InMemoryPersistence#findRelevantAds` y `InMemoryPersistence#findIrrelevantAds`
- Se echa en falta @ControllerAdvice @ExceptionHandler para control de errores.

### Log / Errores

- No existen logs.
- 
### Tests

- Sólo existe un test unitario, que además no hace lo sugiere el nombre ```AdsServiceImplTest#calculateScoresTest``` que debería, al menos, comprobar que ha calculado scores, sin embargo, únicamente busca scores y los almacena.
- Es necesario ampliar test unitarios para ampliar cobertura de código.

## Mejoras Plantadas

### Actualización de versiones

- Versión de SpringBoot sin soporte. Tratar de actualizar a versiones 3.2.X mínimo siguiendo [guía de actualización](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0-Migration-Guide)
- Esta actualización de Springboot implicaría actualizar la versión de Java a la vesión 17 mínimo.

### CI/CD

- Asumiendo Github como repositorio de código, uso de workflows para CI/CD.

### Dockerizar el servicio

- Creación de Dockerfile para la creación del contenedor del servicio.
- Creación de docker-compose para despliegue local de desarrollo/tests

### Maven wrapper

- Para unificar la versión de maven que use el equipo de desarrollo, así como CI/CD, uso de maven wrapper.

### Testing

- Incorporar tests de integración.
- Incorporar test de aceptación (uso de cucumber, por ejemplo)

### Enfoque API first (OpenAPI)

- Especificación de contrato API (fichero yaml)
- Autogeneración de controladores.

### Lombok

- Uso de lombok para evitar *boilerplate code* (contructores, getter/setter, ...) `PublicAd` `QualityAd` `Ad` `Picture`

### mapstruct

- Uso de mapstruct para evitar la implementación de mapeos a mano. Por ejemplo, `InMemoryPersistence#mapToDomain`

### Checkstyle

- Añadir checkstyle al build para unificar criterios de código.

### Análisis de código

- Añadir herramientas de análisis de código (sonarqube, spotbugs, ...)
- Análisis de vulnerabilidades.