# 07-Machine Learning

> **Status:** rascunho v0.1 (19/11/2026): só a estrutura e o que já foi decidido no PRD. As seções 3 a 9 serão preenchidas uma por vez, começando pela M01 (seção 3).
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

*Primeira decisão a tomar (M01). Em aberto.*

O PRD diz duas coisas que precisam ser conciliadas:

- o fine-tuning ensina o modelo a **transformar a pergunta em filtros JSON** (ex.: "café em grãos até R$40" → `{"busca": "café em grãos", "precoMaximo": 40}`);
- o baseline é um **classificador** com scikit-learn.

Um classificador escolhe uma resposta de uma lista fechada; ele não escreve um JSON com vários campos. Para a comparação ser justa, os três modelos precisam fazer **a mesma tarefa**. Opções a avaliar na próxima conversa:

| Opção | Tarefa | Baseline | Observação inicial |
| --- | --- | --- | --- |
| A | Classificar a intenção (`PRODUTOS`, `POLITICAS`, `PEDIDOS`, `FORA_DO_TEMA`) | Classificador de texto com scikit-learn | É exatamente o que o roteador faz (06, seção 3); medida simples (acertou ou não) |
| B | Extrair os filtros em JSON | Regras simples (ex.: expressões regulares) ou um classificador por filtro | Mais próxima do PRD; avaliação e baseline mais trabalhosos |
| C | As duas, em etapas | — | Mais completa; mais tempo e mais pontos para explicar |

*A escolha decide o formato do dataset, o baseline, as métricas e onde o modelo entra na Parte A.*

## 4. Dataset

*A preencher depois da M01.* Já decidido no PRD: perguntas de clientes em **português**, cada uma com a **resposta esperada**. A definir: quantidade, como gerar e revisar as perguntas, divisão treino / validação / teste e licença de publicação.

## 5. Baseline

*A preencher depois da M01.* Já decidido no PRD: **scikit-learn**.

## 6. Fine-tuning

*A preencher.* Já decidido no PRD: **QLoRA** no **Unsloth**, modelo base de cerca de **4B** parâmetros (a definir), treinado no próprio computador (WSL2, RTX 4060).

## 7. Avaliação

*A preencher.* Já decidido no PRD: comparar **baseline × modelo ajustado × modelo sem ajuste**, medindo **taxa de acerto** e **tempo de resposta**.

## 8. Publicação

*A preencher.* Já decidido no PRD: dataset e modelo no **Hugging Face**, o modelo em formato **GGUF**.

## 9. Integração com a Parte A

*A preencher.* Já decidido no PRD: o modelo ajustado roda no **Ollama** e é usado pela Parte A. Onde exatamente ele entra depende da M01.

## 10. Decisões

| Id | Decisão |
| --- | --- |
| — | Nenhuma ainda neste documento |

## 11. Perguntas em aberto

| Pergunta | Quando responder | Onde a resposta entra |
| --- | --- | --- |
| [ ] M01: qual é a tarefa do modelo (intenção, filtros JSON ou as duas)? | Próxima conversa do 07 | Seção 3 |
| [ ] Modelo base de ~4B | Depois da M01 | Seção 6 |
| [ ] Tamanho e forma de criar o dataset | Depois da M01 | Seção 4 |

## Fontes

- 01-PRD do AisleSmart, Parte B (resumida em `resumo-1-prd-e-arquitetura.md`, seção 10)
- [Hugging Face: usar o Ollama com modelos GGUF (documentação oficial)](https://huggingface.co/docs/hub/ollama)
- [Fine-tuning com Unsloth em GPU de consumo (DEV Community)](https://dev.to/jovan_chan_9500711396d4e6/fine-tuning-llama3-unsloth-2026-4odd)
