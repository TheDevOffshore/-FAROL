# 🟠 FAROL — Agente de IA para Análise de Operações RPAS Offshore

> Pergunte em português. O agente escolhe a ferramenta de análise certa, o Pandas calcula sobre dados reais, e a resposta vem com números exatos — **zero alucinação numérica**.

![Python](https://img.shields.io/badge/Python-3.10+-0d3641?logo=python&logoColor=ff7a1a)
![FastAPI](https://img.shields.io/badge/FastAPI-API-0d3641?logo=fastapi&logoColor=ff7a1a)
![Claude](https://img.shields.io/badge/Claude-tool--calling-0d3641?logoColor=ff7a1a)
![Status](https://img.shields.io/badge/demo-funciona_sem_API_key-3ddc97)

## O problema de negócio

Operações de drones em plataformas de petróleo geram dados o tempo todo — logs de voo, meteorologia, manutenção de frota — mas quem toma decisão (coordenador de operação, fiscal de contrato) não escreve SQL nem abre notebook. A pergunta "por que nossa taxa de aborto subiu na P-74?" leva horas para ser respondida manualmente.

O FAROL responde em segundos, em linguagem natural, citando os números reais.

## Por que isso não é "só um chatbot"

A arquitetura separa **raciocínio** de **cálculo**:

```
Pergunta em PT-BR
      │
      ▼
┌─────────────┐   escolhe a ferramenta   ┌──────────────────┐
│  LLM (Claude)│ ───────────────────────▶│  Engine Pandas    │
│ tool-calling │ ◀─────────────────────── │  (analytics.py)   │
└─────────────┘   números determinísticos └──────────────────┘
      │                                          │
      ▼                                          ▼
 Resposta redigida                      CSVs: voos, meteo,
 citando os números                     manutenção (18 meses)
```

O LLM **nunca calcula** — ele decide *qual* análise rodar e redige o resultado. Todo número vem do Pandas. Se a API key não estiver configurada, um roteador local por palavras-chave assume e a demo continua 100% funcional (modo offline).

## Stack

| Camada | Tecnologia | Papel |
|---|---|---|
| API | FastAPI + Pydantic | Endpoints REST + validação |
| Agente | Anthropic API (tool use) | Roteamento semântico + redação |
| Análise | Pandas | 6 ferramentas determinísticas |
| Dados | Gerador sintético próprio | 3.100+ voos com física realista (sazonalidade de vento, degradação de bateria por ciclo, correlação rajada × aborto) |
| Front | HTML + Chart.js (vanilla) | Chat + dashboard, zero build step |

## Rodando

```bash
cd backend
pip install -r requirements.txt
python data_generator.py        # gera os dados em /data
uvicorn main:app --reload       # abre http://localhost:8000
```

**Modo LLM (opcional):** exporte `ANTHROPIC_API_KEY` antes de subir o servidor. Sem a chave, o sistema roda em modo demo offline.

Documentação interativa da API: `http://localhost:8000/docs`

## Exemplos de perguntas

- "Compare as plataformas" → ranking de confiabilidade
- "Por que as missões são abortadas na P-74?" → causas + curva aborto × rajada
- "Como está a saúde da frota?" → bateria, ciclos, custo e downtime por aeronave
- "Qual o pior mês de vento na bacia de Santos?" → série meteorológica

## Decisões de engenharia

1. **Tool-calling em vez de RAG puro** — para dados tabulares, recuperar trechos de texto é frágil; executar funções determinísticas é auditável.
2. **Fallback offline** — demo nunca quebra em entrevista por falta de chave/quota; degradação elegante com `try/except` no agente.
3. **Dados sintéticos com física do domínio** — taxa de aborto sobe com rajada > 22 kt (limite operacional real de RPAS offshore), bateria degrada ~0,045%/ciclo. O dataset conta uma história verdadeira.
4. **Sem framework de front** — um arquivo HTML deixa o foco da avaliação no backend e no agente.

## Roadmap

- [ ] Ingestão de logs reais DJI (AirData CSV)
- [ ] Memória de conversa (follow-up: "e na PCH-1?")
- [ ] Deploy público (Render) + CI com pytest
- [ ] Exportação de relatório PDF da análise

---

**Autor:** William Oliveira — piloto RPAS offshore sênior (ANAC/DECEA/NR-34) em transição para dados & IA.
GitHub: [@TheDevOffshore](https://github.com/TheDevOffshore) · LinkedIn: [wmdevoffshore](https://linkedin.com/in/wmdevoffshore)

*Dados 100% sintéticos. Nenhuma informação operacional real de qualquer empresa foi utilizada.*
