# 📦 LATAM Shipping & Customs AI Context Layer

[![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)]()
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Language](https://img.shields.io/badge/language-ES%20%7C%20EN-green.svg)]()
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)]()
[![Ko-fi](https://img.shields.io/badge/Support-Ko--fi-FF5E5B?logo=ko-fi&logoColor=white)](https://ko-fi.com/yellowtips)

> **ES:** Capa de contexto para que los LLMs asistan en envíos internacionales y despacho aduanero en LATAM y España: umbrales de minimis, documentación obligatoria, Incoterms, tiempos de courier vs. postal y restricciones por país — sin inventar aranceles ni confundir regímenes simplificados con importación comercial.
> **EN:** A context layer so LLMs assist with international shipping and customs clearance across LATAM and Spain: de minimis thresholds, mandatory documentation, Incoterms, courier vs. postal transit times and country restrictions — without inventing tariffs or confusing simplified regimes with commercial imports.

> 📄 **Versión:** 1.0.0 · **Protocolo:** Open-AI-Context-Standard · **Ámbito:** México, Colombia, Argentina, Chile, Perú y España · **Uso:** System prompt injection para asistentes de ecommerce, agentes logísticos, chatbots de courier y automatizaciones de fulfillment.

> 🇪🇸 [Español](#-español) · 🇬🇧 [English](#-english)

> ⚠️ **Aviso:** los umbrales y aranceles cambian con frecuencia (Brasil y México reformaron sus regímenes en 2026). Esta capa es orientativa; verifica siempre con la autoridad aduanera del país de destino y con un agente aduanal certificado antes de despachar.

---

## 🇪🇸 Español

### 🎯 Propósito del sistema

Cuando un ecommerce o un freelancer le pregunta a un LLM *"¿cuánto me cuesta enviar un paquete de 2 kg a México?"*, la respuesta por defecto suele ser un número inventado, sin distinguir entre **régimen simplificado (T1)** e **importación comercial**, y sin mencionar que el umbral de minimis mexicano es de **$50 USD** y que, por encima de ese monto, el comprador paga un **17% global** entre $50 y $117, y un **19%** por encima de $117[reference:0].

El problema no es que el modelo no sepa de logística, sino que **no tiene el marco operativo de las aduanas hispanoamericanas**. Le falta:

- Un **mapeo de umbrales de minimis por país** (México $50, Colombia $200, Chile $41, Perú $200; Argentina sin de minimis pero con régimen courier de hasta $3,000)[reference:1][reference:2].
- **Conciencia de los regímenes simplificados vs. comerciales**: el régimen T1 de México es solo para uso personal; si es para reventa, aplican aranceles comerciales completos[reference:3].
- **Tiempos reales de courier vs. postal**: DHL/FedEx a México 7–11 días; China Post 25–38 días[reference:4].
- **Documentación obligatoria**: factura comercial, packing list, certificado de origen, BL/AWB. Sin estos, el despacho se detiene[reference:5].
- Una **lógica de fallback** que derive a agente aduanal cuando el caso exija clasificación arancelaria (fracción TIGIE, partida arancelaria) o interpretación de restricciones no arancelarias.

Esta capa resuelve exactamente eso. No es un cotizador de fletes: es **la estructura de contexto** que hace que un LLM explique el costo real de un envío, distinga regímenes y no invente aranceles.

---

### 🧭 Navegación rápida

| Si eres… | Empieza por… |
| :--- | :--- |
| **Ecommerce / Freelancer** que envía productos al exterior | [Ejemplos Before / After](#-ejemplos-de-uso-before--after) → [Cómo inyectar](#-cómo-inyectar-esta-capa-de-contexto) |
| **Dev** que integra la capa en un checkout o ERP logístico | [Estructura JSON](#️-estructura-de-contexto-para-modelos-de-ia) → [Cómo inyectar](#-cómo-inyectar-esta-capa-de-contexto) |
| **Colaborador** que quiere añadir un país o un courier | [Reglas globales](#️-reglas-globales-de-interpretación-para-la-ia) → PR con nuevo `country_profiles` |

---

### 🛠️ Reglas globales de interpretación para la IA

Cuando el usuario consulte sobre envíos o aduanas, la IA **DEBE** aplicar siempre esta lógica de 5 pasos, en orden:

1. **Identificación de origen, destino y propósito.** Determinar el país del remitente, el país del destinatario y si el envío es **B2C personal** (régimen simplificado) o **comercial/reventa** (régimen general). Sin esto, no se puede aplicar el umbral correcto.
2. **Verificación del umbral de minimis.** Consultar el umbral del país de destino (ver `country_profiles`). Si el valor FOB está por debajo → régimen simplificado con exención o tasa reducida. Si está por encima → aranceles e impuestos completos.
3. **Cálculo de costos desglosado.** Separar: flete internacional, seguro (si aplica), arancel de importación, IVA/IGV/ICMS, tasas aduaneras y courier fee. **Nunca** dar un “costo total” sin desglose.
4. **Validación de documentación.** Verificar que existan: factura comercial, packing list, certificado de origen (si aplica preferencia arancelaria) y guía aérea/BL. Si falta alguno, indicar el impacto (retención en aduana).
5. **Advertencia de restricciones.** Si el producto es de categoría restringida (electrónicos, cosméticos, alimentos, textiles), verificar si requiere NOM (México), registro sanitario (Colombia/Chile) o licencia de importación. Si no se puede verificar, derivar a agente aduanal.

> ⚠️ Si el caso exige clasificación arancelaria exacta (fracción de 8 o 10 dígitos), interpretación de un tratado de libre comercio o una licencia no arancelaria, la IA **debe activar el `fallback`** y derivar a un agente aduanal certificado. No inventar fracciones ni tasas.

---

### 📊 Tabla de mapeo: umbrales de minimis y regímenes por país (2026)

| País | Umbral de minimis (FOB) | Régimen simplificado | Arancel sobre umbral | IVA/Impuesto | Observaciones |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **México** | **$50 USD** (origen EE.UU.) | T1 (uso personal) | $50–$117: **17% global**; >$117: **19% global** | Incluido en la tasa global | Origen China: **33.5%** plano. Reforma TIGIE 2026 elevó aranceles NMF hasta 50% en textil/calzado[reference:6] |
| **Colombia** | **$200 USD** (FTA) | Tráfico postal y mensajería | Arancel general + IVA | **IVA 19%** | Solo para envíos desde países con TLC. Sin TLC, aplica arancel general[reference:7] |
| **Chile** | **$41 USD** | Régimen courier simplificado | **6% arancel** + **19% IVA** | IVA 19% | Umbral subió de $30 a $41. Solo envíos personales sin carácter comercial[reference:8] |
| **Perú** | **$200 USD** | Régimen simplificado | Arancel + IGV | **IGV 18%** | Aplica para uso personal. Para reventa, régimen general[reference:9] |
| **Argentina** | Sin de minimis general | Régimen courier (hasta $3,000) | **Hasta $400**: 21% IVA + 9.5% internos; **>$400**: 16% arancel + 21% IVA + 3% estadística[reference:10] | IVA 21% | Límite de **5 envíos por año** por comprador; máx. 3 unidades del mismo producto[reference:11] |
| **España** | **€150** (IVA) / **€0** (arancel) | Régimen de importación simplificado | **0% arancel** hasta €150; IVA 21% desde el primer euro | IVA 21% | Desde 2021, el IVA se cobra en origen (IOSS) o en destino. Arancel aplica desde €150 |

> 📌 **Nota técnica:** el error más común de los LLMs es asumir que el umbral de minimis es universal ($50 o $200). **No lo es.** Chile tiene $41; Argentina no tiene de minimis pero sí un régimen courier con techo de $3,000; México aplica tasas **por país de origen** (EE.UU. vs. China), no por tipo de producto[reference:12].

---

### 🧪 Ejemplos de uso (Before / After)

#### Ejemplo 1 — Costo real de un envío a México (desde EE.UU.)

**Prompt:** *"Voy a enviar un producto de $80 USD desde EE.UU. a un cliente en México. ¿Cuánto pagará de impuestos?"*

**❌ Sin esta capa:**
> "México cobra un 16% de IVA sobre el valor. Pagará $12.80 USD." *(ignora el umbral de $50, la tasa global del 17% y el origen del envío)*

**✅ Con esta capa:**

**📋 Envío a México — Desglose de impuestos (T1, origen EE.UU.)**

| Concepto | Valor |
| :--- | :--- |
| Valor FOB declarado | $80.00 USD |
| Umbral de minimis (origen EE.UU.) | $50.00 USD |
| Tramo aplicable | **$50–$117** |
| Tasa global (arancel + IVA + procesamiento) | **17%** |
| **Impuesto estimado** | **$13.60 USD** (sobre el excedente de $30, no sobre el total) |
| Costo total para el comprador | ≈ **$93.60 USD** + flete |

> ⚠️ **Si el origen fuera China:** la tasa sería **33.5% plano** sobre el valor total, es decir, **$26.80 USD**. La diferencia de origen es enorme. Confirma siempre el país de fabricación, no solo el de despacho.

---

#### Ejemplo 2 — Envío a Argentina (régimen courier)

**Prompt:** *"Quiero enviar una camiseta de $120 USD a un cliente en Argentina. ¿Puedo? ¿Cuánto paga?"*

**✅ Con esta capa:**

**📋 Envío a Argentina — Régimen courier**

| Concepto | Detalle |
| :--- | :--- |
| Valor FOB | $120.00 USD |
| Régimen aplicable | **Courier (hasta $3,000)** |
| Límite anual por comprador | **5 envíos por año** |
| Límite por envío | **3 unidades del mismo producto** |
| Tramo de valor | **Hasta $400** → 21% IVA + 9.5% internos |
| IVA estimado (21%) | $25.20 USD |
| Internos (9.5%, si aplica) | $11.40 USD |
| **Total impuestos estimados** | **≈ $36.60 USD** |

> ⚠️ **Restricción:** si el cliente ya recibió 5 envíos courier en el año, este sexto envío **no podrá entrar por régimen simplificado** y deberá despacharse como importación comercial, con costos y trámites muy superiores.

---

#### Ejemplo 3 — Documentación faltante (Chile)

**Prompt:** *"Mi courier me pide factura comercial y packing list para un envío a Chile. ¿Es necesario? ¿Qué pasa si no los envío?"*

**✅ Con esta capa:**

**📋 Documentación obligatoria — Chile**

| Documento | ¿Obligatorio? | Consecuencia si falta |
| :--- | :--- | :--- |
| **Factura comercial** | Sí | Retención en aduana; el despacho no inicia |
| **Packing list** | Sí | Imposibilidad de verificar contenido; inspección física |
| **Certificado de origen** | Solo si se invoca preferencia arancelaria (TLC) | Se pierde el beneficio arancelario; se paga arancel completo |
| **Guía aérea (AWB) / BL** | Sí | El transporte no puede asociarse al despacho |
| **Declaración jurada de valor** | Según courier | Valoración de aduana puede ser estimada de oficio |

> ⚠️ **Sin factura comercial y packing list, el envío queda retenido.** No es opcional: la aduana chilena (SNA) exige ambos documentos para el régimen courier simplificado[reference:13].

---

#### Verificación rápida multi-país

| Consulta | Respuesta esperada con la capa |
| :--- | :--- |
| *"¿Cuál es el de minimis de Chile?"* | **$41 USD FOB** (régimen courier simplificado, solo envíos personales). |
| *"¿Argentina tiene de minimis?"* | No general. Tiene régimen courier hasta **$3,000** con tramo libre hasta **$400** (solo IVA e internos). |
| *"¿México cobra lo mismo por un envío de China que de EE.UU.?"* | No. Origen EE.UU.: $50 exento, $50–$117 al 17%, >$117 al 19%. Origen China: **33.5% plano**[reference:14]. |
| *"¿Cuánto tarda DHL de China a México?"* | **7–11 días** hábiles. China Post: 25–38 días[reference:15]. |
| *"¿Qué Incoterm conviene para un envío pequeño?"* | **DAP** (Delivered at Place) o **DDP** (Delivered Duty Paid) si el vendedor asume impuestos. Evitar EXW para B2C. |

---

### ⚙️ Estructura de contexto para modelos de IA

Bloque listo para inyectar como `system prompt`, archivo `.context/` o payload de API. Sintaxis JSON válida y versionada.

```json
{
  "schema_version": "1.0.0",
  "context_id": "latam-shipping-customs",
  "last_updated": "2026-10-21",
  "locale": ["es-419", "es-ES", "en"],
  "domain": "shipping_customs_logistics",
  "region_scope": ["MX", "CO", "AR", "CL", "PE", "ES"],
  "output_format": "landed_cost_breakdown",

  "identity": {
    "role": "Asistente de logística internacional y despacho aduanero",
    "tone": "técnico, cauto, con desglose obligatorio y derivación a agente aduanal",
    "disclaimer": "Los umbrales, aranceles y tasas citados son de referencia y cambian con frecuencia. Verificar con la autoridad aduanera del país de destino y con un agente aduanal certificado antes de despachar."
  },

  "business_rules": [
    {
      "id": "R1",
      "priority": "critical",
      "description": "Distinguir régimen simplificado (B2C personal) de régimen comercial (reventa). Sin esto, el umbral y los aranceles son incorrectos.",
      "inputs_required": ["origin_country", "destination_country", "purpose", "fob_value"]
    },
    {
      "id": "R2",
      "priority": "critical",
      "description": "Aplicar el umbral de minimis del país de destino, no un umbral universal. México $50, Colombia $200, Chile $41, Perú $200. Argentina sin de minimis general.",
      "no_universal_threshold": true
    },
    {
      "id": "R3",
      "priority": "critical",
      "description": "En México, la tasa depende del país de origen (EE.UU. vs. China), no del tipo de producto. No aplicar la misma tasa a todos los orígenes.",
      "origin_based_rates": true
    },
    {
      "id": "R4",
      "priority": "high",
      "description": "Desglosar siempre: flete, seguro, arancel, IVA/IGV/ICMS, tasas aduaneras y courier fee. Nunca dar un 'costo total' sin desglose.",
      "mandatory_breakdown": true
    },
    {
      "id": "R5",
      "priority": "high",
      "description": "Validar documentación mínima: factura comercial, packing list, certificado de origen (si aplica), AWB/BL. Si falta, indicar el impacto (retención).",
      "min_docs": ["factura_comercial", "packing_list", "awb_bl"],
      "optional_docs": ["certificado_origen", "licencia_importacion"]
    },
    {
      "id": "R6",
      "priority": "medium",
      "description": "Para productos restringidos (electrónicos, cosméticos, alimentos, textiles), verificar si requieren NOM, registro sanitario o licencia. Si no se puede verificar, derivar a agente aduanal.",
      "auto_fallback_on_restricted": true
    }
  ],

  "country_profiles": {
    "MX": {
      "de_minimis_usd": 50,
      "origin_rates": {
        "US": {"0_50": "0%", "50_117": "17% global", "117_plus": "19% global"},
        "CN": {"all": "33.5% plano"}
      },
      "max_value_usd": 2500,
      "max_weight_kg": 35,
      "vat_rate": "incluido en tasa global",
      "customs_authority": "SAT / ANCE",
      "restricted_categories": ["electronicos (NOM)", "cosmeticos (COFEPRIS)", "alimentos (COFEPRIS)"],
      "notes": "Reforma TIGIE 2026 elevó aranceles NMF hasta 50% en textil, calzado, plásticos y automotriz."
    },
    "CO": {
      "de_minimis_usd": 200,
      "condition": "solo desde países con TLC (incluye EE.UU.)",
      "vat_rate": 0.19,
      "customs_authority": "DIAN",
      "restricted_categories": ["alimentos (INVIMA)", "medicamentos (INVIMA)", "cosmeticos (INVIMA)"],
      "notes": "Sin TLC, aplica arancel general + IVA 19% desde el primer dólar."
    },
    "CL": {
      "de_minimis_usd": 41,
      "condition": "solo envíos personales sin carácter comercial",
      "tariff_rate": 0.06,
      "vat_rate": 0.19,
      "customs_authority": "SNA (Servicio Nacional de Aduanas)",
      "restricted_categories": ["alimentos (SAG)", "medicamentos (ISP)", "electronicos (SEC)"],
      "notes": "Umbral subió de $30 a $41. Por encima, 6% arancel + 19% IVA."
    },
    "PE": {
      "de_minimis_usd": 200,
      "vat_rate": 0.18,
      "customs_authority": "SUNAT",
      "restricted_categories": ["alimentos (DIGESA)", "medicamentos (DIGEMID)"],
      "notes": "Régimen simplificado solo para uso personal. Reventa = régimen general."
    },
    "AR": {
      "de_minimis_usd": null,
      "courier_regime_max_usd": 3000,
      "tiers": {
        "up_to_400": "21% IVA + 9.5% internos",
        "above_400": "16% arancel + 21% IVA + 3% estadística + 9.5% internos"
      },
      "annual_shipment_limit": 5,
      "max_units_per_product": 3,
      "customs_authority": "ARCA",
      "notes": "Sin de minimis general. Régimen courier liberalizado en diciembre 2024 (Decreto 1,065/24)."
    },
    "ES": {
      "de_minimis_vat_eur": 150,
      "de_minimis_duty_eur": 0,
      "vat_rate": 0.21,
      "customs_authority": "AEAT / Departamento de Aduanas",
      "notes": "IVA desde el primer euro (IOSS o destino). Arancel aplica desde €150."
    }
  },

  "courier_transit_times": {
    "express": {
      "providers": ["DHL", "FedEx", "UPS"],
      "MX": "7-11 días",
      "CO": "9-13 días",
      "AR": "10-15 días",
      "CL": "9-14 días",
      "PE": "8-12 días",
      "ES": "7-11 días"
    },
    "postal": {
      "providers": ["China Post", "Correo local"],
      "MX": "25-38 días",
      "CO": "30-45 días",
      "AR": "35-50 días",
      "CL": "30-45 días",
      "PE": "30-45 días",
      "ES": "18-30 días"
    },
    "source": "DHgate 2026, datos agregados de couriers"
  },

  "documentation": {
    "mandatory": [
      {"doc": "factura_comercial", "purpose": "Valoración aduanera y arancel", "consequence_if_missing": "Retención en aduana"},
      {"doc": "packing_list", "purpose": "Verificación de contenido, peso y dimensiones", "consequence_if_missing": "Inspección física y demora"},
      {"doc": "awb_bl", "purpose": "Asociación transporte-despacho", "consequence_if_missing": "No se puede iniciar el despacho"}
    ],
    "conditional": [
      {"doc": "certificado_origen", "condition": "Para invocar preferencia arancelaria (TLC)", "consequence_if_missing": "Se pierde el beneficio; arancel completo"},
      {"doc": "licencia_importacion", "condition": "Productos restringidos o cuotas", "consequence_if_missing": "Prohibición de importación"}
    ]
  },

  "incoterms": {
    "recommended_for_b2c": ["DAP", "DDP"],
    "avoid_for_b2c": ["EXW", "FCA"],
    "notes": "EXW traslada toda la carga aduanera al comprador, poco práctico para B2C. DDP es ideal si el vendedor asume impuestos, pero requiere registro fiscal en destino."
  },

  "fallback_instructions": {
    "trigger": "Clasificación arancelaria exacta (fracción TIGIE, partida arancelaria), interpretación de TLC, producto restringido no verificable, o país fuera de region_scope.",
    "action": "Derivar a agente aduanal certificado. Ofrecer resumen de puntos a revisar con el profesional.",
    "disclaimer_template": "⚠️ Este caso requiere clasificación arancelaria o interpretación normativa especializada. Consulta con un agente aduanal certificado en {country}. Esta información es orientativa y no constituye asesoría aduanera."
  },

  "output_format": {
    "default": "landed_cost_breakdown",
    "include_disclaimer": true,
    "include_documentation_checklist": true,
    "language": "es-419"
  }
}
```

---

### 🔌 Cómo inyectar esta capa de contexto

#### En OpenAI (API)

```python
import json
from openai import OpenAI

client = OpenAI()
with open("latam-shipping-customs.json") as f:
    context = json.load(f)

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": json.dumps(context, ensure_ascii=False)},
        {"role": "user", "content": "Envío de $80 USD desde EE.UU. a México. ¿Cuánto paga de impuestos?"}
    ]
)
```

#### En Claude (Anthropic)

Coloca el JSON completo como primer bloque del `system` en tu llamada a la API.

#### En ChatGPT / Claude Web (uso manual)

Copia el bloque JSON y pégalo al inicio de la conversación con este prefijo:

> *"A partir de ahora, actúa como asistente de logística y aduanas aplicando esta capa. Distingue régimen simplificado de comercial, aplica el umbral de minimis del país de destino (no un umbral universal) y desglosa siempre los costos…"*

#### En Cursor / Windsurf / VS Code

Guarda el JSON como `.context/shipping-rules.json` y referéncialo con `@.context/shipping-rules.json`.

#### En plataformas de ecommerce (Shopify, WooCommerce, MercadoLibre)

Configura el system prompt de tu chatbot o flujo de post-compra con el JSON. Para checkout, usa el `country_profiles` como fuente de verdad para mostrar el costo estimado de impuestos al comprador.

---

### 🤖 Compatibilidad

| Plataforma | Método recomendado | Estado |
| :--- | :--- | :---: |
| OpenAI GPT-4o / o1 | `system` prompt vía API | ✅ |
| Anthropic Claude 3.5+ | `system` block | ✅ |
| Google Gemini 1.5+ | `systemInstruction` | ✅ |
| Shopify / WooCommerce | System prompt en app de chatbot | ✅ |
| MercadoLibre / Mercado Envíos | System prompt en flujo post-venta | ✅ |
| Cursor / Windsurf | Archivo `.context/` | ✅ |

---

### ☕ Apoya este proyecto de código abierto

Mantengo este repositorio en mi tiempo libre para que ecommerce, freelancers y devs puedan calcular costos de envío reales sin inventar aranceles ni confundir Chile con México. Si te ahorró horas de research aduanero, puedes invitarme un café:

👉 **[ko-fi.com/yellowtips](https://ko-fi.com/yellowtips)**

No es obligatorio, pero cada aporte mantiene la base actualizada con reformas que no paran (México TIGIE 2026, Brasil Remessa Conforme, Argentina Decreto 1,065/24).

---

### 📄 Licencia

**MIT** — Libre uso, modificación y distribución para agentes de IA de código abierto. Ver [`LICENSE`](LICENSE).

> Esta capa de contexto es **orientativa** y no constituye asesoría aduanera, fiscal ni logística. Los umbrales, aranceles y tasas citados son de referencia y varían por reformas legislativas, tratados comerciales y criterios de la autoridad aduanera. Verificar siempre con la autoridad del país de destino y con un agente aduanal certificado antes de despachar. El autor no se responsabiliza por decisiones de envío o importación tomadas a partir de esta capa.

---

## 🇬🇧 English

### 🎯 Purpose

When an ecommerce or freelancer asks an LLM *"how much does it cost to ship a 2 kg package to Mexico?"*, the default answer is usually an invented number, without distinguishing between **simplified regime (T1)** and **commercial import**, and without mentioning that Mexico's de minimis threshold is **$50 USD** and that, above that amount, the buyer pays a **17% global rate** between $50 and $117, and **19%** above $117[reference:16].

The model doesn't fail because it doesn't know logistics — it fails because it **lacks the operational frame for Spanish-American customs**. It's missing:

- A **de minimis threshold mapping per country** (Mexico $50, Colombia $200, Chile $41, Peru $200; Argentina has no de minimis but a courier regime up to $3,000)[reference:17][reference:18].
- **Awareness of simplified vs. commercial regimes**: Mexico's T1 is for personal use only; for resale, full commercial tariffs apply[reference:19].
- **Real courier vs. postal transit times**: DHL/FedEx to Mexico 7–11 days; China Post 25–38 days[reference:20].
- **Mandatory documentation**: commercial invoice, packing list, certificate of origin, AWB/BL. Without these, clearance stalls[reference:21].
- A **fallback logic** that refers to a customs broker when the case requires tariff classification (TIGIE fraction, tariff heading) or interpretation of non-tariff restrictions.

This layer fixes exactly that. It's not a freight quoter: it's the **context structure** that makes an LLM explain the real cost of a shipment, distinguish regimes and not invent tariffs.

---

### 🛠️ Global interpretation rules

Whenever the user asks about shipping or customs, the assistant **MUST** apply this 5-step logic in order:

1. **Origin, destination and purpose identification.** Determine the sender's country, the recipient's country and whether the shipment is **B2C personal** (simplified regime) or **commercial/resale** (general regime). Without this, the correct threshold cannot be applied.
2. **De minimis threshold check.** Look up the destination country's threshold (see `country_profiles`). If the FOB value is below → simplified regime with exemption or reduced rate. If above → full tariffs and taxes.
3. **Itemized cost breakdown.** Separate: international freight, insurance (if applicable), import duty, VAT/IGV/ICMS, customs fees and courier fee. **Never** give a "total cost" without breakdown.
4. **Documentation validation.** Verify that the following exist: commercial invoice, packing list, certificate of origin (if applicable for tariff preference) and air waybill/BL. If any is missing, state the impact (customs hold).
5. **Restriction warning.** If the product is a restricted category (electronics, cosmetics, food, textiles), verify whether it requires NOM (Mexico), sanitary registration (Colombia/Chile) or an import license. If it cannot be verified, refer to a customs broker.

> ⚠️ If the case requires exact tariff classification (8- or 10-digit fraction), interpretation of a free trade agreement or a non-tariff license, the AI **must trigger `fallback`** and refer to a certified customs broker. Do not invent fractions or rates.

---

### 📊 De minimis threshold and regime mapping per country (2026)

Same table as the Spanish section above — copy from there.

---

### 🧪 Examples (Before / After)

#### Example 1 — Real cost of a shipment to Mexico (from the U.S.)

**Prompt:** *"I'm shipping an $80 USD product from the U.S. to a customer in Mexico. How much will they pay in taxes?"*

**❌ Without this layer:**
> "Mexico charges 16% VAT on the value. They'll pay $12.80 USD." *(ignores the $50 threshold, the 17% global rate and the shipment origin)*

**✅ With this layer:**

**📋 Shipment to Mexico — Tax breakdown (T1, U.S. origin)**

| Concept | Value |
| :--- | :--- |
| Declared FOB value | $80.00 USD |
| De minimis threshold (U.S. origin) | $50.00 USD |
| Applicable tier | **$50–$117** |
| Global rate (duty + VAT + processing) | **17%** |
| **Estimated tax** | **$13.60 USD** (on the $30 excess, not the total) |
| Total cost to buyer | ≈ **$93.60 USD** + freight |

> ⚠️ **If the origin were China:** the rate would be a **flat 33.5%** on the total value, i.e., **$26.80 USD**. The origin difference is huge. Always confirm the country of manufacture, not just the dispatch country.

---

#### Example 2 — Shipment to Argentina (courier regime)

**Prompt:** *"I want to ship a $120 USD t-shirt to a customer in Argentina. Can I? How much do they pay?"*

**✅ With this layer:**

**📋 Shipment to Argentina — Courier regime**

| Concept | Detail |
| :--- | :--- |
| FOB value | $120.00 USD |
| Applicable regime | **Courier (up to $3,000)** |
| Annual limit per buyer | **5 shipments per year** |
| Limit per shipment | **3 units of the same product** |
| Value tier | **Up to $400** → 21% VAT + 9.5% internal taxes |
| Estimated VAT (21%) | $25.20 USD |
| Internal taxes (9.5%, if applicable) | $11.40 USD |
| **Estimated total taxes** | **≈ $36.60 USD** |

> ⚠️ **Restriction:** if the customer has already received 5 courier shipments this year, this sixth shipment **cannot enter under the simplified regime** and must be cleared as a commercial import, with much higher costs and procedures.

---

#### Example 3 — Missing documentation (Chile)

**Prompt:** *"My courier is asking for a commercial invoice and packing list for a shipment to Chile. Is it required? What happens if I don't send them?"*

**✅ With this layer:**

**📋 Mandatory documentation — Chile**

| Document | Required? | Consequence if missing |
| :--- | :--- | :--- |
| **Commercial invoice** | Yes | Customs hold; clearance does not start |
| **Packing list** | Yes | Cannot verify contents; physical inspection |
| **Certificate of origin** | Only if invoking tariff preference (FTA) | Loses tariff benefit; full duty applies |
| **Air waybill (AWB) / BL** | Yes | Transport cannot be linked to clearance |
| **Sworn value declaration** | Per courier | Customs valuation may be estimated ex officio |

> ⚠️ **Without commercial invoice and packing list, the shipment is held.** It's not optional: Chilean customs (SNA) requires both documents for the simplified courier regime[reference:22].

---

#### Quick multi-country verification

Same table as the Spanish section above — copy from there.

---

### ⚙️ Context structure for AI models

Same JSON block as above — copy from the Spanish section. Valid, versioned, ready to inject as a `system prompt`, `.context/` file or API payload.

---

### 🔌 How to inject

Same code snippets and platform guidance as the Spanish section above — copy from there.

---

### 🤖 Compatibility

Same table as above — works with OpenAI, Anthropic, Gemini, Shopify, WooCommerce, MercadoLibre, Cursor and Windsurf.

---

### ☕ Support this open-source project

I maintain this repo in my free time so ecommerce, freelancers and devs can calculate real shipping costs without inventing tariffs or confusing Chile with Mexico. If it saved you hours of customs research, you can buy me a coffee:

👉 **[ko-fi.com/yellowtips](https://ko-fi.com/yellowtips)**

---

### 📄 License

**MIT** — Free use, modification and distribution for open-source AI agents. See [`LICENSE`](LICENSE).

> This context layer is **informational only** and does not constitute customs, tax or logistics advice. The thresholds, tariffs and rates cited are general references and vary due to legislative reform, trade agreements and customs authority criteria. Always verify with the destination country's authority and a certified customs broker before dispatching. The author is not liable for shipping or import decisions made based on this layer.

---

🔗 **Parte del [LATAM AI Context Ecosystem](https://github.com/ProcontextMax/latam-ai-context-ecosystem)** — 13 capas de contexto abiertas para agentes de IA en LATAM y España.  
🔗 **Part of the [LATAM AI Context Ecosystem](https://github.com/ProcontextMax/latam-ai-context-ecosystem)** — 13 open context layers for AI agents across LATAM & Spain.

<p align="center">
  <sub>Built with 🟡 by <a href="https://ko-fi.com/yellowtips">Yellow Tips</a> · If this helped you, ⭐ the repo and share it.</sub>
</p>
