# Agente **Daniela** — asistente virtual para atención al cliente (n8n)

> Asistente conversacional multicanal, construido como flujo de **n8n**. Integra WhatsApp/Chatwoot, LLMs y PostgreSQL para **detectar intención**, **guiar el siguiente paso** y **automatizar** tareas como registro de clientes, envío de catálogo, búsqueda de productos y solicitudes de precios.

## Tabla de contenidos
- [Características](#-características)
- [Arquitectura](#-arquitectura-alto-nivel)
- [Integraciones](#-integraciones)
- [Requisitos](#-requisitos)
- [Importación del flujo en n8n](#-importación-del-flujo-en-n8n)

## ✨ Características
- **Webhook Chatwoot**: recibe mensajes (texto/archivo/audio) y responde directamente en la conversación.
- **Transcripción de audio** con **Gemini** y normalización de texto.
- **Motor de intenciones** (LLM con salida JSON estricta) que decide entre:
  - `register_new_client`
  - `send_catalog` (3 mensajes con *delays* crecientes de 0–3 s)
  - `search_products`
  - `request_prices`
  - `update_cliente_data`
- **Memoria de clientes** en **PostgreSQL** (`client_memory`) y **memoria de chat** por número telefónico.
- **Catálogo**: descarga automática desde **Google Drive** y envío como adjunto cuando aplica.
- **Productos**: consultas SQL sobre productos (código, marca, nombre y categorías normalizadas).
- **Cotizaciones**: resolución de listas activas y control de cuota; si hay límite, redirección automática al equipo adecuado.
- **Enrutamiento a equipos** (Ventas, Asesoría científica, Servicio técnico, Finanzas).
- **Redis** para *debounce*, deduplicación de mensajes, colas y almacenamiento temporal de lotes/selecciones.
- **Formateo de respuestas**: mensajes compactos, listados por lotes (50 ítems), disponibilidad y vencimiento.
- **Reglas duras**: validación de *delays*, consistencia JSON (`terms`/`terms_json`) y precondiciones por flujo.

## 🧱 Arquitectura (alto nivel)
1. Webhook recibe el mensaje y detecta el tipo (texto/audio/archivo).  
2. Si es audio, se transcribe con **Gemini**.  
3. Se consolida contexto y se pasa al **Agente LLM** con *schema* de salida estricta.  
4. Según la acción, se consulta **PostgreSQL/Redis**, se envía **catálogo**, se buscan **productos** o se solicita **precios**.  
5. Las respuestas se envían a **Chatwoot**, respetando los *delays* y el orden.  
6. Si corresponde, se **redirige** la conversación al equipo adecuado.

## 🔌 Integraciones
- **Chatwoot API** (mensajes y asignaciones)
- **Gemini** (transcripción de audio)
- **OpenAI** (razonamiento y salida JSON)
- **PostgreSQL** (memoria de clientes, productos y utilidades)
- **Google Drive** (catálogo PDF)
- **Redis** (estado efímero, lotes, antirrebote)

## 📦 Requisitos
- n8n ≥ 1.0
- PostgreSQL 13+  
- Redis 6+  
- Cuenta/clave para OpenAI y Gemini  
- Instancia de Chatwoot con acceso a la API  
- Catálogo en Google Drive (PDF)

## 📥 Importación del flujo en n8n
1. Abre **n8n** → *Workflows* → **Import**.  
2. Selecciona el archivo `Agente Daniela.json` (incluido en este repo).  
3. Revisa los **credenciales**/**variables** que pide cada nodo (Chatwoot, OpenAI, Gemini, Postgres, Redis, Google Drive).  
4. Guarda y **Activa** el workflow.


