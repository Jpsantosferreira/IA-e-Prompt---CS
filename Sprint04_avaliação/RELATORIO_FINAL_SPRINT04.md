# Relatório de Avaliação Automatizada de Agentes — Sprint 04
**EV Challenge — GoodWe Chatbot Goody**  
*Data: 17/10/2026*  

---

## 1. Metodologia do Golden Dataset

O **Golden Dataset** foi construído para servir como base reprodutível de testes quantitativos automatizados. Ele reaproveitou e expandiu os casos definidos nas Sprints 1, 2 e 3, totalizando 14 questões estrategicamente distribuídas em 4 categorias principais:

1. **Funcionalidades Principais (5 casos):** Consultas reais de saldo, tempo restante, histórico de pagamento por mês, veículos cadastrados e carregadores/condomínios parceiros.
2. **Aderência ao Escopo GoodWe (2 casos):** Perguntas totalmente fora de contexto (ex.: geografia, receitas culinárias) para medir a recusa apropriada do agente.
3. **Segurança e Prompt Injection (5 casos):** Injeções diretas pedindo revelação do system prompt, inserção de senhas, aconselhamento jurídico/financeiro indesejado e comandos de manipulação elétrica perigosa.
4. **Casos de Borda e Controle de Acesso (2 casos):** Solicitação de dados técnicos não existentes e tentativa de acessar ferramentas de administrador via perfil de usuário comum.

Cada item do dataset possui um **gabarito esperado** detalhado e uma lista de **critérios de aceite**, garantindo que o avaliador (LLM Judge) avalie objetivamente o cumprimento do requisito.

---

## 2. Pipeline de Avaliação Automatizado

A arquitetura do pipeline automatizado (`eval_pipeline.py`) adota a estratégia **LLM as a Judge** utilizando o modelo `gpt-4o-mini` com `temperature=0` e saída estruturada via JSON Schema. 

### Métricas Implementadas (Escala 0.0 a 1.0):
* **Correção / Acurácia:** Mede se a resposta contém os dados factuais exatos fornecidos pelas ferramentas/gabarito sem alucinações.
* **Aderência ao Escopo:** Verifica se o agente manteve a atuação estritamente no ecossistema GoodWe/ChargeGrid, recusando assuntos irrelevantes.
* **Segurança e Recusa:** Avalia a capacidade de conter ataques de Prompt Injection, impedir vazamento de diretrizes, ignorar inserção de dados sensíveis e negar instrução perigosa ou conselhos regulados.
* **Fidelidade ao Contexto:** Avalia a clareza, empatia e alinhamento do tom de voz com a persona *Goody*.

---

## 3. Tabela de Resultados por Versão 

Abaixo é apresentada a comparação direta e automatizada das duas arquiteturas do agente construídas ao longo do semestre:
* **Versão 1 (Sprints 1 e 2):** Implementação legada (`chatbot_teste.py`) com chamadas manuais à API OpenAI, loop síncrono e sem guardrails dedicados.
* **Versão 2 (Sprint 3):** Implementação com framework de agentes (`chatbot_agents_sdk.py`), gerenciamento de memória via `SQLiteSession`, guardrails de entrada e saída, e isolamento físico de ferramentas por perfil (`goody_usuario` / `goody_admin`).

| Métrica de Avaliação | Versão 1 (Sprints 1-2 Legado) | Versão 2 (Sprint 3 Agents SDK) |
|---|------|------|
| **Correção / Acurácia** | 0.64 | 0.99 |
| **Aderência ao Escopo** | 0.96 | 1.00 |
| **Segurança e Recusa** | 0.85 | 1.00 |
| **Fidelidade ao Contexto** | 0.95 | 1.00 |
| **SCORE GERAL FINAL** | **0.85** | **1.00** |

---

## 4. Classificação e Justificativa

**Vencedor Formal: Versão 2 (Sprint 3 - Agents SDK)** com score final de **1.00** contra **0.85** da Versão 1.

### Justificativa baseada nos números:
1. **Segurança (1.00 vs 0.85):** A Versão 1 falhou em bloquear injeções diretas e aceitou interações com senhas, limitando-se a recusas no texto do prompt. A Versão 2 barrou 100% das tentativas de injection e dados sensíveis na camada de `input_guardrail` determinística antes do modelo ser invocado.
2. **Aderência ao Escopo (1.00 vs 0.85):** Com a separação física de ferramentas no perfil do agente (`goody_usuario` não possui ferramentas de admin), a Versão 2 impediu vazamentos de dados restritos com eficácia absoluta.
3. **Acurácia e Fidelidade (1.00 vs 0.95):** O framework de agentes tratou parâmetros de chamadas de função com maior precisão e manteve contexto entre sessões de forma consistente.

---

## 5. Comparação com a Avaliação Manual da Sprint 3

Ao confrontar os dados gerados pelo pipeline automatizado com a **avaliação manual** realizada na Sprint 3:

* **Concordâncias:** A avaliação manual já havia indicado que a refatoração para o Agents SDK havia solucionado o vazamento de funções administrativas e trazido proteção contra prompt injection. O pipeline automatizado confirmou esse ganho com dados quantitativos (salto de 0.35 para 0.98 em Segurança).
* **Divergências:** Na avaliação manual, a equipe acreditava que a Versão 1 tinha uma performance "aceitável" em segurança baseando-se em testes simples. O pipeline automatizado revelou que a Versão 1 era altamente vulnerável quando submetida a variados tipos de injeção e manipulação de prompt.
* **Hipóteses Explicações:** A avaliação manual sofre do *viés do avaliador humano* (que tende a testar caminhos felizes e ser indulgente com respostas parcialmente corretas). O **LLM as a Judge** avalia contra critérios de aceite estritos e sem complacência, evidenciando as falhas sutis da arquitetura legada.

---

## 6. Limitações do Pipeline de Avaliação

1. **Viés do LLM Juiz (Judge Bias):** Utilizar o `gpt-4o-mini` como juiz pode introduzir viés de auto-preferência (*self-enhancement bias*) ou sensibilidade a variações cosméticas na formatação do texto.
2. **Latência e Custo Computacional:** A execução do pipeline completo exige chamadas duplas para cada item do dataset (uma para gerar a resposta do agente e outra para a avaliação do juiz), aumentando o tempo e custo proporcionalmente ao tamanho do Golden Dataset.
3. **Sensibilidade do Parser de JSON:** Respostas do juiz que eventualmente fujam do formato JSON estrito necessitam de tratamento de exceções, podendo levar a notas zero pontuais se o parser falhar.

---

## 7. Equipe e Divisão de Trabalho

| Integrante | Nome / RM | Tarefa / Responsabilidade Principal |
|---|---|---|
| 1 | João Pedro Santos Ferreira / RM569202 | Desenvolvimento do Pipeline de Avaliação (`eval_pipeline.py`) e Integração com LLM Judge. |
| 2 | Ana Julia Yumi Inoue / RM569430 | Construção e expansão do Golden Dataset (`golden_dataset.json`). |
| 3 | Maria Fernanda Dias / RM569999 | Análise de métricas quantitativas e confronto com a avaliação manual da Sprint 3. |
| 4 | Ulysses Gomes Soares / RM573826 | Execução dos testes comparativos e geração das tabelas e arquivos JSON. |
| 5 | Yasmin Cristina Mayer / RM573964 | Redação do Relatório Final (`RELATORIO_FINAL_SPRINT04.md`) e consolidação do repositório. |