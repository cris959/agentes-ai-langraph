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
