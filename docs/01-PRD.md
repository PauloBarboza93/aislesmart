# 01-PRD — AisleSmart

Nov 4, 2026 · @Oliveira

## Como ler este documento

Este PRD define **o que** o AisleSmart vai fazer e **por quê**, antes de qualquer código. PRD significa *Product Requirements Document* (Documento de Requisitos do Produto).

Num projeto em que a IA escreve o código, o PRD é o seu principal instrumento de controle. A IA de código vai ler este documento e gerar o que está escrito aqui. Se algo estiver vago, ela vai inventar.

- **Status:** rascunho v0.2, em revisão por Oliveira.
- **Nome do projeto:** AisleSmart ("aisle" é o corredor do supermercado e "smart" é inteligente: um "corredor inteligente").
- **Documentos seguintes:** 02-Arquitetura, 03-Modelo de dados, 04-Contratos de API, 05-Ferramentas MCP, 06-Agentes, 07-Machine Learning.
- **Siglas:** toda sigla aparece com o significado entre parênteses na primeira vez, e o glossário completo está na seção 11.
- **Dica:** as caixas "Para iniciante" explicam termos novos. Pode apagá-las quando não precisar mais.

## 1. Visão geral e problema

O AisleSmart é uma loja online de **alimentos**, feita para estudo e portfólio, com um **assistente de compras por IA** (Inteligência Artificial) que conversa com o cliente, encontra produtos, tira dúvidas sobre as políticas da loja e ajuda a montar o carrinho.

O projeto tem duas partes:

- **Parte A (loja e assistente):** o e-commerce com o assistente de IA, em React e Java.
- **Parte B (Machine Learning, aprendizado de máquina):** treino, ajuste e avaliação de modelos em Python. O modelo treinado na Parte B passa a funcionar dentro do assistente da Parte A.

**Problema que resolve:** em lojas comuns, o cliente precisa navegar por filtros e páginas para achar o que quer. Dúvidas como "aceita troca?" ou "quanto custa o frete para a minha cidade?" ficam escondidas em páginas de FAQ. O assistente permite perguntar em linguagem natural e receber respostas baseadas nos dados reais da loja.

**Segmento da loja (definido):** alimentos (mercearia, bebidas, hortifrúti, laticínios e lanches). É um catálogo fácil de entender, com atributos claros (categoria, marca, peso ou volume, preço, ingredientes e alergênicos) e bom para testar buscas como "café em grãos até R$40" ou "lanche sem lactose". O tema também se aproxima do negócio da vaga-alvo, que é de uma empresa de delivery de comida.

> **Para iniciante:** este é um projeto de **estudo e portfólio**, não uma loja real. Não haverá pagamento de verdade nem dados reais de clientes.

## 2. Objetivos

O projeto tem dois tipos de objetivo: o que Oliveira quer **aprender** e o que o produto precisa **entregar**.

**Objetivos de aprendizado**

1. Entender e aplicar os quatro conceitos de IA: LLM (Large Language Model, modelo de linguagem grande), RAG (Retrieval-Augmented Generation, geração aumentada por recuperação), MCP (Model Context Protocol, protocolo de contexto de modelo) e agentes.
2. Aprender a desenvolver com IA de forma organizada: documentação primeiro, depois skills (arquivos de instruções para a IA de código), depois código.
3. Praticar a stack (conjunto de tecnologias) fullstack: React + Tailwind no front-end, Java + Spring Boot no back-end.
4. Construir a Parte B em Python: treinar e avaliar modelos, fazer fine-tuning (ajuste fino) de um LLM pequeno e publicá-lo no Hugging Face.
5. Montar o portfólio esperado pela trilha AI Engineer do programa de estágio em IA do iFood: código de ML no GitHub, modelo publicado no Hugging Face, projeto de pesquisa e artigo técnico.
6. Ter um projeto completo e explicável no GitHub para usar em entrevistas.

**Objetivos do produto (MVP)**

1. O cliente navega no catálogo, vê detalhes e monta um carrinho, sem precisar da IA.
2. O cliente pergunta ao assistente sobre produtos e recebe sugestões que existem de verdade no catálogo.
3. O cliente pergunta sobre políticas (trocas, entrega, validade) e recebe a resposta com a fonte citada.
4. O assistente consulta estoque e adiciona itens ao carrinho, sempre pedindo confirmação antes.
5. O cliente consulta a situação de um pedido pelo chat.

## 3. Público e personas

O MVP atende duas personas: quem compra e quem administra a loja.

| Persona | Quem é | O que precisa |
| --- | --- | --- |
| Cliente (Ana, 32 anos) | Trabalha o dia todo, compra comida pelo celular, tem pouco tempo e evita lactose | Achar o produto certo rápido, tirar dúvidas sem procurar no FAQ, saber onde está o pedido |
| Administrador(a) da loja | Cadastra produtos e políticas | Cadastrar produtos e documentos de política e ver o assistente usar essas informações atualizadas |

> **Para iniciante:** persona é um personagem fictício que representa um tipo de usuário. Ela ajuda a decidir o que entra no produto: "a Ana precisa disso?"

## 4. Escopo do MVP

O MVP é a menor versão que já mostra os quatro conceitos de IA funcionando juntos (Parte A) e o modelo treinado em uso (Parte B). Tudo o que não for essencial para isso fica para depois.

> **Para iniciante:** MVP significa *Minimum Viable Product* (Produto Mínimo Viável). Definir o que fica **fora** é tão importante quanto o que fica dentro: evita que o projeto cresça sem fim.

**Dentro do MVP: Parte A (loja e assistente)**

- Catálogo com cerca de 30 a 50 produtos fictícios, de alimentos, com categoria, marca, peso ou volume, preço, ingredientes, alergênicos e estoque
- Página de listagem com busca simples e filtro por categoria
- Página de detalhe do produto
- Carrinho: adicionar, remover, alterar quantidade
- Pedido simulado (sem pagamento real), com status fictício
- Documentos de política da loja: trocas e devoluções, entrega, validade dos produtos, formas de pagamento
- Widget de chat com o assistente de IA, com respostas em tempo real (streaming)
- Assistente com RAG (busca nos dados da loja), ferramentas via MCP (protocolo de ferramentas) e agentes especializados
- Ambiente local com Docker Compose (ferramenta que sobe o banco de dados com um comando)

**Dentro do MVP: Parte B (Machine Learning, em Python)**

- Conjunto de dados com perguntas de clientes em português, criado para o projeto e publicado no Hugging Face
- Modelo clássico com scikit-learn (biblioteca de ML em Python) que classifica a intenção da pergunta: produtos, políticas ou pedidos. Serve de base de comparação
- Fine-tuning (ajuste fino) de um LLM pequeno, de cerca de 4B (4 bilhões) de parâmetros, com a técnica QLoRA (Quantized Low-Rank Adaptation: treina só uma pequena parte do modelo, por isso cabe numa placa de vídeo de 8 GB). O modelo aprende a transformar a pergunta em filtros de busca
- Modelo ajustado publicado no Hugging Face e usado pelo assistente da Parte A, rodando no Ollama
- Projeto de pesquisa comparando os modelos com métricas (taxa de acerto e tempo de resposta)
- Artigo técnico com os resultados da pesquisa

**Fora do MVP (talvez em versões futuras)**

- Pagamento real e emissão de nota fiscal
- Login com redes sociais (no MVP, um usuário de teste fixo)
- Painel administrativo completo (no MVP, cadastro por script de seed)
- Recomendações personalizadas por histórico de compras
- Aplicativo mobile
- Deploy em produção com alta disponibilidade
- Competições no Kaggle (plataforma de competições de ciência de dados): decidido deixar de fora por falta de tempo

## 5. Funcionalidades e histórias de usuário

São 10 histórias, agrupadas em loja básica (sem IA), assistente de IA e modelo treinado. Cada uma tem critérios de aceite: as condições que precisam ser verdadeiras para considerar a história pronta.

> **Para iniciante:** a história segue o modelo "Como \[quem\], quero \[o quê\], para \[por quê\]". Os critérios de aceite viram os testes. Quando pedir código à IA, cole a história e os critérios no pedido.

### Loja básica (sem IA)

**HU-01 — Ver catálogo.** Como cliente, quero ver a lista de produtos, para escolher o que comprar.

- [ ] A lista mostra nome, foto, preço e categoria
- [ ] Posso filtrar por categoria e buscar por nome
- [ ] A lista é paginada (12 produtos por página)

**HU-02 — Ver detalhe do produto.** Como cliente, quero ver os detalhes de um produto, para decidir se ele me atende.

- [ ] Mostra descrição, peso ou volume, ingredientes, alergênicos e estoque
- [ ] Um produto sem estoque aparece como "Indisponível" e não pode ir ao carrinho

**HU-03 — Gerenciar carrinho.** Como cliente, quero adicionar, remover e alterar itens, para montar minha compra.

- [ ] O total é recalculado a cada alteração
- [ ] Não é possível colocar mais unidades do que o estoque

**HU-04 — Fechar pedido simulado.** Como cliente, quero finalizar o carrinho, para gerar um pedido.

- [ ] O pedido recebe um número e o status "Recebido"
- [ ] O estoque é reduzido após o pedido

### Assistente de IA

**HU-05 — Conversar com o assistente.** Como cliente, quero abrir um chat na loja, para pedir ajuda em linguagem natural.

- [ ] O widget fica em todas as páginas
- [ ] A resposta aparece aos poucos (streaming), sem esperar o texto inteiro
- [ ] O assistente lembra o que foi dito antes na mesma conversa

**HU-06 — Encontrar produtos pelo chat.** Como cliente, quero descrever o que procuro, para receber sugestões.

- [ ] Ex.: "café em grãos até R$40" retorna só produtos reais do catálogo, dentro do preço
- [ ] As sugestões aparecem como cards clicáveis
- [ ] Se não houver produto adequado, o assistente diz isso e não inventa

**HU-07 — Tirar dúvidas sobre políticas.** Como cliente, quero perguntar sobre trocas, entrega e validade, para não precisar procurar no FAQ.

- [ ] A resposta usa só os documentos de política da loja (RAG, busca nos documentos)
- [ ] A resposta cita qual documento foi usado
- [ ] Pergunta fora do assunto da loja recebe uma recusa educada

**HU-08 — Adicionar ao carrinho pelo chat.** Como cliente, quero pedir ao assistente para colocar um item no carrinho, para comprar sem sair da conversa.

- [ ] O assistente verifica o estoque antes (ferramenta exposta via MCP)
- [ ] O assistente **pede confirmação** antes de alterar o carrinho
- [ ] O carrinho na tela é atualizado depois da confirmação

**HU-09 — Consultar pedido pelo chat.** Como cliente, quero perguntar "onde está meu pedido?", para saber o status.

- [ ] O assistente mostra o status só de pedidos do próprio cliente
- [ ] Se o número não existir, o assistente avisa e não inventa um status

### Modelo treinado (Parte B)

**HU-10 — Entender filtros na pergunta.** Como cliente, quero escrever do meu jeito ("algo sem lactose para o café da manhã até R$30"), para que o assistente entenda o que procuro sem eu usar filtros.

- [ ] O modelo ajustado transforma a pergunta em filtros estruturados em JSON (formato de dados em texto), por exemplo: categoria, preço máximo e restrições
- [ ] O modelo acerta os filtros em pelo menos 85% das perguntas do conjunto de teste (meta inicial, a revisar após os primeiros testes)
- [ ] O modelo está publicado no Hugging Face, com documentação (model card)

## 6. O assistente de IA

O assistente combina cinco peças: um **LLM** (Large Language Model, modelo de linguagem grande) que entende e escreve, **RAG** (Retrieval-Augmented Generation, geração aumentada por recuperação) para responder com dados da loja, **MCP** (Model Context Protocol, protocolo de contexto de modelo) para executar ações, **agentes** que dividem o trabalho e o **modelo ajustado** da Parte B, que entende os filtros da pergunta.

| Conceito | Explicação simples | Uso no AisleSmart |
| --- | --- | --- |
| LLM (Large Language Model, modelo de linguagem grande) | O "cérebro" que entende a pergunta e escreve a resposta. Sozinho, não conhece a loja. | Interpreta o pedido do cliente e redige as respostas |
| RAG (geração aumentada por recuperação) | Antes de responder, busca trechos relevantes nos documentos e entrega ao LLM como contexto. | Responde sobre produtos e políticas usando só os dados da loja |
| Embeddings e banco vetorial | Transformam textos em números que representam o significado, para buscar por semelhança e não só por palavra exata. | "Lanche saudável" encontra "barra de cereal integral com castanhas" |
| Tools (ferramentas) | Funções do sistema que o LLM pode pedir para executar. Quem executa é o código, não o LLM. | Verificar estoque, adicionar ao carrinho, consultar pedido |
| MCP (Model Context Protocol, protocolo de contexto de modelo) | Um padrão aberto para expor ferramentas e dados a qualquer aplicação de IA, como uma "tomada universal". | As ferramentas da loja ficam num servidor MCP próprio |
| Agente | Um LLM com um papel, instruções e um conjunto de ferramentas, que decide os passos para cumprir uma tarefa. | Roteador + agentes de produtos, políticas e pedidos |
| Modelo ajustado (fine-tuning) | Um LLM pequeno treinado com exemplos da loja para fazer uma tarefa específica muito bem. | Transforma a pergunta em filtros de busca: categoria, preço máximo e restrições (Parte B) |

```mermaid
flowchart TD
    C["Cliente no chat (widget React)"] -->|pergunta em linguagem natural| R["Agente roteador<br/>o LLM identifica a intenção"]
    R --> P["Agente de produtos<br/>RAG no catálogo"]
    R --> PO["Agente de políticas<br/>RAG nos documentos da loja"]
    R --> PE["Agente de pedidos<br/>carrinho e status via MCP"]
    P --> V[("Banco vetorial (pgvector)")]
    PO --> V
    PE --> M["Servidor MCP<br/>estoque, carrinho, pedidos"]
    M --> A["API da loja<br/>Spring Boot + PostgreSQL"]
```

O roteador lê a pergunta e encaminha para um único agente; só o agente de pedidos pode alterar dados, e sempre com confirmação do cliente.

**Exemplo de conversa (HU-06 + HU-08)**

1. Ana: "Quero um café em grãos de torra média, até R$40."
2. O modelo ajustado transforma a pergunta em filtros (categoria: café; tipo: grãos; torra: média; preço máximo: R$40), e o roteador chama o agente de produtos.
3. O agente busca no banco vetorial, aplica os filtros e mostra 3 cards.
4. Ana: "Coloca o segundo no carrinho."
5. O roteador chama o agente de pedidos, que verifica o estoque pela ferramenta de estoque.
6. O assistente mostra a proposta "1 pacote de 500 g do Café X, por R$36" com o botão **Confirmar**. Ana clica e o item entra no carrinho.

## 7. Requisitos não funcionais e regras da IA

Requisitos não funcionais dizem **como** o sistema deve se comportar (segurança, desempenho, qualidade), não **o que** ele faz.

**Regras do assistente (obrigatórias)**

- Nunca inventar produto, preço, estoque ou status de pedido: esses dados vêm sempre das ferramentas ou do RAG
- Nunca alterar carrinho ou pedido sem confirmação explícita do cliente
- Responder só sobre a loja; pergunta fora do tema recebe recusa educada
- Citar a fonte quando a resposta vier de um documento de política
- Ignorar instruções escondidas em textos de usuário ou de documentos (proteção contra *prompt injection*)
- Não pedir nem guardar dados sensíveis (CPF, número de cartão) no chat

**Qualidade e segurança técnica**

- Chaves de API (senhas de acesso a serviços) em variáveis de ambiente, nunca no código nem no GitHub
- Primeira parte da resposta do chat em até 3 segundos no ambiente local
- Testes automatizados para regras de negócio (carrinho, estoque, pedido)
- Um conjunto de 20 perguntas de teste com a resposta esperada, para avaliar o assistente a cada mudança
- Logs de cada chamada ao LLM e às ferramentas, para entender o que o assistente fez
- Interface responsiva (funciona no celular) e em português do Brasil

## 8. Stack tecnológica

Front-end, back-end, LLM e Machine Learning estão definidos; as demais peças são propostas a confirmar no documento 02-Arquitetura.

| Camada | Tecnologia | Situação |
| --- | --- | --- |
| Front-end | JavaScript, React, Tailwind CSS (com Vite) | Definido |
| Back-end | Java 21, Spring Boot | Definido |
| Framework de IA | Spring AI (ChatClient, Advisors, suporte a MCP e RAG) | Proposto |
| Banco de dados | PostgreSQL | Proposto |
| Banco vetorial | pgvector (extensão do PostgreSQL) | Proposto |
| Ambiente local | Docker Compose | Proposto |
| Versionamento | Git + GitHub | Proposto |
| LLM | Ollama (local e gratuito); chat: qwen3:8b (alternativa: llama3.1:8b); embeddings: nomic-embed-text | Definido |
| Machine Learning (Parte B) | Python, Jupyter Notebooks (cadernos de código interativos), scikit-learn, Hugging Face Transformers e Datasets, Unsloth (ferramenta de fine-tuning que economiza memória da placa de vídeo) | Definido |
| Publicação de modelos e dados | Hugging Face (conta gratuita, repositórios públicos) | Definido |
| Ambiente de treino | WSL2 (Windows Subsystem for Linux, um Linux dentro do Windows) | Definido |

**Decisões tomadas**

- [ ] Nome: AisleSmart ("aisle" = corredor de supermercado, "smart" = inteligente)
- [ ] Segmento: loja de alimentos
- [ ] LLM: Ollama, local e sem custo; modelos escolhidos para a RTX 4060 (8 GB de VRAM, a memória da placa de vídeo) no Windows, com 32 GB de RAM
- [ ] Arquitetura: começar com um monólito (um único back-end Spring Boot bem organizado); o servidor MCP ficará dentro do próprio back-end
- [ ] Login: usuário fixo de teste no MVP; login com senha fica para depois
- [ ] Demo: vídeo de demonstração + código no GitHub, sem custo de servidor
- [ ] Parte B: Python e ferramentas de Machine Learning aprovadas
- [ ] Kaggle: fora do projeto, por falta de tempo

A versão estável mais recente do Spring AI indicada na documentação oficial é a 2.0.1 ([documentação](https://docs.spring.io/spring-ai/reference/2.1/api/mcp/mcp-server-boot-starter-docs.html)); confirmar ao criar o projeto.

## 9. Métricas de sucesso

O MVP está pronto quando todas as 10 histórias passam nos critérios de aceite e o assistente acerta pelo menos 18 das 20 perguntas de teste.

| Métrica | Meta | Como medir |
| --- | --- | --- |
| Histórias concluídas | 10 de 10 | Checklist da seção 5 |
| Acerto do assistente | 18 de 20 perguntas (90%) | Conjunto de perguntas de teste |
| Produtos inventados | Zero | Conferir cada sugestão contra o catálogo |
| Ações sem confirmação | Zero | Testes da HU-08 |
| Projeto explicável | Oliveira consegue explicar cada parte em 5 minutos | Ensaio de apresentação |
| Portfólio | README com diagrama, prints e vídeo de demo | Repositório no GitHub |
| Modelo e dados publicados | 1 modelo ajustado + 1 conjunto de dados no Hugging Face | Páginas no Hugging Face com documentação |
| Pesquisa e artigo | Comparação de modelos com métricas + 1 artigo publicado | Relatório no GitHub e link do artigo |

## 10. Riscos e cuidados

O maior risco de um primeiro projeto com IA é a IA gerar muito código que Oliveira ainda não entende.

| Risco | Como evitar |
| --- | --- |
| Código gerado que não entendo | Pedir à IA uma etapa pequena por vez e pedir que explique o código antes de seguir |
| Escopo crescer demais | Seguir a lista "Fora do MVP"; ideias novas vão para uma lista de futuro |
| Fine-tuning não caber na placa de vídeo | Usar modelo de cerca de 4B parâmetros com QLoRA e textos curtos; fechar outros programas durante o treino |
| Assistente inventando respostas | Regras da seção 7 + perguntas de teste a cada mudança |
| Versões de bibliotecas mudando | Conferir a documentação oficial antes de cada etapa |
| Prazo do processo seletivo | Priorizar o que fica visível: repositório com os documentos, depois a Parte A básica, depois a Parte B |

## 11. Glossário (siglas e termos)

- **Agente:** um LLM com um papel, instruções e ferramentas, que decide os passos para cumprir uma tarefa
- **API (Application Programming Interface):** interface de programação; forma de um sistema conversar com outro por requisições
- **Back-end / Front-end:** a parte que roda no servidor / a parte que aparece no navegador
- **Chunk:** pedaço de um documento usado no RAG
- **Dataset:** conjunto de dados usado para treinar ou testar um modelo
- **Embeddings:** números que representam o significado de um texto, usados para buscar por semelhança
- **Fine-tuning:** ajuste fino; treinar um modelo já pronto com exemplos próprios para uma tarefa específica
- **GGUF:** formato de arquivo de modelo usado pelo Ollama
- **HU (História de Usuário):** descrição de uma funcionalidade do ponto de vista de quem usa
- **IA (Inteligência Artificial):** sistemas que realizam tarefas que normalmente exigem inteligência humana
- **JSON (JavaScript Object Notation):** formato de dados em texto, fácil de ler por pessoas e programas
- **LLM (Large Language Model):** modelo de linguagem grande; o "cérebro" que entende e escreve texto
- **MCP (Model Context Protocol):** protocolo de contexto de modelo; padrão aberto para expor ferramentas e dados a aplicações de IA
- **ML (Machine Learning):** aprendizado de máquina; programas que aprendem padrões a partir de dados
- **Model card:** página de documentação de um modelo no Hugging Face
- **MVP (Minimum Viable Product):** produto mínimo viável; a menor versão que já entrega valor
- **PRD (Product Requirements Document):** documento de requisitos do produto
- **Prompt:** o texto de instrução enviado ao LLM
- **QLoRA (Quantized Low-Rank Adaptation):** técnica de fine-tuning que comprime o modelo e treina só uma pequena parte dele, economizando memória
- **RAG (Retrieval-Augmented Generation):** geração aumentada por recuperação; buscar trechos relevantes antes de o LLM responder
- **Seed:** script que preenche o banco com dados de exemplo
- **Skill:** arquivo de instruções que ensina a IA de código a seguir os padrões do projeto
- **Stack:** conjunto de tecnologias usado no projeto
- **Streaming:** resposta enviada aos poucos, enquanto é gerada
- **VRAM (Video RAM):** memória da placa de vídeo; limita o tamanho do modelo que cabe nela
- **WSL2 (Windows Subsystem for Linux):** recurso que roda um Linux dentro do Windows

## 12. Próximos documentos

1. 02-Arquitetura: componentes, como se comunicam e onde cada um roda
2. 03-Modelo de dados: tabelas e relacionamentos
3. 04-Contratos de API: endpoints (endereços) da loja e do chat
4. 05-Ferramentas MCP: nome, entrada e saída de cada ferramenta
5. 06-Agentes: papel, instruções e ferramentas de cada agente
6. 07-Machine Learning: dados, modelos, treino, avaliação e publicação (Parte B)
7. Skills para a IA de código (CLAUDE.md e skills por camada)

## Fontes

- [Spring AI: MCP Server Boot Starter (documentação oficial)](https://docs.spring.io/spring-ai/reference/2.1/api/mcp/mcp-server-boot-starter-docs.html)
- [Spring AI: Getting Started with MCP (documentação oficial)](https://docs.spring.io/spring-ai/reference/guides/getting-started-mcp.html)
- [Spring AI 2.0 RC: tool calling, MCP e RAG em Java (artigo técnico, em chinês)](https://gitcode.csdn.net/6a2a18b6662f9a54cb7ca98d.html)
- [Programa de estágio GenAI do iFood: trilhas e etapas (Seja Trainee)](https://sejatrainee.com.br/estagio-ifood-genai-vagas-abertas/)
- [Hugging Face: limites de armazenamento (documentação oficial)](https://huggingface.co/docs/hub/repositories-recommendations)
- [Hugging Face: usar o Ollama com modelos GGUF (documentação oficial)](https://huggingface.co/docs/hub/ollama)
- [Fine-tuning com Unsloth em GPU de consumo (DEV Community)](https://dev.to/jovan_chan_9500711396d4e6/fine-tuning-llama3-unsloth-2026-4odd)
- [Guia do Unsloth Studio: modelos por VRAM (InsiderLLM)](https://insiderllm.com/guides/unsloth-studio-setup-guide/)
