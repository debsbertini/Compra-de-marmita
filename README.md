# Dona Panela — Site Completo & Chatbot de Vendas (100% Client-Side)

Aplicação web completa e responsiva (**Landing Page + Chatbot Integrado** em HTML, CSS e JavaScript puro), desenvolvida para o restaurante **Dona Panela**.

O sistema opera **100% no navegador (client-side), sem uso de LLMs ou APIs externas**, integrando uma interface gastronômica moderna e apetitosa diretamente a um motor clássico de **Processamento de Linguagem Natural (NLP)** baseado em **Tokenização**, na arquitetura simbólica do **ELIZA** (*Joseph Weizenbaum, MIT, 1966*) e em uma **Máquina de Estados Finitos (FSM)** para gestão do pedido.

---

## 🎨 Experiência Visual e Estrutura do Site (UI/UX)

O site foi projetado para proporcionar uma jornada de compra intuitiva, acolhedora e apetitosa para qualquer cliente:

1. **Identidade Visual ("Dona Panela")**:
   - Paleta inspirada em comida caseira feita na hora: tons quentes de **terracota/laranja queimado** (`#C84B31`), **ocre dourado** (`#D97724`), fundo **creme/off-white** (`#FAF6F0`), **verde-oliva** (`#4A6B3B`) para ações de compra e tipografia escura de alto contraste (`Fraunces` + `Plus Jakarta Sans`).
   - Layout 100% responsivo (Desktop, Tablet e Mobile) com navegação suave e microinterações.
2. **Cabeçalho Fixo (Header)**:
   - Identidade do restaurante **Dona Panela**, links de navegação rápida (*Pratos de Hoje*, *Cardápio da Semana*, *Fazer Pedido*), badge de status **Aberto agora** e destaque para **Entrega fixa R$ 5,00**.
3. **Hero Section (Banner Principal)**:
   - Chamada acolhedora sobre refeições caseiras fresquinhas, destaques do serviço, imagem em alta definição com preço inicial e botão principal (CTA) **"Pedir Agora"**.
4. **Seção "Pratos de Hoje" (Dinâmica)**:
   - Detecta automaticamente o dia da semana atual do cliente (`new Date().getDay()`) e exibe em destaque os **2 pratos do dia** em cards grandes com foto em alta definição, descrição apetitosa, preço oficial do `CARDAPIO_OFICIAL` e botão **"Pedir este prato"** que rola até o chat e envia o pedido automaticamente para o motor NLP/ELIZA.
5. **Seção "Cardápio da Semana & Bebidas" (Abas Interativas)**:
   - Navegação por abas de **Segunda a Domingo + Bebidas**, marcando automaticamente o dia atual com a tag *"Hoje"* e permitindo pedir qualquer prato ou bebida com um clique.
6. **Imagens em Alta Definição com Fallback Visual (`onerror`)**:
   - Fotografias reais dos pratos com `object-fit: cover` e `loading="lazy"`.
   - Caso o usuário esteja sem internet ou alguma imagem falhe ao carregar, o evento `onerror` ativa automaticamente um **gradiente estilizado com ícone/emoji do prato**, garantindo que nenhum ícone de imagem quebrada apareça na tela.
7. **Seção de Atendimento / Chatbot Integrado + Botão Flutuante**:
   - Guia passo a passo ao lado da janela do assistente virtual, atalhos rápidos de 1 clique e botão flutuante no canto inferior direito para acessar o chat de qualquer ponto da página.
8. **Saudação Dinâmica do Chatbot por Dia da Semana**:
   - Ao iniciar ou reiniciar o atendimento, o chatbot identifica o dia atual do usuário (ex: *quinta-feira*) e já apresenta os 2 pratos do dia com seus preços oficiais, perguntando qual o cliente prefere ou se deseja consultar outro dia.

---

## 🧠 Arquitetura Técnica do Chatbot (100% Client-Side)

### 1. Pipeline Clássico de NLP (Normalização e Tokenização)
Toda mensagem digitada ou acionada pelos botões dos cards passa por um pipeline determinístico de 4 estágios:

1. **Normalização Léxica (`NLP.normalizar`)**:
   - Conversão para letras minúsculas (`toLowerCase()`).
   - Decomposição canônica Unicode (`normalize('NFD')`) e remoção de diacríticos/acentos (`[\u0300-\u036f]`).
   - Padronização de termos compostos (ex: `terça-feira` $\rightarrow$ `terca`) e limpeza de pontuação supérflua.
2. **Tokenização e Remoção de Stopwords (`NLP.tokenizar`)**:
   - Segmentação da frase em um array de tokens atômicos (palavras).
   - Filtragem de *stopwords* (artigos, preposições e conectivos irrelevantes para a extração de intenção).
3. **Mapeamento de Sinônimos e Lematização Simples (`MAPA_SINONIMOS`)**:
   - Redução de variações coloquiais e diminutivos ao token canônico do domínio:
     - `"refri"`, `"coca"`, `"latinha"` $\rightarrow$ `"refrigerante"`
     - `"franguinho"` $\rightarrow$ `"frango"`
     - `"feijuca"` $\rightarrow$ `"feijoada"`
     - `"estrogonofe"` $\rightarrow$ `"strogonoff"`
     - `"credito"`, `"debito"`, `"maquininha"` $\rightarrow$ `"cartao"`
4. **Extração de Entidades Baseada em Tokens (`NLP.extrairEntidades`)**:
   - Identificação estruturada de **dias da semana**, **pratos** do `CARDAPIO_OFICIAL`, **bebidas**, **quantidades** (algarismos ou numerais por extenso como `"duas"`), **personalizações gastronômicas** (ex: `"sem cebola"`, `"arroz integral"`, `"bem passada"`, `"feijão separado"`) e **formas de pagamento** (`Pix`, `Cartão`, `Dinheiro`).

---

### 2. Motor Inspirado no ELIZA (1966)
Inspirado no artigo clássico de Joseph Weizenbaum (*"ELIZA — A Computer Program For the Study of Natural Language Communication Between Man and Machine"*), o motor combina:

- **Regras com Prioridade (`rank` / peso)**:
  - Cada regra possui um peso numérico (`rank`). As entradas são avaliadas da maior prioridade para a menor:
    - `Rank 100`: Guardrail de imutabilidade de preços e recusa de descontos/frete grátis.
    - `Rank 95`: Guardrail de bloqueio de assuntos fora do escopo (política, programação, etc.).
    - `Rank 90`: Cancelamento/reinício do pedido.
    - `Rank 85`: Consulta de resumo e cálculo parcial/total do carrinho.
    - `Rank 82`: Validação de conflito entre prato e dia da semana ou tentativa de impor preço diferente da tabela oficial.
    - `Rank 80`: Identificação e adição de prato do cardápio oficial.
    - `Rank 78`: Desambiguação automática (ex: quando o cliente pede apenas `"frango"`, lista as 4 opções de frango da semana).
    - `Rank 75 & 72`: Consulta de cardápio por dia da semana ou cardápio completo.
- **Decomposição e Reassemblagem (Pattern Matching)**:
  - Expressões regulares decompõem a sentença do usuário capturando fragmentos livres (ex: `/(?:eu\s+)?(?:quero|gostaria\s+de|prefiro)\s+(.*)/i` ou `/(?:eu\s+)?n[aã]o\s+(?:gosto\s+de|como)\s+(.*)/i`) e os remontam em templates naturais de atendimento.
- **Reflexão Pronominal (`refletirPronomes`)**:
  - Os fragmentos capturados têm seus pronomes e verbos convertidos da 1ª pessoa para a 2ª pessoa (`"eu"` $\rightarrow$ `"você"`, `"meu/minha"` $\rightarrow$ `"seu/sua"`, `"estou"` $\rightarrow$ `"está"`), permitindo que o fallback estilo ELIZA acolha o que o cliente disse e o redirecione educadamente para as opções do cardápio da **Dona Panela**.

---

### 3. Máquina de Estados Finitos (FSM) do Pedido
O fluxo comercial é controlado por uma máquina de estados em memória que conduz o cliente até a conclusão da compra:

```text
[INICIO] ──> [ESCOLHA_PRATO] ──> [PERSONALIZACAO] ──> [BEBIDA] ──> [ENDERECO] ──> [PAGAMENTO] ──> [CONFIRMADO]
```

- **Cálculo Automático**: O sistema mantém os itens e quantidades no carrinho, calcula o subtotal exato e soma a **taxa de entrega fixa de R$ 5,00**, apresentando o resumo detalhado antes da confirmação.

---

## 🛡️ Segurança, Privacidade e Guardrails

1. **Processamento 100% Local (Sem LLMs ou Backend)**:
   - Nenhuma requisição a APIs de IA é realizada. Não há necessidade de chaves de API.
   - Interface limpa e focada exclusivamente na experiência de compra do cliente final.
2. **Encapsulamento e Integridade (`IIFE` + `Object.freeze`)**:
   - Todo o script executa dentro de uma **IIFE (Immediately Invoked Function Expression)** em modo estrito (`'use strict'`).
   - Nenhuma variável, função ou estado do carrinho fica exposto no objeto global `window`.
   - O objeto `CARDAPIO_OFICIAL` (pratos, bebidas e taxa de entrega de R$ 5,00) é protegido em múltiplos níveis com `Object.freeze()`, impedindo adulteração de preços via console.

---

## 🚀 Como Executar Localmente

Como o projeto é 100% estático e client-side, não requer instalação de pacotes nem servidor backend:

1. Abra diretamente o arquivo [`index.html`](file:///c:/Users/migue/OneDrive/Documentos/compra%20de%20marmita/index.html) em qualquer navegador moderno; **OU**
2. Utilize a extensão **Live Server** no VS Code.
