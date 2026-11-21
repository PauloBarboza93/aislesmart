# 07-Machine Learning

> **Status:** rascunho v0.3 (21/11/2026): M02 decidida (seção 4.1): a entrada tem a intenção anterior e a mensagem. v0.2 (20/11/2026): M01, o modelo classifica a intenção (seção 3). As seções 4 a 9 serão preenchidas uma por vez.
> **Autor:** Paulo Oliveira
> **Depende de:** 01-PRD (Parte B), 05-Ferramentas MCP (filtros da `buscarProdutos`) e 06-Agentes (intenções e as 20 perguntas de teste).

## 1. Objetivo

Definir a **Parte B** do AisleSmart: o trabalho de ML (Machine Learning, aprendizado de máquina) em Python que mostra, com números, se um modelo pequeno ajustado resolve uma tarefa do assistente melhor do que as alternativas mais simples.

É a parte do portfólio que atende ao que a vaga pede além do código da loja: implementação de ML no GitHub, modelo e dataset publicados no Hugging Face e uma pequena pesquisa comparativa.

## 2. Visão geral

```
1. Dataset          perguntas de clientes em português + resposta esperada
      │
      ▼
2. Baseline         modelo simples com scikit-learn (régua de comparação)
      │
      ▼
3. Fine-tuning      LLM pequeno (~4B) ajustado com QLoRA no Unsloth
      │
      ▼
4. Avaliação        baseline × modelo ajustado × modelo sem ajuste
      │              (taxa de acerto e tempo de resposta)
      ▼
5. Publicação       dataset e modelo (GGUF) no Hugging Face → roda no Ollama
```

> **Para iniciante:** o **baseline** (modelo de referência) é o jeito mais simples de resolver a tarefa. Ele existe para responder uma pergunta: "o fine-tuning (ajuste fino) melhorou de verdade, ou um modelo simples já resolvia?". A **QLoRA** (Quantized Low-Rank Adaptation, adaptação de baixo posto quantizada) comprime o modelo e treina só uma pequena parte dele, por isso cabe nos 8 GB de VRAM (memória da placa de vídeo) da RTX 4060. O **GGUF** é o formato de arquivo de modelo que o Ollama usa.

## 3. A tarefa do modelo

*Decidido em 20/11/2026 (M01).*

**Tarefa:** classificar a intenção da mensagem do cliente em uma das 4 intenções do roteador (06, A01). É a mesma tarefa que o roteador já faz com o `qwen3:8b`.

```
"aceita troca?"          → POLITICAS
"coloca 2 no carrinho"   → PEDIDOS
"quero café até R$40"    → PRODUTOS
"oi"                     → FORA_DO_TEMA
```

**Quem disputa** (detalhes na seção 7):

| Concorrente | O que é |
| --- | --- |
| Baseline | Classificador de texto do scikit-learn (ponto de partida: TF-IDF, *Term Frequency–Inverse Document Frequency*, que transforma o texto em números pelas palavras importantes, com regressão logística; confirmar na seção 5) |
| Modelo ~4B sem ajuste | O mesmo modelo base, só com instrução, para medir o ganho do ajuste |
| Modelo ~4B ajustado | O modelo base depois do fine-tuning com QLoRA |
| `qwen3:8b` | O modelo que o roteador usa hoje (06) |

**A pergunta da pesquisa:** *"Para rotear as mensagens de uma loja, um LLM pequeno ajustado vale a pena, ou um modelo clássico já resolve?"*

**Por que esta opção:**

- A comparação é justa: todos respondem a mesma coisa (uma das 4 palavras), e a medida é simples (acertou ou errou, e quanto tempo levou).
- Mantém o scikit-learn, como o PRD previa, sem trocar tecnologia.
- Encaixa no roteador sem mudar o código do 06: o método `interpretar` (A03) continua igual; muda só o nome do modelo no Ollama.
- O dataset é simples (pergunta + intenção), e as 20 perguntas de teste do 06 (seção 8) servem de começo.

**Por que não extrair filtros em JSON (opções descartadas):** o scikit-learn não gera JSON, então o baseline teria de ser feito com regras (outra tecnologia); e o agente de produtos já extrai os filtros quando chama a `buscarProdutos` (06, A05), então o modelo ajustado faria um trabalho repetido e exigiria uma etapa nova antes do agente. Fazer as duas tarefas somaria o trabalho das duas.

**Resultado esperado, com honestidade:** em tarefas de classificação com poucas classes, modelos clássicos costumam ficar perto dos LLMs, e às vezes na frente, respondendo muito mais rápido. Qualquer resultado serve à pesquisa, porque a conclusão vem da medição, não de uma suposição.

> **Para a entrevista:** *"Medi se um LLM ajustado vale a pena para rotear mensagens, comparando com um classificador clássico. A escolha final veio dos números: acerto e tempo de resposta."*

**Documentos alinhados com a M01 (20/11/2026):** 01-PRD (HU-10 e seções 4 e 6), 02-Arquitetura (seções 3, 5, 8, 9 e 10), 04-Contratos de API (seção 9) e 05-Ferramentas MCP (seções 3, 4.1 e 8). Antes, eles diziam que o modelo ajustado extrairia filtros.

## 4. Dataset

Perguntas de clientes em **português**, cada uma com a **intenção esperada** (M01).

### 4.1 Formato de cada exemplo

*Decidido em 21/11/2026 (M02).*

Cada exemplo tem **3 colunas**:

| Coluna | O que é | Exemplo |
| --- | --- | --- |
| `intencao_anterior` | A intenção da mensagem anterior da conversa; `NENHUMA` na primeira mensagem | `PRODUTOS` |
| `mensagem` | O que o cliente escreveu | `e o segundo?` |
| `intencao` | A resposta esperada: `PRODUTOS`, `POLITICAS`, `PEDIDOS` ou `FORA_DO_TEMA` | `PRODUTOS` |

**Por que a intenção anterior:** é a mesma entrada que o roteador recebe no 06 (A10). Sem ela, continuações como "e o segundo?" não têm assunto, e o modelo erraria justamente onde o roteador acerta. Os quatro concorrentes recebem o mesmo texto; no scikit-learn, a intenção anterior entra como mais uma palavra do texto. Assim, a pesquisa mede a tarefa real e o 06 não muda.

**Alternativa descartada:** só a mensagem (2 colunas). Mais simples, mas o modelo erraria as continuações e o A10 teria de ser resolvido de outro jeito na loja (retrabalho no 06).

> **Para o dataset:** é preciso ter exemplos de continuação (mesma intenção da anterior) e de troca de assunto (intenção diferente da anterior), para o modelo não aprender só a repetir a intenção anterior.

### 4.2 Tamanho, origem e divisão

*A preencher (M03).* A definir: quantidade, como gerar e revisar as perguntas, equilíbrio entre as 4 intenções, divisão treino / validação / teste e licença de publicação.

## 5. Baseline

*A preencher.* Já decidido no PRD: classificador de intenção com **scikit-learn**. Ponto de partida a confirmar: TF-IDF com regressão logística.

## 6. Fine-tuning

*A preencher.* Já decidido no PRD: **QLoRA** no **Unsloth**, modelo base de cerca de **4B** parâmetros (a definir), treinado no próprio computador (WSL2, RTX 4060).

## 7. Avaliação

*A preencher.* Já decidido no PRD: comparar **baseline × modelo ajustado × modelo sem ajuste**, medindo **taxa de acerto** e **tempo de resposta**.

## 8. Publicação

*A preencher.* Já decidido no PRD: dataset e modelo no **Hugging Face**, o modelo em formato **GGUF**.

## 9. Integração com a Parte A

*A preencher.* Já decidido: o modelo ajustado roda no **Ollama** e entra no **roteador** (M01), no lugar do `qwen3:8b` só nessa chamada. Os três agentes continuam com o `qwen3:8b`.

## 10. Decisões

| Id | Decisão |
| --- | --- |
| M01 | O modelo da Parte B classifica a intenção da mensagem (`PRODUTOS`, `POLITICAS`, `PEDIDOS`, `FORA_DO_TEMA`); concorrentes: baseline scikit-learn, modelo ~4B sem ajuste, modelo ~4B ajustado e `qwen3:8b`; o modelo ajustado entra no roteador. Extrair filtros em JSON foi descartado (seção 3) |
| M02 | Cada exemplo tem 3 colunas: `intencao_anterior` (`NENHUMA` na primeira mensagem), `mensagem` e `intencao`; todos os concorrentes recebem a intenção anterior, como o roteador (06, A10) (seção 4.1) |

## 11. Perguntas em aberto

| Pergunta | Quando responder | Onde a resposta entra |
| --- | --- | --- |
| [x] M01: qual é a tarefa do modelo? **Classificar a intenção** (20/11/2026) | — | Seção 3 |
| [x] M02: o modelo recebe também a intenção anterior, como o roteador (06, A10)? **Sim** (21/11/2026) | — | Seção 4.1 |
| [ ] M03: tamanho, origem e divisão do dataset | Próxima decisão | Seção 4.2 |
| [ ] Modelo base de ~4B | Antes do fine-tuning | Seção 6 |
| [ ] O `qwen3:8b` e o modelo ~4B cabem juntos nos 8 GB de VRAM, ou o Ollama vai alternar entre eles (mais espera)? | Seção 9, com teste prático | Seção 9 |
| [ ] Se o scikit-learn vencer, o que vai para a loja? (Ele roda em Python, e a loja é Java) | Depois da avaliação | Seções 7 e 9 |

## Fontes

- [Text Classification in the LLM Era: Where do we stand? (artigo, arXiv)](https://arxiv.org/pdf/2502.11830): regressão logística competitiva com LLMs ajustados em classificação
- [Reconhecimento de intenção em e-commerce multilíngue com QLoRA (artigo, arXiv)](https://arxiv.org/pdf/2510.21970v1)
- [Como treinar um modelo de decisão para classificar intenções com Unsloth e Qwen3.5-4B (Machine Learning Mastery)](https://machinelearningmastery.com/how-to-train-your-own-decision-model-like-jev-using-unsloth/)
- 01-PRD do AisleSmart, Parte B (resumida em `resumo-1-prd-e-arquitetura.md`, seção 10)
- [Hugging Face: usar o Ollama com modelos GGUF (documentação oficial)](https://huggingface.co/docs/hub/ollama)
- [Fine-tuning com Unsloth em GPU de consumo (DEV Community)](https://dev.to/jovan_chan_9500711396d4e6/fine-tuning-llama3-unsloth-2026-4odd)
