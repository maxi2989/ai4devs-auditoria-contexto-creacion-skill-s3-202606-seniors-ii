# Entregable · Sesión 3 — Copilotos IA

- **Nombre / usuario:**
- **Fecha de entrega:**
- **Repo auditado en la Parte A:** API NET 6 que se utiliza para orquestar llamadas a otras APIs con más de 2 años de desarrollo.

---

## 1. Hallazgos de la auditoría (Parte A)

1. Las invocaciones a las otras APIs se hacen a través de un ApiGateway (no lo detectó)
2. No advirtió que se usa Dapper como ORM en caso de hacer consultas SQL en el repository
3. Si bien detectó que esta API es la encargada de interacturan con Microsoft Dynamics, nunca especificó como es el proceso de autenticación con esta plataforma.

---

## 2. SKILL.md de la skill creada (Parte B)

> Pega aquí el contenido completo de tu `.claude/skills/<nombre-skill>/SKILL.md`
> (o enlaza al archivo en tu repositorio sandbox).

```markdown
---
name: documentar-csharp-xml
description: >-
  Documenta métodos, clases, interfaces y propiedades de C# con comentarios XML en español. Usar al documentar funciones .NET, añadir o completar /// summary, param, returns, remarks, exception, typeparam, o cuando el usuario pida documentación XML en C#.
---

# Documentar funciones C# con XML

## Objetivo

Añadir o mejorar comentarios XML (`///`) en código C# para mostrar información clara y consistente.

## Reglas generales

1. **Idioma:** español en `<summary>`, `<remarks>`, `<param>`, `<returns>`, `<exception>` y textos visibles por negocio. Mantener términos técnicos habituales en inglés (DTO, API, JWT, HTTP, async).
2. **Sangría:** tabulaciones, igual que el código del proyecto.
3. **Alcance:** documentar solo lo que se pide o el miembro editado; no documentar en bloque archivos enteros salvo que lo pidan.
4. **Estilo:** frases completas en presente o infinitivo («Obtiene…», «Valida…»). Sin redundar el nombre del miembro («Este método obtiene…» → «Obtiene…»).
5. **Precisión:** describir comportamiento real (validaciones, null, excepciones, efectos secundarios). No inventar reglas de negocio.
6. **Nullable:** si un parámetro o retorno es anulable, indicarlo en el texto cuando aporte valor («Identificador del socio; null si no se conoce»).

## Workflow

1. Leer la firma, el cuerpo y usos del miembro.
2. Identificar tipo de miembro (método, propiedad, clase, interfaz, enum).
3. Elegir etiquetas XML necesarias (ver tabla).
4. Redactar `<summary>` en una o dos frases.
5. Añadir `<param>`, `<returns>`, `<exception>`, etc. solo cuando aporten información no obvia.
6. En controladores API, complementar con `[ProducesResponseType]` si el proyecto ya lo usa.
7. Verificar que cada `<param name="...">` coincide exactamente con el nombre del parámetro en C#.

## Etiquetas XML por caso

| Situación | Etiquetas |
|-----------|-----------|
| Cualquier miembro público | `<summary>` |
| Método con parámetros | `<param name="nombre">` |
| Método con retorno | `<returns>` |
| Método genérico | `<typeparam name="T">` |
| Lanza excepciones documentables | `<exception cref="TipoExcepcion">` |
| Comportamiento no obvio o contexto de negocio | `<remarks>` |
| Uso recomendado | `<example>` + `<code>` |
| Referencia cruzada | `<seealso cref="Miembro"/>` |
| Propiedad | `<summary>`; `<value>` solo si el significado del valor no es obvio |

**Orden recomendado:** `summary` → `remarks` → `typeparam` → `param` → `returns` → `exception` → `example` → `seealso`.

## Plantillas

### Método de servicio / aplicación

```csharp
/// <summary>
/// Obtiene la cobertura vigente del socio indicado.
/// </summary>
/// <param name="socioId">Identificador único del socio.</param>
/// <param name="cancellationToken">Token para cancelar la operación asíncrona.</param>
/// <returns>DTO con la cobertura activa, o null si el socio no existe.</returns>
/// <exception cref="ArgumentOutOfRangeException">Se produce cuando <paramref name="socioId"/> es menor o igual a cero.</exception>
public async Task<CoberturaDto?> ObtenerCoberturaAsync(int socioId, CancellationToken cancellationToken)
```

### Clase o interfaz

```csharp
/// <summary>
/// Orquesta las operaciones de consulta y actualización de cobertura de socios.
/// </summary>
public interface ICoberturaService
```

### Propiedad

```csharp
/// <summary>
/// Indica si el socio tiene cobertura activa en la fecha de consulta.
/// </summary>
public bool TieneCoberturaActiva { get; set; }
```

### Controlador API (XML + atributos OpenAPI)

```csharp
/// <summary>
/// Devuelve la cobertura vigente de un socio.
/// </summary>
/// <param name="socioId">Identificador del socio.</param>
/// <param name="cancellationToken">Token de cancelación.</param>
/// <returns>Cobertura encontrada.</returns>
/// <response code="200">Cobertura obtenida correctamente.</response>
/// <response code="404">No existe un socio con el identificador indicado.</response>
[HttpGet("{socioId:int}")]
[ProducesResponseType(typeof(CoberturaDto), StatusCodes.Status200OK)]
[ProducesResponseType(typeof(HandlerExceptionResponse), StatusCodes.Status404NotFound)]
public async Task<IActionResult> GetCoberturaAsync(int socioId, CancellationToken cancellationToken)
```

Usar `<response code="...">` en acciones de controlador cuando el proyecto documenta códigos HTTP en XML; mantener coherencia con `[ProducesResponseType]`.

### Enum

```csharp
/// <summary>
/// Estados posibles de una solicitud de cobertura.
/// </summary>
public enum EstadoSolicitud
{
	/// <summary>
	/// La solicitud fue recibida y está pendiente de revisión.
	/// </summary>
	Pendiente,

	/// <summary>
	/// La solicitud fue aprobada.
	/// </summary>
	Aprobada
}
```

## Qué no documentar

- Miembros `private` salvo que el equipo lo exija.
- Getters/setters triviales sin lógica (`{ get; set; }` obvios).
- Parámetros `cancellationToken` con texto genérico repetitivo en todo el archivo; una vez por método basta con la plantilla estándar.
- Comentarios que solo repiten el tipo («parámetro de tipo int»).

## Edición de documentación existente

- Conservar información correcta; mejorar redacción y completar huecos.
- No eliminar `<exception>` o `<param>` válidos.
- Si la firma cambió, actualizar nombres en `<param>` y eliminar los obsoletos.

## Checklist antes de terminar

- [ ] Cada `<param name="...">` existe en la firma.
- [ ] `<returns>` coherente con el tipo de retorno (incl. `Task`, `void`, nullable).
- [ ] `cref` en `<exception>` y `<seealso>` apunta a tipos visibles.
- [ ] Texto en español, sin errores de concordancia.
- [ ] Sin documentación contradictoria con el código.

## Ejemplos adicionales

Ver [examples.md](examples.md) para casos con genéricos, métodos de extensión y repositorios.
```

**examples.md**
# Ejemplos — documentación XML en C#

## Método genérico

```csharp
/// <summary>
/// Mapea una colección de entidades al DTO indicado.
/// </summary>
/// <typeparam name="TEntity">Tipo de entidad de origen.</typeparam>
/// <typeparam name="TDto">Tipo de DTO de destino.</typeparam>
/// <param name="entidades">Colección a mapear.</param>
/// <returns>Lista de DTOs mapeados; vacía si <paramref name="entidades"/> es null o está vacía.</returns>
public IReadOnlyList<TDto> Mapear<TEntity, TDto>(IEnumerable<TEntity>? entidades)
```

## Método de extensión

```csharp
/// <summary>
/// Registra los servicios de aplicación de cobertura en el contenedor DI.
/// </summary>
/// <param name="services">Colección de servicios de la aplicación.</param>
/// <returns>La misma instancia de <paramref name="services"/> para encadenar llamadas.</returns>
public static IServiceCollection AddCoberturaApplication(this IServiceCollection services)
```

## Repositorio (capa Infrastructure)

```csharp
/// <summary>
/// Ejecuta la consulta de cobertura por identificador de socio usando Dapper.
/// </summary>
/// <param name="socioId">Identificador del socio.</param>
/// <param name="cancellationToken">Token de cancelación.</param>
/// <returns>Registro de cobertura o null si no hay resultados.</returns>
public async Task<CoberturaEntity?> ObtenerPorSocioIdAsync(int socioId, CancellationToken cancellationToken)
```

## Validador FluentValidation

```csharp
/// <summary>
/// Define las reglas de validación para <see cref="CrearSocioRequest"/>.
/// </summary>
public class CrearSocioRequestValidator : AbstractValidator<CrearSocioRequest>
{
	/// <summary>
	/// Inicializa las reglas de validación del request de creación de socio.
	/// </summary>
	public CrearSocioRequestValidator()
```

## Antes / después

**Antes (insuficiente):**

```csharp
/// <summary>
/// Get socio
/// </summary>
public SocioDto GetSocio(int id)
```

**Después (alineado con la skill):**

```csharp
/// <summary>
/// Obtiene los datos del socio por su identificador.
/// </summary>
/// <param name="id">Identificador único del socio.</param>
/// <returns>DTO con los datos del socio.</returns>
/// <exception cref="KeyNotFoundException">Se produce cuando no existe un socio con el identificador indicado.</exception>
public SocioDto GetSocio(int id)
```


---

## 3. Diario de decisiones

*Skill creada:* documentar-csharp-xml - La skill guía al agente para documentar miembros C# con comentarios XML (///) en español.

*Decisiones de diseño tomadas:*
- Decisión 1: Texto en español y términos técnicos en inglés para mantener coherencia con las reglas del equipo
- Decisión 2: ejemplos separados en un archivo propio para que el archivo skill sea conciso.
- Decisión 3: especifiqué qué no documentar para no ensuciar el código con demasiada información irrelevante.

*Qué me resultó fácil:*
- Armar el detalle de la skill y los ejemplos porque tenía un documento de base que usamos para unificar la manera de documentar de los desarrolladores

*Qué me resultó ambiguo o difícil de decidir:*
- el nivel de detalle a aplicar y cuantos ejemplos incluír

*Tiempo real invertido:*
- total: ~2 
- lectura previa: ~1h 
- diseño: ~20min
- escritura: ~30min

*Qué probarías si tuvieras más tiempo:*
- mejoraría las convenciones de documentación basándome en algún standard 

*¿Usaste IA para crear la skill?* (qué partes generaste con IA y qué partes decidiste tú)
- si me ayudé con IA para establecer el formato porque la necesitaba para cursor ya que en la empresa usamos esa herramienta, y la documentación que leí era de claude por lo que no estaba seguro que la estructura sea la misma.

### Resultado de la prueba (Paso 8)

- ¿Se activó cuando lo esperabas? si
- ¿El resultado fue el que querías? si, lo realizó como lo hacen los desarrolladores normalmente. De todas maneras pienso que con mejores lineamientos podría mejorarse la documentación adaptándose a algún standard.


**Ejecución**
Voy a leer la skill de documentación XML y el archivo completo para documentar el método en la línea 9.
Documentando el método según la implementación y el estilo de los demás miembros de la interfaz.

**Documentación anterior realizada por un desarrollador**
/// <summary>
/// Envía los datos de un socio a la interface de Business Central
/// </summary>
/// <param name="request"></param>
/// <returns></returns>
Task SendAltaToBC(IEnumerable<SocioRequestDto> request);

**Resultado de la SKILL**
/// <summary>
/// Informa a Business Central el alta de uno o varios socios.
/// </summary>
/// <param name="request">Colección de socios a dar de alta en Business Central.</param>
/// <returns></returns>
Task SendAltaToBC(IEnumerable<SocioRequestDto> request);