# 07-Machine Learning

> **Status:** rascunho v0.5 (21/11/2026): M04 decidida (seção 4.3): ~100 exemplos de treino e validação escritos à mão e ~220 gerados pelo `qwen3:8b` local, todos revisados. v0.4 (21/11/2026): M03, cerca de 400 exemplos, com teste de 80 escrito só à mão (seção 4.2). v0.3 (21/11/2026): M02, a entrada tem a intenção anterior e a mensagem (seção 4.1). v0.2 (20/11/2026): M01, o modelo classifica a intenção (seção 3). As seções 4 a 9 serão preenchidas uma por vez.
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

### 4.2 Tamanho e divisão

*Decidido em 21/11/2026 (M03).*

| Parte | Quantidade | Para que serve | Origem |
| --- | --- | --- | --- |
| Treino | ~270 | Os modelos aprendem com estes | À mão + gerados (M04) |
| Validação | ~50 | Acompanhar o fine-tuning e perceber se o modelo está "decorando" os exemplos | À mão + gerados |
| Teste | 80 (20 por intenção) | A nota final da pesquisa; nenhum modelo vê estes exemplos antes | Só à mão |
| **Total** | **~400** (~100 por intenção) | | |

**Regras:**

1. **Mesma quantidade por intenção** em cada parte, para nenhum modelo tirar nota alta só por chutar a intenção mais comum.
2. **Mistura de situações em todas as partes:** cerca de metade como primeira mensagem (`intencao_anterior = NENHUMA`), um quarto de continuação (mesma intenção da anterior) e um quarto de troca de assunto (intenção diferente) (seção 4.1).
3. **O teste é escrito só à mão** e inclui as 20 perguntas de teste do 06 (seção 8), adaptadas ao formato da seção 4.1. Assim, nenhum modelo é avaliado em texto parecido com o que um LLM gerou, e a pesquisa conversa com os testes do assistente.
4. Nenhuma mensagem se repete entre treino, validação e teste.

**Por que esse tamanho:** dá para revisar linha por linha em uma ou duas sessões. Para 4 classes, a regressão logística já funciona bem com algumas dezenas de exemplos por classe, e algumas centenas costumam bastar para o fine-tuning de uma tarefa simples. Se a validação mostrar que falta, o treino cresce depois sem refazer o teste.

**Alternativas descartadas:** ~200 exemplos (o teste ficaria pequeno e a nota, instável) e ~1.000 (revisão de dias, com ganho provável pequeno para 4 classes).

### 4.3 Geração dos exemplos

*Decidido em 21/11/2026 (M04).*

| Parte | Quantos | Como |
| --- | --- | --- |
| Teste | 80 | Só à mão (M03) |
| Treino e validação | ~100 | À mão, por Paulo Oliveira |
| Treino e validação | ~220 | Gerados pelo `qwen3:8b` no Ollama local e revisados um a um |

**Como gerar:**

- A geração roda no caderno `ml/notebooks/01_dados.ipynb` (02, seção 8), chamando o Ollama local. O texto do pedido (prompt) usado fica salvo no repositório, para qualquer pessoa repetir a geração.
- O pedido inclui a intenção, a intenção anterior e a situação (primeira mensagem, continuação ou troca de assunto), para manter as proporções da M03.
- **Variedade de propósito:** abreviações ("vcs", "qto"), frases sem acento, erros de digitação, mensagens curtas e longas, perguntas e pedidos diretos. Cliente real não escreve como num livro.

**Como revisar (cada exemplo gerado):**

- Descartar mensagens repetidas ou quase iguais a outras.
- Corrigir o rótulo quando a intenção estiver errada, seguindo as regras do roteador (06, A04): assunto da loja vence saudação; entre dois assuntos da loja, vale o primeiro.
- Descartar mensagens que não pareçam escritas por um cliente.
- Conferir que nenhuma mensagem é igual ou muito parecida com uma do teste.

**Por que o `qwen3:8b`:**

- **Licença:** os modelos Qwen3 têm pesos abertos sob a licença Apache 2.0, que permite usar e publicar o que eles geram. Como o dataset vai para o Hugging Face, isso importa.
- **Sem custo:** já está instalado e roda no próprio computador.
- **Sem zona cinzenta:** os termos de uso do Claude.ai proíbem usar o serviço para desenvolver produtos que concorram com os da Anthropic. Um classificador de intenções para uma loja dificilmente se encaixa nisso, mas usar um modelo de licença aberta evita a dúvida.

**Cuidado já resolvido:** o `qwen3:8b` também é um dos concorrentes da avaliação. Como o teste é escrito só à mão (M03), ele não leva vantagem por ter gerado parte do treino.

**Alternativa descartada:** escrever os ~320 exemplos à mão. Mais autêntico, mas muito mais trabalhoso; gerar com LLM e revisar é uma prática comum e fica documentada na pesquisa.

### 4.4 Formato do arquivo e licença

*A preencher (M05).*

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
| M03 | Cerca de 400 exemplos (~100 por intenção): ~270 de treino, ~50 de validação e 80 de teste (20 por intenção), escrito só à mão e com as 20 perguntas do 06; mesma quantidade por intenção; metade primeira mensagem, um quarto continuação e um quarto troca de assunto (seção 4.2) |
| M04 | Treino e validação: ~100 exemplos escritos à mão e ~220 gerados pelo `qwen3:8b` no Ollama local (licença Apache 2.0), com variedade de escrita e revisão um a um; o prompt de geração fica salvo no repositório (seção 4.3) |

## 11. Perguntas em aberto

| Pergunta | Quando responder | Onde a resposta entra |
| --- | --- | --- |
| [x] M01: qual é a tarefa do modelo? **Classificar a intenção** (20/11/2026) | — | Seção 3 |
| [x] M02: o modelo recebe também a intenção anterior, como o roteador (06, A10)? **Sim** (21/11/2026) | — | Seção 4.1 |
| [x] M03: tamanho e divisão do dataset. **~400 exemplos, teste de 80 escrito à mão** (21/11/2026) | — | Seção 4.2 |
| [x] M04: como gerar os exemplos de treino e validação. **~100 à mão + ~220 gerados pelo `qwen3:8b` local, revisados** (21/11/2026) | — | Seção 4.3 |
| [ ] M05: formato do arquivo e licença de publicação | Próxima decisão | Seção 4.4 |
| [ ] Modelo base de ~4B | Antes do fine-tuning | Seção 6 |
| [ ] O `qwen3:8b` e o modelo ~4B cabem juntos nos 8 GB de VRAM, ou o Ollama vai alternar entre eles (mais espera)? | Seção 9, com teste prático | Seção 9 |
| [ ] Se o scikit-learn vencer, o que vai para a loja? (Ele roda em Python, e a loja é Java) | Depois da avaliação | Seções 7 e 9 |

## Fontes

- [Logistic Regression makes small LLMs strong and explainable "tens-of-shot" classifiers (Bank of England)](https://www.bankofengland.co.uk/working-paper/2025/improving-text-classification-logistic-regression-llms-tens-of-shot-classifiers): bom desempenho com dezenas de exemplos por classe
- [Qwen3: modelos densos de 0.6B a 32B sob licença Apache 2.0 (blog oficial do Qwen)](https://qwenlm.github.io/blog/qwen3/)
- [Termos de uso do Claude.ai, seção 3 (Anthropic)](https://www.anthropic.com/legal/consumer-terms)
- [Text Classification in the LLM Era: Where do we stand? (artigo, arXiv)](https://arxiv.org/pdf/2502.11830): regressão logística competitiva com LLMs ajustados em classificação
- [Reconhecimento de intenção em e-commerce multilíngue com QLoRA (artigo, arXiv)](https://arxiv.org/pdf/2510.21970v1)
- [Como treinar um modelo de decisão para classificar intenções com Unsloth e Qwen3.5-4B (Machine Learning Mastery)](https://machinelearningmastery.com/how-to-train-your-own-decision-model-like-jev-using-unsloth/)
- 01-PRD do AisleSmart, Parte B (resumida em `resumo-1-prd-e-arquitetura.md`, seção 10)
- [Hugging Face: usar o Ollama com modelos GGUF (documentação oficial)](https://huggingface.co/docs/hub/ollama)
- [Fine-tuning com Unsloth em GPU de consumo (DEV Community)](https://dev.to/jovan_chan_9500711396d4e6/fine-tuning-llama3-unsloth-2026-4odd)
