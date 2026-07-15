# LangGraph: Orquestacion de Agentes y Multiagentes — Aula 7

Sistema multiagente de gestion de emails construido con LangGraph. Un agente clasificador (triage) analiza correos entrantes y decide si ignorarlos, notificar al usuario o delegarlos a un agente de respuesta que redacta emails, agenda reuniones y consulta disponibilidad.

## Arquitectura

```
                    +-------------------+
   Email entrante   |   triage_router   |
  ----------------->| (structured output)|
                    +--------+----------+
                             |
              +--------------+--------------+
              |              |              |
          "ignore"       "notify"      "respond"
              |              |              |
            [END]          [END]    +-------v--------+
                                   | response_agent |
                                   |  (ReAct agent) |
                                   +----------------+
                                         |
                                   Tools disponibles:
                                   - write_email
                                   - schedule_meeting
                                   - check_calendar_availability
```

El **triage_router** usa `with_structured_output` para clasificar emails en 3 categorias (ignore/notify/respond) con razonamiento paso a paso. Solo los emails clasificados como "respond" se envian al **response_agent**, un agente ReAct con acceso a herramientas.

## Tecnologias

| Componente | Tecnologia |
|---|---|
| Orquestacion | LangGraph (StateGraph, Command routing) |
| LLM | Qwen-Max via Alibaba Model Studio (endpoint OpenAI-compatible) |
| Agente | `create_react_agent` (langgraph-prebuilt) |
| Structured Output | Pydantic + function calling |
| Herramientas | LangChain Tools (`@tool` decorator) |

## Estructura del proyecto

```
.
├── Clase_07_MultiAgente.ipynb   # Notebook principal
├── prompts.py                   # System prompts (triage + agente)
├── .env                         # Variables de entorno (no versionado)
└── README.md
```

## Configuracion

### 1. Entorno virtual

```bash
python -m venv .venv
source .venv/bin/activate  # Linux/Mac
.venv\Scripts\activate     # Windows
```

### 2. Dependencias

```bash
pip install langchain-openai langgraph-prebuilt python-dotenv pydantic langchain-core
```

### 3. Variables de entorno

Crear un archivo `.env` en la raiz del proyecto:

```env
DASHSCOPE_API_KEY=sk-tu-api-key-de-alibaba-model-studio
TAVILY_API_KEY=tvly-tu-api-key-de-tavily
```

La `DASHSCOPE_API_KEY` se obtiene en [Alibaba Model Studio](https://dashscope.console.aliyun.com/). El modelo `qwen-max` incluye un millon de tokens gratuitos.

### 4. Ejecutar

Abrir `Clase_07_MultiAgente.ipynb` y ejecutar todas las celdas en orden.

## Flujo del notebook

1. **Configuracion** — Carga de variables de entorno y creacion del LLM
2. **Router estructurado** — Define el modelo `Router` (Pydantic) y prueba la clasificacion de un email
3. **Herramientas** — Define tools para escribir emails, agendar reuniones y consultar calendario
4. **Agente ReAct** — Crea el agente con `create_react_agent` y lo prueba de forma aislada
5. **Grafo multiagente** — Conecta triage_router + response_agent en un StateGraph con Command routing
6. **Pruebas end-to-end** — Ejecuta el grafo completo con un email de spam (se ignora) y una consulta real (se responde)

## Notas

- El endpoint usado es el internacional de DashScope: `https://dashscope-intl.aliyuncs.com/compatible-mode/v1`
- Las herramientas son simuladas (devuelven strings placeholder) — en produccion se conectarian a APIs reales
- El warning de deprecacion de `create_react_agent` esta silenciado; cuando LangGraph v2 se publique, el import migrara a `from langchain.agents import create_agent`
