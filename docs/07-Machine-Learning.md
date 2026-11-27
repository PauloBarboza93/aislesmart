# 07-Machine Learning

> **Status:** rascunho v0.11 (27/11/2026): M10 decidida (seção 7): métricas, leitura das respostas como na loja e medição do tempo. v0.10 (26/11/2026): M09, exportação em GGUF `q4_k_m` para o Ollama como `aislesmart-intencao`; seção 6 (fine-tuning) completa. v0.9 (26/11/2026): M08, configuração do treino com os valores recomendados pelo Unsloth (seção 6.2). v0.8 (25/11/2026): M07, modelo base Qwen3-4B-Instruct-2507 (seção 6.1). v0.7 (25/11/2026): M06, baseline com TF-IDF de pedaços de 2 a 5 letras e regressão logística (seção 5). v0.6 (25/11/2026): M05, 3 arquivos CSV com a coluna `origem` e licença CC BY 4.0; seção 4 (dataset) completa. v0.5 (21/11/2026): M04, ~100 exemplos de treino e validação escritos à mão e ~220 gerados pelo `qwen3:8b` local, todos revisados (seção 4.3). v0.4 (21/11/2026): M03, cerca de 400 exemplos, com teste de 80 escrito só à mão (seção 4.2). v0.3 (21/11/2026): M02, a entrada tem a intenção anterior e a mensagem (seção 4.1). v0.2 (20/11/2026): M01, o modelo classifica a intenção (seção 3). As seções 4 a 9 serão preenchidas uma por vez.
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
| Modelo ~4B sem ajuste | O mesmo modelo base (Qwen3-4B-Instruct-2507, M07), só com instrução, para medir o ganho do ajuste |
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

*Decidido em 25/11/2026 (M05).*

**Arquivos:** três CSV (*Comma-Separated Values*, valores separados por vírgula), em UTF-8, na pasta `ml/dados/`:

| Arquivo | Conteúdo |
| --- | --- |
| `treino.csv` | ~270 exemplos |
| `validacao.csv` | ~50 exemplos |
| `teste.csv` | 80 exemplos, só escritos à mão |

**Colunas:** as 3 da M02 e mais `origem`:

```
intencao_anterior,mensagem,intencao,origem
PRODUTOS,e o segundo?,PRODUTOS,manual
NENHUMA,vcs trocam produto vencido?,POLITICAS,gerado
```

- `origem` vale `manual` ou `gerado` (M04). Os modelos **não** leem essa coluna; ela serve para a avaliação comparar o desempenho nos exemplos escritos à mão e nos gerados.

**Por que CSV:** abre em planilha (Excel, WPS, LibreOffice) para a revisão exemplo por exemplo (M04), é lido direto pelo pandas e pelo scikit-learn, e o Hugging Face aceita CSV. Um arquivo por parte deixa a divisão da M03 visível e evita misturar teste com treino.

**Codificação:** UTF-8 comum, o padrão do Python e do Hugging Face. Para revisar numa planilha, abrir o arquivo escolhendo UTF-8 na janela de importação e salvar de novo como CSV em UTF-8, para os acentos não se perderem.

**Licença:** CC BY 4.0 (*Creative Commons Attribution*): qualquer pessoa pode usar o dataset, inclusive comercialmente, desde que dê o crédito. É compatível com a M04 (o Qwen3 é Apache 2.0) e fica registrada no *dataset card* (o README do dataset no Hugging Face).

**Alternativas descartadas:** JSON Lines (mais comum em fine-tuning, mas ruim de revisar em planilha; se o Unsloth pedir esse formato, o caderno converte na hora, sem mudar os arquivos) e um arquivo único com uma coluna de divisão (mais fácil de misturar as partes por engano).

## 5. Baseline

*Decidido em 25/11/2026 (M06).*

Um *pipeline* (sequência de etapas) do scikit-learn, no caderno `ml/notebooks/02_baseline.ipynb` (02, seção 8):

```python
from sklearn.pipeline import make_pipeline
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.linear_model import LogisticRegression

baseline = make_pipeline(
    TfidfVectorizer(analyzer="char_wb", ngram_range=(2, 5), strip_accents="unicode"),
    LogisticRegression(max_iter=1000),
)
```

| Peça | O que faz |
| --- | --- |
| `TfidfVectorizer` | Transforma o texto em números (TF-IDF, *Term Frequency–Inverse Document Frequency*: dá mais peso ao que é frequente na mensagem e raro no conjunto) |
| `strip_accents="unicode"` | Remove os acentos ("café" e "cafe" viram a mesma coisa). Letras minúsculas já são o padrão |
| `analyzer="char_wb"`, `ngram_range=(2, 5)` | Usa pedaços de 2 a 5 letras dentro de cada palavra, em vez de palavras inteiras. Tolera erros de digitação e abreviações (M04): "trca" e "troca" ainda dividem pedaços |
| `LogisticRegression` | O classificador que escolhe uma das 4 intenções. `max_iter=1000` só dá mais tentativas para o cálculo terminar |

**Entrada:** a intenção anterior e a mensagem num texto só (M02), por exemplo `"PRODUTOS | e o segundo?"`. A coluna `origem` não entra (M05).

**Uso dos dados:**

1. Treinar com `treino.csv`.
2. Usar `validacao.csv` para **uma única comparação**: pedaços de 2 a 5 letras (`char_wb`, `(2, 5)`) contra palavras inteiras (`word`, `(1, 2)`). Fica a configuração que acertar mais.
3. O `teste.csv` só é usado **uma vez, no fim**, junto com os outros concorrentes (seção 7).

**Por que assim:** poucas linhas, fáceis de explicar inteiras; treina em segundos, sem placa de vídeo; e é o baseline clássico para classificar texto.

**Alternativas descartadas:**

- Pedaços de até 8 letras: viram quase palavras inteiras, perdem a tolerância a erros de digitação e, com ~270 exemplos de treino, aumentam o risco de o modelo decorar o treino (*overfitting*), com ganho provável pequeno.
- Outros classificadores (SVM, *Support Vector Machine*; Naive Bayes): ganho provável pequeno e tiraria o foco da pesquisa.
- Busca automática de parâmetros (*grid search*): com ~50 exemplos de validação, o risco é ajustar demais a eles.

## 6. Fine-tuning

Já decidido no PRD: **QLoRA** no **Unsloth**, treinado no próprio computador (WSL2, RTX 4060).

### 6.1 Modelo base

*Decidido em 25/11/2026 (M07).*

**Modelo:** `Qwen3-4B-Instruct-2507` (licença Apache 2.0), carregado no Unsloth pela versão já comprimida em 4 bits que o próprio Unsloth publica no Hugging Face.

**Por que este:**

- **Sem o modo "pensar":** esta versão só responde direto, sem blocos de raciocínio. Isso elimina no roteador o cuidado anotado no 06 (A03), de desligar o pensamento do `qwen3`.
- **Cabe na RTX 4060:** pela tabela do Unsloth, o QLoRA precisa de pelo menos ~3,5 GB para 3B e ~5 GB para 7B; um 4B fica perto de 4 GB, com folga nos 8 GB.
- **Leve no Ollama:** comprimido em 4 bits (Q4_K_M), tem cerca de 2,5 GB, menos da metade do `qwen3:8b`.
- **Comparação justa:** é da mesma família do `qwen3:8b` (M01), então a pesquisa compara tamanho e ajuste, não famílias diferentes de modelo.
- **Licença Apache 2.0:** permite publicar o modelo ajustado no Hugging Face (seção 8).

**Alternativas descartadas:**

- **Qwen3.5-4B** (Apache 2.0, mais novo): também processa imagens, o que traz peças sem uso aqui, e tem arquitetura mais recente, com mais chance de problema na conversão para o Ollama. Pode entrar depois como experimento extra.
- **Gemma 3 4B e Llama 3.2 3B:** licenças próprias, com mais regras para publicar o modelo ajustado.

### 6.2 Configuração do treino

*Decidido em 26/11/2026 (M08).* Valores recomendados pela documentação do Unsloth, sem ajuste manual.

**Formato de cada exemplo** (conversa curta, com a mesma instrução em todos):

```
[sistema]    Classifique a mensagem do cliente de uma loja de alimentos em uma
             intenção: PRODUTOS, POLITICAS, PEDIDOS ou FORA_DO_TEMA.
             Responda só com a palavra.
[usuário]    Intenção anterior: PRODUTOS
             Mensagem: e o segundo?
[assistente] PRODUTOS
```

- O modelo aprende **só a resposta** (a palavra da intenção), não a instrução: opção `train_on_responses_only` do Unsloth.
- **A mesma instrução é usada na avaliação** do modelo sem ajuste e do `qwen3:8b` (seção 7), para que a única diferença entre eles seja o treino.
- O texto final da instrução pode ser refinado no caderno, desde que seja um só para todos os concorrentes.

**Parâmetros do LoRA:**

| Parâmetro | Valor | O que é |
| --- | --- | --- |
| `r` (posto) | 16 | Quanto o LoRA pode aprender; o Unsloth sugere 16 ou 32, e 16 basta para uma tarefa simples |
| `lora_alpha` | 16 | A "força" do ajuste; regra comum: o posto vezes 1 ou 2 |
| Camadas treinadas | Todas as camadas lineares | Padrão dos exemplos do Unsloth |

**Treino:**

| Parâmetro | Valor | Por quê |
| --- | --- | --- |
| Taxa de aprendizado | `2e-4` | Ponto de partida recomendado pelo Unsloth para LoRA e QLoRA |
| Épocas | até 3 | Recomendação de 1 a 3; mais que isso aumenta o risco de decorar o treino |
| Validação | ao fim de cada época | Fica a época com o menor erro em `validacao.csv` |
| Tamanho máximo do texto | 256 tokens | As mensagens são curtas; economiza memória |
| Semente (*seed*) | número fixo | Para repetir o treino com o mesmo resultado |

O treino roda no caderno `ml/notebooks/03_finetuning.ipynb` (02, seção 8).

**Alternativa descartada:** testar várias combinações de parâmetros. Com ~50 exemplos de validação, o ganho seria pequeno e o risco de ajustar demais à validação, alto.

### 6.3 Exportação para o Ollama

*Decidido em 26/11/2026 (M09).*

**1. GGUF comprimido em `q4_k_m`**, com uma linha do Unsloth, no fim do caderno `03_finetuning.ipynb`:

```python
model.save_pretrained_gguf("aislesmart-intencao", tokenizer, quantization_method="q4_k_m")
```

- O `q4_k_m` é o método marcado como recomendado na documentação do Unsloth e é a mesma compressão do modelo base no Ollama (~2,5 GB).
- **Comparação justa:** o modelo sem ajuste também roda em `q4_k_m` (seção 7), então a única diferença entre os dois é o treino.

**2. `Modelfile`** (o arquivo de configuração que o Ollama precisa): o Unsloth gera automaticamente, já com o formato de conversa do treino. Acrescentamos:

- a **instrução fixa da M08**, a mesma do treino;
- `PARAMETER temperature 0`: a mesma mensagem recebe sempre a mesma resposta, o certo para classificar e para repetir os testes.

**3. Nome no Ollama:** `aislesmart-intencao`.

```bash
ollama create aislesmart-intencao -f Modelfile
```

**4. Adaptador LoRA:** a parte treinada (alguns MB) também é guardada, para ser publicada no Hugging Face (seção 8) e reaproveitada sem baixar o GGUF.

**Alternativas descartadas:** `q8_0` (perde menos qualidade, mas tem quase o dobro do tamanho e deixaria a comparação com o modelo base em `q4_k_m` injusta) e a conversão manual com o llama.cpp (mais passos e mais chance de erro, sem ganho).

## 7. Avaliação

*Decidido em 27/11/2026 (M10).* Os quatro concorrentes (seção 3) são avaliados no `teste.csv`, uma única vez (M06), no caderno `ml/notebooks/04_avaliacao.ipynb`.

### 7.1 Métricas de acerto

Todas calculadas com o scikit-learn (`accuracy_score`, `f1_score`, `confusion_matrix`).

| Métrica | O que mostra | Por quê |
| --- | --- | --- |
| **Taxa de acerto** (*accuracy*) | % de mensagens com a intenção certa | Meta de 85% da HU-10 (PRD); a mais fácil de explicar |
| **F1 macro** | Média simples do F1 das 4 intenções (o F1 mede, por intenção, quanto o modelo acerta sem confundir com as outras) | Todas as intenções pesam igual; mostra se o modelo vai mal em uma delas |
| **Matriz de confusão** | Qual intenção foi confundida com qual | Explica os erros (ex.: `FORA_DO_TEMA` virando `POLITICAS`) |
| **Acerto por situação** | Primeira mensagem, continuação e troca de assunto (M03) | Mostra se a intenção anterior (M02) foi aprendida |

### 7.2 Leitura das respostas dos LLMs

- Cada resposta passa pela mesma regra do método `interpretar` do 06 (A03), reescrita em Python: sem acentos, maiúsculas, singular ou plural; o que não for reconhecido vira `FORA_DO_TEMA`. Assim, a nota é a que a loja teria de verdade.
- Também se conta, à parte, quantas respostas vieram fora do formato (mais de uma palavra ou palavra fora da lista).
- Todos os LLMs recebem a mesma instrução fixa (M08) e rodam com `temperature 0` (M09).

### 7.3 Tempo de resposta

- Mede-se o tempo de cada mensagem e informa-se a **mediana** (o valor do meio, que não é distorcido por uma resposta lenta isolada).
- **Aquecimento:** a primeira chamada de cada modelo (que carrega o modelo na placa de vídeo) não entra na conta.
- **Um modelo por vez** na placa de vídeo, para um não atrapalhar o outro.
- O baseline roda no processador (CPU); isso fica dito junto do resultado.

### 7.4 Rodadas e resultado

- Com `temperature 0`, os LLMs sempre dão a mesma resposta: **uma rodada basta**.
- O caderno salva uma tabela final (acerto, F1 macro, respostas fora do formato e mediana do tempo por concorrente) e a matriz de confusão como imagem, para o README e o Hugging Face (seção 8).
- **Qual modelo vai para a loja** é decidido na seção 9, com o resultado em mãos.

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
| M05 | Três arquivos CSV em UTF-8 (`treino.csv`, `validacao.csv`, `teste.csv`) em `ml/dados/`, com as colunas da M02 e mais `origem` (`manual` ou `gerado`, só para a análise); licença CC BY 4.0 (seção 4.4) |
| M06 | Baseline: `TfidfVectorizer(analyzer="char_wb", ngram_range=(2, 5), strip_accents="unicode")` + `LogisticRegression`; entrada "intenção anterior \| mensagem"; a validação compara só com palavras inteiras (`word`, `(1, 2)`); teste usado uma vez, no fim (seção 5) |
| M07 | Modelo base do fine-tuning: `Qwen3-4B-Instruct-2507` (Apache 2.0; só modo sem pensamento; ~4 GB de VRAM no QLoRA; ~2,5 GB no Ollama em Q4_K_M). Qwen3.5-4B, Gemma 3 4B e Llama 3.2 3B descartados (seção 6.1) |
| M08 | Treino com os valores recomendados pelo Unsloth: exemplo em formato de conversa com instrução fixa (a mesma na avaliação), aprendendo só a resposta; LoRA `r=16`, `lora_alpha=16`, todas as camadas lineares; taxa `2e-4`, até 3 épocas com a melhor escolhida pela validação, 256 tokens, semente fixa (seção 6.2) |
| M09 | Exportação com `save_pretrained_gguf` em `q4_k_m` (mesma compressão do modelo sem ajuste); `Modelfile` gerado pelo Unsloth, com a instrução fixa da M08 e `temperature 0`; nome no Ollama `aislesmart-intencao`; adaptador LoRA guardado para publicação (seção 6.3) |
| M10 | Avaliação no teste, uma vez: taxa de acerto (meta 85%), F1 macro, matriz de confusão e acerto por situação; respostas dos LLMs lidas pela regra do `interpretar` (A03) em Python, contando as fora do formato; mediana do tempo com aquecimento e um modelo por vez na placa; uma rodada (`temperature 0`) (seção 7) |

## 11. Perguntas em aberto

| Pergunta | Quando responder | Onde a resposta entra |
| --- | --- | --- |
| [x] M01: qual é a tarefa do modelo? **Classificar a intenção** (20/11/2026) | — | Seção 3 |
| [x] M02: o modelo recebe também a intenção anterior, como o roteador (06, A10)? **Sim** (21/11/2026) | — | Seção 4.1 |
| [x] M03: tamanho e divisão do dataset. **~400 exemplos, teste de 80 escrito à mão** (21/11/2026) | — | Seção 4.2 |
| [x] M04: como gerar os exemplos de treino e validação. **~100 à mão + ~220 gerados pelo `qwen3:8b` local, revisados** (21/11/2026) | — | Seção 4.3 |
| [x] M05: formato do arquivo e licença de publicação. **3 CSV em UTF-8 com a coluna `origem`; CC BY 4.0** (25/11/2026) | — | Seção 4.4 |
| [x] M06: modelo do baseline. **TF-IDF com pedaços de 2 a 5 letras + regressão logística** (25/11/2026) | — | Seção 5 |
| [x] M07: modelo base de ~4B para o fine-tuning. **Qwen3-4B-Instruct-2507** (25/11/2026) | — | Seção 6.1 |
| [x] M08: configuração do treino. **Valores recomendados pelo Unsloth** (26/11/2026) | — | Seção 6.2 |
| [x] M09: exportação do modelo ajustado para o Ollama. **GGUF `q4_k_m`, `aislesmart-intencao`** (26/11/2026) | — | Seção 6.3 |
| [x] M10: como avaliar. **Acerto, F1 macro, matriz de confusão, por situação; mediana do tempo** (27/11/2026) | — | Seção 7 |
| [ ] M11: o que publicar no Hugging Face e como (dataset, modelo, cards) | Próxima decisão | Seção 8 |
| [ ] O `qwen3:8b` e o modelo ~4B cabem juntos nos 8 GB de VRAM, ou o Ollama vai alternar entre eles (mais espera)? | Seção 9, com teste prático | Seção 9 |
| [ ] Se o scikit-learn vencer, o que vai para a loja? (Ele roda em Python, e a loja é Java) | Depois da avaliação | Seções 7 e 9 |

## Fontes

- [Logistic Regression makes small LLMs strong and explainable "tens-of-shot" classifiers (Bank of England)](https://www.bankofengland.co.uk/working-paper/2025/improving-text-classification-logistic-regression-llms-tens-of-shot-classifiers): bom desempenho com dezenas de exemplos por classe
- [Qwen3: modelos densos de 0.6B a 32B sob licença Apache 2.0 (blog oficial do Qwen)](https://qwenlm.github.io/blog/qwen3/)
- [Termos de uso do Claude.ai, seção 3 (Anthropic)](https://www.anthropic.com/legal/consumer-terms)
- [Hugging Face: formatos aceitos no upload de datasets, entre eles CSV (documentação oficial)](https://huggingface.co/docs/datasets/v2.10.0/upload_dataset)
- [scikit-learn: `TfidfVectorizer`, parâmetros `analyzer`, `ngram_range` e `strip_accents` (documentação oficial)](https://scikit-learn.org/stable/modules/generated/sklearn.feature_extraction.text.TfidfVectorizer.html)
- [Unsloth: requisitos de VRAM para QLoRA por tamanho de modelo (documentação oficial)](https://docs.unsloth.ai/get-started/beginner-start-here/unsloth-requirements)
- [Unsloth: Qwen3-4B-Instruct-2507 em GGUF, só modo sem pensamento (Hugging Face)](https://huggingface.co/unsloth/Qwen3-4B-Instruct-2507-GGUF)
- [Unsloth: guia de hiperparâmetros do LoRA, posto, alpha, taxa de aprendizado e épocas (documentação oficial)](https://unsloth.ai/docs/get-started/fine-tuning-llms-guide/lora-hyperparameters-guide)
- [Unsloth: salvar em GGUF e métodos de compressão (documentação oficial)](https://unsloth.ai/docs/basics/inference-and-deployment/saving-to-gguf)
- [Unsloth: salvar para o Ollama e criação automática do `Modelfile` (documentação oficial)](https://unsloth.ai/docs/basics/inference-and-deployment/saving-to-ollama)
- [scikit-learn: `f1_score` e a média `macro` (documentação oficial)](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.f1_score.html)
- [Text Classification in the LLM Era: Where do we stand? (artigo, arXiv)](https://arxiv.org/pdf/2502.11830): regressão logística competitiva com LLMs ajustados em classificação
- [Reconhecimento de intenção em e-commerce multilíngue com QLoRA (artigo, arXiv)](https://arxiv.org/pdf/2510.21970v1)
- [Como treinar um modelo de decisão para classificar intenções com Unsloth e Qwen3.5-4B (Machine Learning Mastery)](https://machinelearningmastery.com/how-to-train-your-own-decision-model-like-jev-using-unsloth/)
- 01-PRD do AisleSmart, Parte B (resumida em `resumo-1-prd-e-arquitetura.md`, seção 10)
- [Hugging Face: usar o Ollama com modelos GGUF (documentação oficial)](https://huggingface.co/docs/hub/ollama)
- [Fine-tuning com Unsloth em GPU de consumo (DEV Community)](https://dev.to/jovan_chan_9500711396d4e6/fine-tuning-llama3-unsloth-2026-4odd)
