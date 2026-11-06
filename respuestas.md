# Respuestas — Deber 6

## A2UI

El ejemplo usa un hallazgo real de la revisión de seguridad. En `buscar_usuario`, el argumento `nombre` se concatena dentro de una consulta SQL y después se ejecuta. Si un llamador controla ese argumento, podría cambiar la condición de búsqueda. Esto se comprobó leyendo el código, sin ejecutar `usuarios.py`.

### Salida convencional

| Archivo | Líneas | Problema | Severidad |
| --- | --- | --- | --- |
| `usuarios.py` | 18 y 21–22 | Posible inyección SQL por concatenar `nombre` en la consulta, sin parámetros SQL. | Alta |

### Descripción de la tarjeta

Ilustración didáctica; no es un mensaje A2UI estándar válido

```json
{
  "catalog": "revision-seguridad-v1",
  "surfaceId": "revision-sql",
  "component": "Card",
  "children": [
    {"component": "Text", "text": "Archivo: usuarios.py"},
    {"component": "Text", "text": "Líneas: 18 y 21–22"},
    {"component": "Text", "text": "Problema: posible inyección SQL"},
    {"component": "Text", "text": "Severidad: alta"},
    {
      "component": "Button",
      "id": "pedir-revision",
      "label": "Solicitar revisión adicional",
      "action": {
        "name": "pedir_revision",
        "context": {
          "findingId": "sql-buscar-usuario",
          "file": "usuarios.py",
          "lines": [18, 21, 22]
        }
      }
    }
  ]
}
```

Los identificadores del JSON son propuestos para este diseño. El catálogo permitido sería cerrado y conocido por el cliente:

| Componente | Uso y límites |
| --- | --- |
| `Card` | Agrupa los componentes aprobados de un hallazgo; no carga contenido externo. |
| `Text` | Muestra texto plano; no interpreta HTML, scripts ni código. |
| `Button` | Muestra una etiqueta y permite únicamente el evento `pedir_revision` con el contexto definido. No recibe comandos ni URLs. |

En una implementación real, el agente o LLM produciría la descripción a partir del hallazgo verificado. El cliente comprobaría que el JSON se pueda interpretar y cumpla el esquema, los tipos, campos, identificadores, límites de tamaño y catálogo acordados. Después, el renderer convertiría los componentes aprobados en controles de la plataforma, con estilos controlados por la aplicación. No ejecutaría código enviado por el modelo. [Catálogos de A2UI](https://a2ui.org/concepts/catalogs/).

**Generative UI** es la idea general de generar o adaptar una interfaz con IA. **A2UI** ofrece un protocolo declarativo para describir interfaces con componentes acordados entre agente y cliente. MCP conecta agentes con herramientas y recursos, pero **no es obligatorio para renderizar**: el renderer puede recibir la descripción por otro transporte. [A2UI](https://a2ui.org/), [transportes](https://a2ui.org/concepts/transports/).

Si llega JSON inválido, un componente prohibido, una acción peligrosa o una URL no autorizada, el cliente debe rechazar la propuesta, bloquear la interacción y mostrar un error o la tabla convencional como alternativa segura. No debe intentar ejecutar lo recibido. En este catálogo no se permiten URLs; una ampliación requeriría una lista explícita de destinos autorizados.

Al pulsar el botón, el evento propuesto enviaría `pedir_revision`, el identificador del hallazgo, el archivo y las líneas. El cliente añadiría `surfaceId` e identificador del botón para relacionarlo con la tarjeta. No enviaría credenciales ni el proyecto completo. El backend obtendría la identidad desde la sesión autenticada y comprobaría la autorización sobre el proyecto y el hallazgo, la acción permitida y la validez de los datos. No confiaría en una ruta o una identidad declarada por el JSON. El botón solo solicitaría otra revisión; no autorizaría correcciones ni un Pull Request. La validación visual no sustituye a estos controles del servidor. [Acciones de A2UI](https://a2ui.org/concepts/actions/).

El archivo `evidencias/parte4_interfaz.png` es un boceto estático identificado como propuesta visual. No es una captura de una aplicación funcionando y su botón no envía eventos.

## Preguntas finales

### 1. ¿Por qué dos especialistas y no uno solo? ¿Qué mejora y qué costo agrega?

Separar seguridad y propuestas de pruebas permite que cada especialista siga un objetivo concreto y entregue resultados fáciles de comprobar. El orquestador puede compararlos y reunirlos. El costo es más consumo de tokens, coordinación y trabajo para detectar respuestas repetidas o contradictorias. En este deber, el especialista de pruebas quedó diseñado; no se afirma que se haya ejecutado.

### 2. ¿Cómo verificaste la ejecución real del subagente y sus hallazgos? ¿Qué harías si fallara?

Se utilizó **Codex CLI 0.162.1** con una invocación explícita de `revisor-seguridad`, definido en `.codex/agents/revisor-seguridad.toml`. La captura muestra la señal **`Started /root/revision_usuarios`**, el seguimiento de la ejecución y la respuesta posterior. Los hallazgos se contrastaron con las líneas reales del código, sin ejecutarlo. La captura conserva completos los hallazgos 3–5; los dos primeros se comprobaron directamente por lectura.

Si el especialista fallara, el orquestador marcaría esa parte como incompleta, conservaría lo válido e informaría el motivo. Solicitaría una corrección o un nuevo intento autorizado, sin inventar resultados ni presentar la revisión como terminada.

### 3. ¿Qué permisos mínimos le diste y qué no pudo garantizar Codex?

El alcance mínimo indicado fue lectura y búsqueda exclusivamente en el archivo recibido, sin modificar archivos, ejecutar el módulo, consultar otros archivos ni implementar correcciones. El TOML contiene **instrucciones de solo lectura**, pero no garantiza técnicamente esos permisos: el subagente depende de los permisos y del sandbox de la sesión principal. No se debe confundir una instrucción de comportamiento con un bloqueo técnico. [Permisos de subagentes en Codex](https://learn.chatgpt.com/docs/agent-configuration/subagents).

### 4. ¿Cuándo bastan subagentes locales y cuándo convendría A2A?

Los subagentes locales bastan cuando el equipo trabaja dentro del mismo entorno y bajo la configuración de la sesión principal. A2A convendría si el especialista pertenece a otra empresa o funciona en otro servidor, con descubrimiento, mensajes por red y autenticación propios. La delegación realizada fue a un **subagente local de Codex**, no a un servidor A2A. La Parte 3 solo plantea ese escenario remoto; no implementa A2A.

### 5. ¿Cómo manejaría el cliente `input-required` o una Task fallida?

Con `input-required`, el cliente mostraría qué información falta y enviaría los datos autorizados conservando el identificador de la tarea y su contexto. Con `failed`, informaría el error y mantendría el trabajo como incompleto. Si corresponde reintentar, iniciaría otra Task después de revisar la causa; una tarea terminada en fallo no se reinicia. [Ciclo de una tarea A2A](https://a2a-protocol.org/latest/topics/life-of-a-task/).

### 6. ¿Qué valida el cliente antes de dibujar la tarjeta y qué valida el backend antes de atender el evento?

El cliente valida el JSON, el esquema, el catálogo, los textos y las acciones permitidas; también bloquea URLs no autorizadas. El backend valida la autenticación, la autorización sobre el recurso y la acción, y comprueba los datos contra el hallazgo conocido. Que un botón se vea correctamente no demuestra que quien lo pulsa tenga permiso para ejecutar su acción.

### 7. ¿Cuándo conviene usar una plantilla fija en lugar de generar la interfaz con el LLM?

Una plantilla fija conviene cuando los campos y las acciones siempre son los mismos, como esta tarjeta de seguridad. Es más predecible, fácil de probar y mantener, y reduce costo y demora. El LLM puede limitarse a entregar datos validados. Generar la interfaz resulta más útil cuando cambian las necesidades y se justifica adaptar su estructura.

## Evidencias y verificación

Las cinco imágenes existen en la copia local y se enlazan con rutas relativas:

| Parte | Evidencia | Estado |
| --- | --- | --- |
| 1 | [Flujo del equipo](evidencias/parte1_flujo.png) | Existe. |
| 2 | [Definición del subagente](evidencias/parte2_subagente.png) | Existe. |
| 2 | [Delegación y respuesta](evidencias/parte2_llamada.png) | Existe; la respuesta visible es parcial, como se explica en la pregunta 2. |
| 3 | [Agent Card y flujo A2A](evidencias/parte3_a2a.png) | Existe. |
| 4 | [Propuesta visual A2UI](evidencias/parte4_interfaz.png) | Existe; es un boceto didáctico. |

No se modificaron `usuarios.py`, `equipo.md` ni el subagente, y no se volvieron a ejecutar agentes. El archivo `Deber6.md` no se encontró en la copia local; las respuestas siguen las siete preguntas proporcionadas para esta entrega, en su mismo orden.
