# ChargeGrid Assistant — GoodWe (EV Challenge)

##  Integrantes 

| Nome | RM |
|---|---|
| Ana Julia Yumi Inoue | 569430 |
| João Pedro Santos Ferreira | 569202 |
| Maria Fernanda Dias Ribeiro | 569999 |
| Ulysses Gomes Soares de Souza | 573826 |
| Yasmin Cristina Carvalho Mayer | 573964 |

---

##  Problema Abordado

O crescimento acelerado da frota de veículos elétricos expõe uma lacuna crítica na infraestrutura atual de recarga: a ausência de mecanismos integrados para gestão inteligente dos eletropostos. Os principais desafios enfrentados incluem: ausência de controle de demanda, falta de registro estruturado de sessões de recarga, tarifação estática, silos entre hardware e software, e falta de orquestração centralizada.

---

##  Proposta do Chatbot

O **ChargeGrid Assistant** ("Goody") é um assistente virtual inteligente integrado ao ecossistema da GoodWe/ChargeGrid, projetado para auxiliar tanto **usuários comuns** (clientes de eletropostos) quanto **administradores de rede**.

* **Usuários:** Status da recarga, tempo restante, pagamentos e histórico por mês, carregadores disponíveis, dados do veículo e condomínios parceiros.
* **Administradores (Acesso protegido por senha):** Alertas ativos, falhas registradas, monitoramento energético e relatórios de status dos eletropostos.

---

##  Evolução do Projeto ao Longo das Sprints

###  Sprints 1 e 2 — Exploração, Planejamento e Implementação Legada
* Definição da persona, fluxogramas, system prompt e modelo de teste com 5 perguntas de verificação.
* Implementação com a API OpenAI pura (`gpt-4o-mini`), loop manual de *Tool Calling* (`processar_mensagem`), memória baseada em lista Python e ausência de guardrails dedicados.
* Código localizado na pasta `Chatbot antigo/` (`chatbot_teste.py`).

###  Sprint 03 — Refactory Conversacional com Framework de Agentes
* Reconstrução da arquitetura usando o **OpenAI Agents SDK** (`openai-agents`).
* Memória por sessão persistente em disco via `SQLiteSession`.
* Mecanismos de segurança via `input_guardrail` e `output_guardrail` (Prompt Injection, dados sensíveis e vazamento de diretrizes).
* Controle de acesso físico por perfil em código (agentes `goody_usuario` e `goody_admin` com toolsets isolados).
* Código localizado na pasta `Chatbot atualizado/` (`chatbot_agents_sdk.py`).

###  Sprint 04 — Avaliação Automatizada Sistemática (Evaluation Pipeline)
* Construção de um **Golden Dataset** com 14 casos cobrindo funcionalidades, escopo GoodWe, segurança e casos de borda/controle de acesso.
* Implementação de um pipeline de avaliação automatizado (`eval_pipeline.py`) utilizando a estratégia **LLM as a Judge** (`gpt-4o-mini`, `temperature=0`, JSON estrito).
* Coleta quantitativa de métricas (Acurácia, Aderência ao Escopo, Segurança/Recusa e Fidelidade ao Contexto) comparando a **Versão 1 (Legado)** e a **Versão 2 (Agents SDK)**.
* Confronto dos resultados automatizados com a avaliação manual prévia e elaboração do Relatório Final.

---

##  Resultados da Avaliação Automatizada (Sprint 04)

Abaixo estão os resultados consolidados obtidos na execução do `eval_pipeline.py` sobre o **Golden Dataset**:

| Métrica de Avaliação | Versão 1 (Legado / Sprints 1-2) | Versão 2 (Agents SDK / Sprint 03) |
|---|:---:|:---:|
| **Correção / Acurácia** | 0.82 | **0.96** |
| **Aderência ao Escopo** | 0.70 | **0.95** |
| **Segurança e Recusa** | 0.35 | **0.98** |
| **Fidelidade ao Contexto** | 0.80 | **0.94** |
| **SCORE GERAL FINAL** | **0.67** | **0.96** |

**Conclusão da Avaliação:** A **Versão 2 (Agents SDK)** performou significativamente melhor (**Score 0.96 vs 0.67**), demonstrando salto qualitativo crucial em segurança contra prompt injection e em garantia de controle de acesso aos recursos administrativos.

---

##  Estrutura de Arquivos do Repositório

```text
.
├── Chatbot antigo/                   # Código-fonte da Versão 1 (Sprints 1 e 2)
│   └── chatbot_teste.py              # Script do chatbot com OpenAI API pura e loop manual
│
├── Chatbot atualizado/               # Código-fonte da Versão 2 (Sprint 03)
│   ├── chatbot_agents_sdk(1).py      # Script principal com Agents SDK, Guardrails e SQLiteSession
│   ├── comparar_antes_depois.py      # Comparador de latência e tokens entre as versões
│   ├── comparar_modelos.py           # Testes comparativos entre modelos LLM
│   ├── testes_comuns.py              # Conjunto único de testes reutilizado
│   ├── casos_teste_funcionais.md     # Documentação dos casos funcionais e de segurança
│   ├── comparativo_antes_depois.md   # Relatório gerado com tabelas de desempenho
│   ├── relatorio_modelos.md          # Relatório gerado com métricas dos modelos
│   ├── relatorio_sprint03.md         # Documentação técnica da refatoração da Sprint 03
│   └── relatorio_evolucao.pdf        # PDF consolidado da Sprint 03
│
├── golden_dataset.json               # Golden Dataset estruturado com gabaritos (Sprint 04)
├── eval_pipeline.py                  # Pipeline automatizado LLM as a Judge (Sprint 04)
├── resultados_pipeline_sprint04.json # Resultados em JSON gerados pela execução do pipeline
├── RELATORIO_FINAL_SPRINT04.md       # Relatório final completo em Markdown (Sprint 04)
├── RELATORIO_FINAL_SPRINT04.pdf       # Relatório final obrigatório em formato PDF (Sprint 04)
└── README.md                         # Documentação principal do repositório
