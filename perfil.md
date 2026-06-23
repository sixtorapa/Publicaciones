# Perfil de Sixto Ramírez Parras — material de referencia para "Publicaciones"

> Este fichero alimenta al agente programado que propone ideas de post.
> Amplíalo cuando quieras (pega texto de tu LinkedIn, anécdotas nuevas,
> proyectos nuevos) — el agente siempre lee la versión más reciente.

## Quién es

- Formación: B.Sc. Economía (Granada / University College Dublin, 7 matrículas
  de honor) + M.Sc. Economía Industrial (Carlos III) + M.Sc. Data Science,
  Big Data & Business Analytics (Complutense).
- No viene de informática pura — perfil analítico/cuantitativo (econometría,
  pricing, forecasting) que migró a ingeniería de IA (RAG, agentes, LLMs).
  Ese arco — de analista de pricing a AI Engineer construyendo sistemas
  multiagente en producción — es en sí mismo un buen hilo narrativo para posts.
- Actualmente **AI Engineer en UPartnerMedia** (desde mayo 2025, Madrid).
  5+ años construyendo sistemas de datos/IA en producción en pharma, media
  y entretenimiento.
- En proceso de preparación para entrevistas técnicas de AI Engineering.
- Inglés C1, español nativo.
- LinkedIn: https://www.linkedin.com/in/sixto-ramirez-parras/
  (no accesible por API/scraping — ampliar este fichero a mano si se quiere
  usar contenido real del perfil).
- GitHub: https://github.com/sixtorapa

## Trayectoria profesional

- **AI Engineer — UPartnerMedia** (mayo 2025 – presente, Madrid). Sistema RAG
  multiagente en producción con un AgentRouter propio (LangChain, ChromaDB,
  Gemini) que orquesta agentes especializados (DocumentQA, SQL, Summary,
  WebSearch); retrieval híbrido BM25 + vector (RRF 55/45) con reranking
  FlashRank. API REST en Flask con auth multiusuario, gestión de sesiones y
  observabilidad con LangSmith; Docker + CI/CD (GitHub Actions → Railway) con
  despliegues sin downtime. Pipeline de retrieval en dos pasadas con chunking
  semántico, enriquecimiento de metadata por LLM y una capa de ReasoningAgent
  para formato de salida "executive grade". Workflows agénticos conectando
  LLMs con fuentes de datos estructuradas (SQLite/Redshift, Excel, vector
  stores) vía tool-calling y memoria conversacional.
- **Data Scientist — Keesing Media Group** (feb 2020 – may 2025, Madrid).
  Pipelines de forecasting y planificación con XGBoost y Prophet: +20% de
  precisión, -10% de desperdicio de impresión vía predicción automatizada de
  demanda. Segmentación de clientes (K-Means, DBSCAN) y analítica predictiva
  para retención. Pipelines de datos y dashboards para operaciones,
  presupuesto y planificación de ventas.
- **Revenue Analyst — Stage Entertainment** (dic 2018 – ago 2019, Madrid).
  Modelos de pricing dinámico y yield management: +20% de ingresos, +10% de
  conversión vía A/B testing y modelado estadístico. Dashboards de pricing y
  forecasting de ingresos automatizado — su primera incursión en sistemas de
  decisión basados en datos.
- **Business Consultant — Solchaga Recio & Asociados** (oct 2017 – may 2018,
  Madrid). Proyecto de analítica inmobiliaria para un banco grande: definición
  de KPIs y dashboards para apoyar decisiones de inversión.

## Proyecto 1 — RAG_HR_ASSISTANT

Asistente RAG de RRHH en producción (Flask + LangChain + ChromaDB + OpenAI),
desplegado en Railway con Postgres. En el CV aparece como "HR Knowledge Base
Assistant" — demo: raghrassistant-production.up.railway.app — repo:
github.com/sixtorapa/RAG_HR_ASSISTANT. Puntos fuertes para contar:

- Ingesta híbrida: loader de PDF que nunca descarta páginas, OCR selectivo
  (solo si hay señal visual + texto pobre), extracción de tablas a Markdown.
- Chunking parent-child: macro-chunks por página (deterministas) + micro-chunks
  hijos para precisión de retrieval, con enriquecimiento LLM (headline+summary)
  que nunca toca el texto original.
- Retrieval híbrido BM25 + vector (MMR), con prefiltro NATIVO por metadata en
  Chroma (no post-filtrado en Python) y two-pass retrieval para evitar mezclar
  documentos en preguntas genéricas.
- Guardarriles de seguridad estilo banca: RBAC por departamento (fail-closed),
  DLP de entrada determinista (IBAN/tarjeta/DNI por checksum matemático, sin
  ML), columnas SQL sensibles bloqueadas, defensa en profundidad contra SQL
  injection del propio LLM (solo SELECT, conexión read-only).
- Router multi-agente: heurísticas deterministas + LLM function-calling,
  agentes intermedios + agente de razonamiento final.
- Evaluación con RAGAS comparando 3 configuraciones — hallazgo real: el
  reranker EMPEORÓ las métricas y daba OOM en Railway, así que se dejó opt-in
  en vez de en el pipeline por defecto (medir, no asumir).
- Incidente real en producción: crash loop por un `set -e` mal puesto en el
  script de arranque, diagnosticado por logs, reproducido en local y arreglado
  antes de redeploy.

## Proyecto 2 — aviation-rag-service-improved

RAG sobre manuales de aviación (Boeing/Airbus) + menús de vuelo, hecho para
una evaluación técnica (Overwatch AI). Resultado: 1.000 de precisión/recall/F1
en retrieval a nivel de página sobre el set de calibración.

- BM25 (40%) + vector (60%) justificado por tipo de contenido (tablas técnicas
  densas vs. prosa de menús/alérgenos).
- Reranking con cross-encoder + expansión de contexto ±1 página para
  información partida entre páginas consecutivas.
- En vez de OCR para diagramas: páginas renderizadas como imagen enviadas a un
  LLM multimodal (Gemini Vision) que interpreta relaciones visuales (flechas,
  colores, asignación de roles) que OCR no puede representar — con el cálculo
  movido a tiempo de ingesta (coste fijo una vez) en vez de tiempo de query.
- Modelo de citación por "evidencia reportada por el LLM": solo se citan
  páginas que el propio modelo dice haber usado como evidencia, no todas las
  recuperadas — reduce citas infladas.

## Proyecto 3 — Bid Intelligence Agent

Sistema multi-agente con LangGraph que automatiza el análisis de licitaciones
públicas españolas (pliegos) para empresas de arquitectura/ingeniería.

- Grafo con estado compartido (`BidState`), edges condicionales y un bucle de
  revisión real Writer↔Reviewer (máx. 3 iteraciones) — no un router lineal.
- Salidas tipadas con Pydantic en cada nodo (Requisito, ScoringResult,
  TechnicalProposal, ReviewFeedback), porque el propio código (no una persona)
  tiene que ramificarse según la respuesta del LLM (GO/NO-GO, aprobado/no).
- Decisión de ingeniería destacable: el agente de "research" se deja
  deliberadamente como stub (placeholder honesto) en vez de darle un GPT-4o
  sin acceso a búsqueda real — porque alucinaría datos de mercado con
  apariencia de reales, que es peor que un stub obviamente falso.
- Roadmap futuro: fine-tuning (QLoRA + LLaMA 3.2 en Modal) para el extractor,
  justificado por ROI (lenguaje muy repetitivo/específico del dominio legal).

## Proyecto 4 — AI-Powered CRM System (Sandoz, Trabajo de Fin de Máster)

Pipeline de CRM con IA para automatizar el reporting de visitas comerciales
en farma. Carpeta local: `CRM-AI-SANDOZ` (repo: CRM_AI_Sandoz, demo en
HuggingFace Spaces).

- Whisper (voz a texto) + Gemini (extracción de entidades, resumen,
  validación de integridad) + FAISS (RAG sobre histórico de visitas) para
  generar el reporte de la visita comercial de forma automática.
- Workflow agéntico multi-paso con actualización dinámica del vectorstore,
  filtrado semántico y un "5-point integrity checker" que verifica que el
  reporte esté completo antes de registrarlo.
- App en producción con Gradio: auth por sesión, dashboards interactivos
  (Matplotlib, Plotly) y analítica de ventas.

## Proyecto 5 — AI-Powered Triage System (Hospital Clínico San Carlos & Microsoft)

Finalista en el Health AI Hackathon. Sistema de triaje inteligente
combinando NLP, procesamiento de datos clínicos y algoritmos de soporte a
la decisión.

- Pipeline de priorización de pacientes en tiempo real, integrando análisis
  de síntomas y modelos de risk scoring.
- Backend en FastAPI + dashboard interactivo para soporte a la decisión
  clínica.

## Educación

- M.Sc. Data Science, Big Data & Business Analytics — Universidad
  Complutense de Madrid (feb 2023 – feb 2024). Enfoque en ML, deep learning,
  NLP, LLMs, optimización e ingeniería de IA.
- M.Sc. Economía Industrial — Universidad Carlos III de Madrid (sept 2015 –
  jun 2017). Econometría, inferencia causal, modelado cuantitativo.
- B.Sc. Economía — Universidad de Granada / University College Dublin
  (sept 2011 – jun 2015). Graduado con siete matrículas de honor.

## Stack técnico

- IA/ML: LangChain, FAISS, Gemini API, XGBoost, Prophet, Scikit-learn,
  PyTorch, HuggingFace.
- Ingeniería/despliegue: Python, FastAPI, Flask, Streamlit, Gradio, GCP,
  Docker, Git.
- Datos/analítica: Pandas, SQL, Spark, Power BI, Tableau.

## Tono e intención de los posts

- Profesional pero directo, sin venderse de forma artificial.
- Preferir anécdotas concretas con cifras/decisiones reales sobre
  generalidades ("hice RAG") — el detalle es lo que distingue.
- Vale combinar: lecciones técnicas, comparativas entre proyectos, el arco de
  carrera (pricing/analítica de negocio → AI Engineer), decisiones de
  ingeniería defendibles, preparación para entrevistas de AI Engineering.
