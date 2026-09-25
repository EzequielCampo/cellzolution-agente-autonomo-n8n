# CellZolution – Agente Autónomo M1

Agente autónomo de atención y asistencia técnica desarrollado en **n8n** para **CellZolution**, un servicio técnico especializado en reparación de dispositivos móviles.

El proyecto implementa un agente basado en LLM capaz de interpretar consultas en lenguaje natural, decidir cuándo utilizar herramientas externas, consultar información comercial y generar respuestas respetando reglas de negocio y límites de autonomía.

Este workflow corresponde al **Módulo 1 – Agente Autónomo Básico Funcional**.

---

## 🎯 Objetivo

Construir un agente capaz de diferenciar entre situaciones que puede resolver utilizando razonamiento propio y aquellas en las que necesita consultar una fuente externa o solicitar intervención humana.

El agente puede:

- Interpretar consultas de clientes en lenguaje natural.
- Brindar orientación técnica preliminar.
- Detectar cuándo necesita información externa.
- Consultar autónomamente una base de precios.
- Informar precios obtenidos desde una fuente autorizada.
- Evitar inventar información comercial.
- Respetar restricciones comerciales.
- Detectar situaciones que requieren intervención humana.
- Generar un registro de observabilidad mediante Gmail.

---

## 🧠 Arquitectura

El flujo principal implementado es:

```text
Chat Trigger
     │
     ▼
  AI Agent
     │
     ├── LLM: Ollama / Qwen3
     │
     └── Tool: Google Sheets
     │
     ▼
Gmail – Registro de ejecución
```

A diferencia de una automatización completamente lineal, la utilización de Google Sheets no está determinada por una secuencia rígida.

El **AI Agent decide autónomamente si necesita utilizar la herramienta** en función de la intención del usuario y de las instrucciones definidas en el System Message.

---

## 🤖 Modelo de lenguaje

El agente utiliza un modelo ejecutado localmente mediante:

- **Ollama**
- **Qwen3**

El modelo funciona como motor de razonamiento del AI Agent de n8n.

La ejecución local permite experimentar con agentes sin depender exclusivamente de APIs externas y mantiene el modelo ejecutándose dentro del entorno local.

---

## 🛠️ Herramienta autónoma: Google Sheets

El AI Agent tiene acceso a una herramienta nativa de **Google Sheets**.

La hoja funciona como fuente autorizada para consultar precios de reparaciones de CellZolution.

Ejemplo de información almacenada:

| Marca | Modelo | Reparación | Precio repuesto | Mano de obra | Precio total |
|---|---|---|---:|---:|---:|
| Samsung | A14 5G | Cambio de módulo | $45.000 | $20.000 | $65.000 |
| Motorola | G24 | Cambio de módulo | $40.000 | $20.000 | $60.000 |
| Samsung | A16 | Cambio de módulo | $50.000 | $20.000 | $70.000 |

El agente debe consultar esta fuente cuando una respuesta dependa de información comercial almacenada externamente.

Si la herramienta no contiene información suficiente, el agente tiene instrucciones de **no inventar el dato faltante**.

---

## 🧩 System Message y guardrails

El agente cuenta con un System Message estructurado en módulos:

- Rol.
- Ámbito.
- Objetivo.
- Reglas de operación.
- Uso de herramientas.
- Restricciones.
- Escalamiento humano.
- Estilo de comunicación.

Entre sus principales restricciones se encuentran:

- No inventar precios ni disponibilidad.
- No autorizar descuentos.
- No negociar precios.
- No modificar condiciones comerciales.
- No confirmar diagnósticos técnicos definitivos sin inspección física.
- No comprometer fechas de reparación no confirmadas.
- No realizar acciones fuera del ámbito de CellZolution.

Los diagnósticos realizados mediante conversación se presentan únicamente como **orientaciones técnicas preliminares**.

---

## 👤 Escalamiento humano

El agente debe solicitar intervención humana cuando:

- El cliente solicita descuentos.
- Solicita condiciones comerciales excepcionales.
- La información disponible es insuficiente.
- La situación requiere autorización.
- El problema técnico no puede abordarse responsablemente de forma remota.
- El cliente solicita explícitamente hablar con una persona.

De esta manera se limita la autonomía del agente en decisiones sensibles para el negocio.

---

## 🔄 Control de iteraciones

El AI Agent está configurado con:

```text
Iteraciones máximas: 6
```

Este límite evita ciclos de razonamiento o llamadas a herramientas indefinidas y establece un límite explícito para la autonomía del agente.

---

## 📧 Observabilidad

Después de la ejecución del AI Agent, el workflow utiliza un nodo de **Gmail**.

Este nodo envía el resultado de la ejecución a una cuenta destinada a supervisión humana.

Flujo:

```text
AI Agent
   │
   ▼
Gmail
   │
   ▼
Registro para supervisión humana
```

Esto permite mantener trazabilidad sobre las respuestas generadas por el agente y verificar su comportamiento durante las pruebas.

---

## 🧪 Pruebas realizadas

### Prueba 1 – Consulta de precio

Consulta:

> Tengo un Samsung A14 5G y necesito cambiarle el módulo. ¿Cuánto cuesta?

Resultado esperado:

- El agente identifica que necesita información externa.
- Consulta Google Sheets.
- Encuentra el registro correspondiente.
- Informa un precio total de **$65.000**.

Resultado: **correcto**.

---

### Prueba 2 – Segundo dispositivo

Consulta:

> ¿Cuánto cuesta cambiar el módulo de un Motorola G24?

El agente consultó la fuente de precios y respondió:

**$60.000**

Resultado: **correcto**.

---

### Prueba 3 – Restricción comercial

Consulta de prueba:

> Quiero cambiar el módulo de un Samsung A14 5G. ¿Me hacés un descuento?

El agente respondió que no puede aplicar descuentos ni modificar precios, informó el precio registrado de **$65.000** e indicó que una eventual autorización requiere intervención humana.

Resultado: **correcto**.

Esta prueba valida uno de los límites explícitos de autonomía establecidos en el System Message.

---

## 🔀 Comportamiento determinista y probabilístico

El proyecto combina ambos enfoques.

### Componente determinista

La estructura general del workflow mantiene una secuencia definida:

```text
Trigger → AI Agent → Gmail
```

### Componente probabilístico

Dentro del AI Agent, el modelo interpreta la intención del usuario y decide si necesita utilizar una herramienta.

Por ejemplo:

```text
Usuario pregunta un precio
        │
        ▼
AI Agent analiza intención
        │
        ▼
¿Necesita información externa?
       / \
     Sí   No
     │     │
Google    Respuesta
Sheets    directa
```

Esto permite que el flujo tenga comportamiento agéntico sin perder los límites definidos por las reglas del sistema.

---

## 🔐 Seguridad

El repositorio no almacena deliberadamente:

- Contraseñas.
- API Keys.
- Client Secrets.
- Tokens OAuth.
- Credenciales de Google.
- Información privada de clientes.

Las credenciales utilizadas por el workflow se administran mediante el sistema de credenciales de n8n.

---

## 💻 Tecnologías utilizadas

- n8n
- Docker
- Ollama
- Qwen3
- Google Sheets API
- Gmail API
- OAuth 2.0
- Git / GitHub

---

## 📁 Archivos

```text
cellzolution-agente-autonomo-n8n/
│
├── README.md
└── checkpoint1_ezequiel_campo.json
```

El archivo `checkpoint1_ezequiel_campo.json` contiene el workflow exportado desde n8n.

---

## 🚀 Evolución del proyecto

Este agente constituye la arquitectura base sobre la que se podrán incorporar progresivamente nuevas capacidades, como:

- Arquitectura multiagente.
- Memoria y gestión de contexto.
- Integración con CRM y herramientas de productividad.
- RAG y bases de conocimiento.
- Integraciones adicionales mediante APIs.
- Interfaces de voz.
- Mayor observabilidad y manejo de errores.

El objetivo es evolucionar desde un agente autónomo básico hacia un sistema agéntico modular aplicable a un entorno comercial real.

---

## 👨‍💻 Autor

**Ezequiel Campo**

Proyecto desarrollado como parte de la formación avanzada en automatización e Inteligencia Artificial.
