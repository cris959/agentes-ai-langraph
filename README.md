# 🚀 Práctica Didáctica: Conexión Nativa con Google Gemini API 2026

Este proyecto es una guía práctica y didáctica para aprender a integrar los modelos de Inteligencia Artificial de Google de forma nativa utilizando el nuevo SDK oficial (`google-genai`), abstrayéndonos de librerías intermedias y configurando el entorno de forma segura en **Google Colab**.

## 🔑 Obtención de las Credenciales (Gemini API Key)
Para poder interactuar con los modelos de inteligencia artificial de este proyecto, es necesario contar con una clave de API personal y gratuita provista por Google. Esta llave actúa como tu pase de acceso y vincula las peticiones de tu código con los servidores oficiales de la IA.
### 📌 Pasos para generar tu API Key:
1- Ingresar a Google AI Studio: Dirigite al portal oficial de desarrolladores a través del siguiente enlace:
👉 **Google AI Studio** (Get API Key)

2- Iniciar Sesión: Accedé utilizando cualquier cuenta de Google (Gmail).

3- Crear la Clave: Hacé clic en el botón destacado que dice "**Create API Key**" (Crear clave de API).

4- Seleccionar el Proyecto: Podés asociarla a un proyecto existente de Google Cloud o elegir la opción de crear una clave en un proyecto nuevo de forma automática.

5- Copiar y Guardar: Copiá el string de texto largo que se genera (comienza habitualmente con **AQ.Ab...**) y guardalo de forma segura en el panel de Secretos (Llavecita) de tu entorno de Google Colab bajo el nombre de **GEMINI_KEY**.
___
⚠️ Nota de seguridad: Mantener esta clave bajo estricta confidencialidad. El entorno de desarrollo está configurado para leerla de forma interna a través de variables de entorno del sistema (**userdata.get**), evitando así que la credencial quede expuesta públicamente al subir el repositorio a plataformas como GitHub.
____

## 📌 Contexto y Aprendizaje (Troubleshooting)
Durante el desarrollo de este laboratorio, nos enfrentamos y solucionamos varios desafíos típicos de entornos de producción:
* **Bypass de Intermediarios:** Descartamos el uso de frameworks externos que arrastraban conflictos de rutas y desactualizaciones de endpoints.
* **Manejo de Errores Críticos:** Solucionamos problemas de compatibilidad de API (`404 NOT_FOUND` por endpoints obsoletos de la generación 1.5) e identificamos cuotas excedidas (`429 RESOURCE_EXHAUSTED`).
* **Auditoría de Modelos:** Implementamos scripts de inspección directa por HTTP para mapear el catálogo real activo de la cuenta (`ListModels`), descubriendo la disponibilidad y migrando con éxito hacia la joya de la corona actual: **Gemini 2.5 Flash**.


## 🛠️ El Script de Inspección que abrió el candado
Lo que hicimos fue usar la librería requests de Python para pegarle directamente al endpoint de inspección de Google (ListModels).

Este es el bloque de código didáctico que ejecutamos para obtener la verdad del servidor:

````
Python
import requests
import json
from google.colab import userdata

print("🔍 Inspeccionando los modelos disponibles para tu GEMINI_KEY...")

try:
    # 1. Traemos tu llave del panel seguro
    api_key = userdata.get("GEMINI_KEY")
    
    # 2. Le pegamos al endpoint oficial que lista los modelos del proyecto
    url = f"https://generativelanguage.googleapis.com/v1beta/models?key={api_key}"
    
    response = requests.get(url)
    response_data = response.json()
    
    if response.status_code == 200:
        print("\n✅ ¡Lista obtenida con éxito! Modelos que soportan 'generateContent':")
        print("-" * 60)
        
        # 3. Recorremos la lista que mandó Google
        modelos = response_data.get('models', [])
        for m in modelos:
            # Filtramos solo los que sirven para chatear o generar texto
            if 'generateContent' in m.get('supportedGenerationMethods', []):
                # Imprimimos el string exacto que Python necesita
                print(f"👉 {m['name']}")
                
        print("-" * 60)
    else:
        print(f"\n❌ Error del servidor (Código {response.status_code}):")
        print(json.dumps(response_data, indent=2))

except Exception as e:
    print(f"\n❌ Error inesperado: {e}")
````

## 🛠️ Arquitectura del Código

El proyecto está modularizado en tres celdas independientes para mantener una lógica limpia y escalable:

1. **Celda 1 (Configuración):** Inicializa el cliente oficial leyendo de forma segura las credenciales sin exponer código sensible en texto plano.
2. **Celda 2 (Abstracción):** Una función "caja negra" (`consultar_gemini`) que encapsula la construcción del Prompt (utilizando *f-strings* dinámicos) y la llamada al modelo.
3. **Celda 3 (Orquestación):** El punto de entrada donde se define el caso de uso (ej. analizar los impactos de la IA en la educación) y se ejecuta el flujo.

---

## 🚀 Requisitos e Instalación

Para replicar este cuaderno en tu entorno de desarrollo, asegurate de seguir estos pasos:

## 1. Instalar el SDK Oficial de Google
```bash
pip install google-genai

```


## 2. Configurar la Variable de Entorno (Seguridad Ante Todo)
Este repositorio no contiene llaves expuestas por motivos de seguridad informática. El código utiliza la API de secretos del sistema para inyectar la credencial en tiempo de ejecución.

° En Google Colab: Dirigirse al panel izquierdo de la Llavecita (Secrets), añadir una clave con el nombre exacto de GEMINI_KEY y activar el interruptor azul de "Notebook access".

° En Entorno Local: Configurar la variable en tu terminal o archivo .env:
```
Bash
export GEMINI_KEY="tu_api_key_aquí"
```

## 💻 El Código Principal

````
import os
from google.colab import userdata
from google import genai

# 1. Conexión segura
api_key = userdata.get("GEMINI_KEY")
client = genai.Client(api_key=api_key)

# 2. Función modular
def consultar_gemini(tema: str) -> str:
    prompt = f"Explica de manera clara y actualizada cuáles son los impactos de la inteligencia artificial en el área de {tema}."
    try:
        response = client.models.generate_content(
            model='gemini-2.5-flash',
            contents=prompt,
        )
        return response.text
    except Exception as e:
        return f"❌ Error en la petición: {e}"

# 3. Ejecución del Caso de Uso
tema_interes = "educación"
resultado = consultar_gemini(tema_interes)
print(resultado)
````

## 📊 Límites de la Capa Gratuita (Free Tier)

Para evitar bloqueos por consumo de recursos (**RESOURCE_EXHAUSTED**), tener en cuenta las siguientes reglas de juego para **gemini-2.5-flash**:

* 1.500 solicitudes por día (Máximo diario).

* 15 solicitudes por minuto (RPM). Nota: Si usas bucles automatizados, agrega un retraso con time.sleep() para evitar saturar el canal.

* 1.000.000 de tokens por minuto (TPM).

# Agente Autónomo de Consulta en Tiempo Real con LangGraph y Gemini 2.5

Este proyecto implementa un agente de inteligencia artificial autónomo utilizando el patrón de arquitectura **ReAct (Reasoning and Acting)**. La solución combina las capacidades de razonamiento avanzado de **Gemini 2.5 Flash** con la potencia del motor de búsqueda web de **Tavily API**, permitiendo responder a consultas de usuarios con datos actualizados en tiempo real mediante un flujo de ejecución dinámico orquestado por **LangGraph**.


## 🔑 Configuración del Entorno y Obtención de API Keys

Para que el agente pueda autenticarse con los servicios externos, es necesario dar de alta las credenciales de los proveedores de IA y búsqueda en la web.

### 1. Obtención de la API Key de Tavily
1. Ingresa al sitio oficial de desarrolladores en [Tavily AI (https://tavily.com)](https://tavily.com).
2. Regístrate o inicia sesión (puedes vincular directamente tu cuenta de GitHub).
3. Ve al panel principal (*Dashboard*) y en la sección **API Keys** copia la clave pública generada automáticamente en tu plan gratuito (*Free Tier* de 1,000 búsquedas mensuales).

### 2. Almacenamiento Seguro en Google Colab
Por motivos de seguridad y para evitar la filtración accidental de credenciales en los *commits* de GitHub, **no se deben escribir las API Keys directamente en el código de las celdas**. 

En su lugar, el proyecto utiliza el gestor de secretos nativo de Google Colab:

1. En el menú lateral izquierdo de tu cuaderno de Colab, haz clic en el icono de la **llave de seguridad (Secrets / Secretos)**.
2. Agrega una nueva variable con el nombre exacto: `TAVILY_API_KEY`.
3. En el campo *Value*, pega la clave que copiaste desde el panel de Tavily.
4. Asegúrate de activar el interruptor de **Acceso al cuaderno (Notebook access)** para que el entorno de Python pueda consumir la variable en tiempo de ejecución.
5. Repite el mismo proceso para tu credencial de Gemini bajo el nombre `GEMINI_KEY`.

El backend del proyecto inyectará automáticamente estas claves en las variables de entorno del sistema operativo utilizando la librería `google.colab.userdata`:

````
python
import os
from google.colab import userdata

# Inyección automatizada y segura en el entorno de ejecución
os.environ["TAVILY_API_KEY"] = userdata.get("TAVILY_API_KEY")
os.environ["GOOGLE_API_KEY"] = userdata.get("GEMINI_KEY")
````


## 🚀 Arquitectura del Sistema

A diferencia de los scripts secuenciales rígidos, este desarrollo implementa un agente capaz de decidir de forma autónoma si requiere buscar información externa o si puede responder directamente desde su base de conocimiento integrada.

El bucle de ejecución se basa en el ciclo clásico de los Agentes ReAct:
1. **Pensamiento (Thought):** El modelo analiza el prompt del usuario y evalúa si necesita herramientas adicionales para cumplir el objetivo.
2. **Acción (Action):** Si requiere datos en tiempo real (por ejemplo, resultados deportivos recientes, noticias de última hora), invoca autónomamente la herramienta `web_search_tool` (Tavily).
3. **Observación (Observation):** El agente recibe el JSON crudo con los resultados de la web, extrae el contenido semántico relevante y las fuentes correspondientes.
4. **Respuesta Final (Final Answer):** Se consolida un reporte en lenguaje natural enriquecido con citas directas y enlaces de referencia.

## 🛠️ Tecnologías y Librerías Utilizadas

* **Python 3.12** (Entorno de ejecución en Google Colab)
* **Google Gemini 2.5 Flash** (Modelo Fundacional de Lenguaje Principal)
* **LangGraph (v1.0+)**: Orquestador del ciclo de vida y manejo del estado del agente autónomo (`create_react_agent`).
* **LangChain Google GenAI (`ChatGoogleGenerativeAI`)**: Puente de conexión moderno para interactuar con la API de Google de forma estructurada.
* **Tavily Search API**: Herramienta optimizada para LLMs encargada del web scraping y la extracción de fuentes limpias.

## 📦 Características Destacadas e Inyecciones de Comportamiento

* **Autonomía Decisoria:** El modelo consume recursos de búsqueda en internet de manera inteligente, evitando consultas redundantes para preguntas de cultura general.
* **Transparencia en Fuentes (Citations):** Se implementó una regla de negocio inyectada directamente en el flujo conversacional (`messages`) que obliga al agente a estructurar de forma estricta una sección de **"Fuentes consultadas"** con los hipervínculos exactos de donde extrajo los datos (por ejemplo, reportajes de ESPN, El País o Sofascore).
* **Robustez ante Cambios de API:** El backend está diseñado utilizando la sintaxis de mensajes nativa de LangGraph, previniendo errores comunes de firmas depreciadas como `state_modifier` o wrappers antiguos de agentes centralizados de LangChain.

## 💻 Ejemplo de Flujo Interno

````
text
👤 Usuario: ¿Cómo salió el partido de Estados Unidos contra Paraguay anoche en el mundial?
⏳ El agente evalúa la línea temporal del corte de conocimiento...
🧠 Pensamiento: "No poseo el resultado en mi base de datos fija. Debo ejecutar Tavily."
🔧 Acción: web_search_tool.invoke("resultado Estados Unidos vs Paraguay mundial 2026")
📄 Observación: { "results": [{"url": "...", "content": "..."}] }
🤖 Respuesta Final: [Reporte Deportivo Redactado] + Sección de Fuentes con enlaces verificados.
````

## 📚 Agente de Búsqueda Científica Inteligente con LangGraph

![GitHub License](https://img.shields.io/github/license/cris959/agentes-ai-langraph?color=blue&style=flat-square)
![Python Version](https://img.shields.io/badge/python-3.12-brightgreen?style=flat-square)
![Framework](https://img.shields.io/badge/framework-LangGraph-orange?style=flat-square)    

Este proyecto implementa un Sistema Multi-Agente orquestado con LangGraph y motorizado por Gemini 2.5 Flash. La aplicación optimiza costos de infraestructura mediante un ruteo determinista en Python y expone una interfaz visual interactiva utilizando Gradio.

El objetivo principal es resolver consultas de usuarios decidiendo dinámicamente si se requiere una búsqueda avanzada de papers científicos en la API de arXiv o si se puede resolver mediante el conocimiento general del modelo, unificando los resultados en un reporte final estructurado.

## 🏗️ Arquitectura del Sistema
A diferencia de los agentes ReAct tradicionales y lineales, este sistema utiliza un grafo de estados condicional (StateGraph) para controlar el flujo de ejecución de forma eficiente:

````
Plaintext
                        ┌──> [Investigador Node] ──> [Redactor Node] ──> [END]
                        │         (API arXiv)             (LLM Report)
[START] ──> [Ruteador] ─┤
             (Python)   │
                        └──> [Asistente Casual Node] ────────────────────> [END]
                                  (LLM Chat)
````
1- START: El usuario ingresa su consulta a través de la interfaz.

2- Ruteador Determinar (Middleware): Una función pura en Python analiza la consulta mediante palabras clave. Si detecta intención científica, desvía el flujo al nodo de investigación; si es charla o teoría general, lo manda al nodo casual. Esto ahorra un 33% de llamadas al LLM por ejecución, protegiendo la cuota de la API.

3- Investigador Node: Consume de forma nativa la API pública de arXiv usando urllib (evitando conflictos de dependencias de librerías de terceros) y extrae títulos y abstracts.

4- Redactor Node: Toma la información cruda del estado y el LLM genera un informe final unificado y formateado en Markdown.

5- Asistente Casual Node: Responde directamente consultas generales sin tocar herramientas externas.

## 🛠️ Tecnologías Utilizadas
* Core AI: LangChain & LangGraph (Orquestación de agentes).

* LLM: Google Gemini 2.5 Flash (langchain-google-genai).

* UI Framework: Gradio (Interfaz de usuario localizada al castellano).

* Backend Tools: Python Nativo (urllib, xml.etree.ElementTree) para el consumo de la API de arXiv.

## 🚀 Instalación y Configuración
Para correr este proyecto en tu entorno local o en Google Colab, asegurate de instalar las versiones estables compatibles para evitar conflictos con los servidores web internos:

````
Bash

# Instalación de dependencias de IA
pip install -qU langchain-google-genai langgraph langchain-core

# Instalación del ecosistema de servidores y UI (Versiones Blindadas)
pip install -qU starlette==0.49.1 fastapi gradio
````

## Variables de Entorno
Configurá tu API Key de Google AI Studio antes de iniciar el grafo:

````
Python
import os
os.environ["GOOGLE_API_KEY"] = "TU_GEMINI_API_KEY"
````

## 💻 Estructura del Código Principal
El corazón del proyecto radica en la abstracción de la herramienta nativa y la compilación del flujo de trabajo:
````
Python
# Herramienta nativa de parsing XML para arXiv
@tool
def tool_cientifica(query: str) -> str:
    """Busca artículos científicos en arXiv y retorna sus títulos y resúmenes."""
    # ... (Lógica de consumo con urllib y parsing ET)
````    
La definición de las conexiones de nuestro Grafo de Estados se configura de la siguiente manera:

````
Python
workflow_inteligente = StateGraph(EstructuraEstado)

# Registro de Nodos
workflow_inteligente.add_node("investigador_node", agente_investigador)
workflow_inteligente.add_node("redactor_node", agente_redactor)
workflow_inteligente.add_node("casual_node", agente_casual)

# Ruteo Condicional Eficiente (0% Costo de API)
workflow_inteligente.add_conditional_edges(
    START,
    ruteador_inteligente,
    {
        "ir_a_investigar": "investigador_node",
        "ir_a_charla": "casual_node"
    }
)

# Conexiones Fijas
workflow_inteligente.add_edge("investigador_node", "redactor_node")
workflow_inteligente.add_edge("redactor_node", END)
workflow_inteligente.add_edge("casual_node", END)

app_con_ruteador = workflow_inteligente.compile()
````

## 🖥️ Interfaz de Usuario (Gradio)
La aplicación incluye una interfaz web limpia con sus controles completamente en castellano:

* Enviar Consulta: Procesa el grafo y devuelve la respuesta en formato Markdown enriquecido.

* Limpiar: Resetea el cuadro de diálogo.

````
Python
iface = gr.Interface(
    fn=run_graph,
    inputs=gr.Textbox(lines=3, label="Escribe tu pregunta:"),
    outputs=gr.Markdown(label="Respuesta del Agente Científico"),
    title="📚 Agente de Búsqueda Científica Inteligente",
    submit_btn="Enviar Consulta",
    clear_btn="Limpiar",
    flagging_mode="never"
)
iface.launch(share=True)
````

## 📝 Licencia

Este proyecto está bajo la Licencia MIT. Para más detalles, consulta el archivo [LICENSE](https://github.com/cris959/agentes-ai-langraph/blob/main/LICENSE) adjunto en este repositorio.

Copyright © 2026 [Christian Garay](https://github.com//cris959/agentes-ai-langraph) - Backend Developer.
