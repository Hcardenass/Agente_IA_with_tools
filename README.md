# Agente de IA Conversacional con Integración de Múltiples Herramientas API - Langgraph/Langchain/MemorySaver

## 1. Diseño de arquitectura
1.	Usuario: 
•	Inicia la conversación desde la interfaz Gradio.

2.	Agente Conversacional (LangGraph + LangChain):
•	Recibe el mensaje y consulta MemorySaver para contexto.
•	Determina la intención y elige la herramienta/API adecuada.

3.	Enrutamiento de Herramientas (@tool):
   
•  APIs externas:

    -	PokeAPI
    -	LinkedIn Data API
    -	Amazon Search API

•  APIs propias (desplegadas en Cloud Run):

    -	Futbolista API (posición de jugador)
    -	One Piece API (tripulación de personaje)
    -	Traducción API (texto → inglés)

4.	Llamada a las API
•	El agente hace la petición HTTP y obtiene JSON o texto.
•	Parsea y formatea los datos (tipos de Pokémon, resumen de perfil, listado de productos…).

5.	Generación de Respuesta:
•	El LLM (OpenAI) toma esos datos y los convierte en lenguaje natural.

6.	Salida al Usuario:
•	Devuelve la respuesta al usuario en Gradio.
•	Guarda el intercambio en memoria para posibles seguimientos.

![Arquitectura de Agente con Herramientas API](Arquitectura%20Multiherramientas.gif)


## 2. Descripción de Herramientas y Funciones

Herramienta/API	Función principal
PokeAPI	extractorPokemon(name: str) → str
– Recibe el nombre de un Pokémon y devuelve sus tipos y dos habilidades principales.	 

LinkedIn Data API	extractorLinkedIn(url: str) → str
– Obtiene el headline y el summary de un perfil profesional de LinkedIn.

Amazon Search API	extractorProductoAmazon(query: str) → str
– Busca por palabra clave en Amazon y retorna las 3 primeras coincidencias con título, precio, rating y enlace.	 

YouTube Media Downloader API	extractorAlbumes(cantante: str) → str
– Busca playlists de un artista en YouTube y devuelve los títulos de las 3–5 más relevantes.	 

Futbolista API (personalizada)	posiciondejugador(futbolista: str) → str
– Devuelve la posición en el campo de un jugador de fútbol dado su nombre.	 

One Piece API (personalizada)	obtenerTripulacion(personaje: str) → str
– Retorna el nombre de la tripulación pirata a la que pertenece un personaje de One Piece.	 

API de Traducción (personalizada)	translate_to_english(texto: str) → str
– Traduce cualquier texto dado al inglés mediante un endpoint propio.	 

## 4. Despliegue de la aplicación con Gradio

```python

!pip install -qU gradio

import gradio as gr
from langchain_core.messages import HumanMessage

def run_agent(prompt: str) -> str:
    respuesta = ""
    for step in agent.stream(
        {"messages": [HumanMessage(content=prompt)]},
        config,
        stream_mode="values",
    ):
        respuesta = step["messages"][-1].content
    return respuesta

def respond(user_message, chat_history):
    if chat_history is None:
        chat_history = []
    bot_message = run_agent(user_message)
    chat_history.append((user_message, bot_message))
    return "", chat_history

with gr.Blocks() as demo:
    with gr.Row():
        # Columna izquierda muy estrecha
        with gr.Column(scale=1, min_width=150):
            gr.Markdown(
                """
                ## 🤖 Agente Multiherramientas
                **APIs:**  
                • PokeAPI  
                • LinkedIn  
                • Amazon  
                • Futbolista (Cloud Run)  
                • One Piece (Cloud Run)  
                • Traducción (Cloud Run)
                """
            )
        # Columna derecha grande para el chat
        with gr.Column(scale=3):
            chatbot = gr.Chatbot(label="Chat", height=600)
            txt = gr.Textbox(
                show_label=False,
                placeholder="Escribe tu mensaje aquí…",
                lines=1
            )
            send = gr.Button("Enviar")
            send.click(respond, [txt, chatbot], [txt, chatbot])
            txt.submit(respond, [txt, chatbot], [txt, chatbot])

demo.launch(share=True)

```
## 5. Reflexión

A medida que sumas más fuentes de datos, el agente puede dudar sobre cuál usar para una pregunta concreta, lo que podría retrasar o complicar la respuesta. Por eso es clave contar con un enrutador inteligente que evalúe rápidamente cada consulta y seleccione la API adecuada, y con una memoria de contexto que refuerce patrones ya usados. Con esa orquestación ágil y precisa, tu agente dejará de ser un simple integrador para convertirse en un “superagente” capaz de aprovechar múltiples herramientas sin perder velocidad ni coherencia.

