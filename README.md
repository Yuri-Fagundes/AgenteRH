# Assistente RAG — Políticas Internas

Aplicação de perguntas e respostas sobre políticas internas de RH utilizando Inteligência Artificial Generativa e arquitetura RAG (Retrieval-Augmented Generation).

O usuário realiza perguntas em português por meio de uma interface de chat e recebe respostas contextualizadas a partir de informações recuperadas de um documento PDF.

## Problema resolvido

Políticas internas de empresas costumam estar distribuídas em documentos extensos, tornando a consulta manual demorada e sujeita a interpretações inconsistentes.

Este projeto propõe um assistente virtual capaz de facilitar a consulta dessas informações, permitindo que funcionários encontrem respostas de maneira rápida por meio de perguntas em linguagem natural.

A aplicação utiliza arquitetura RAG: antes de gerar uma resposta, recupera os trechos mais relevantes do documento PDF e os fornece ao modelo de linguagem como contexto.

O prompt orienta o modelo a:

- Responder em português do Brasil.
- Utilizar as informações recuperadas do documento como fonte para as respostas.
- Ser claro e objetivo.
- Informar quando não encontrar informações suficientes no contexto.

## Arquitetura simplificada

A aplicação possui duas etapas principais: preparação da base de conhecimento e recuperação de informações para geração das respostas.

### 1. Preparação do documento (indexação)

```text
┌─────────────────────────┐
│ Arquivo politica_rh.pdf │
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│ PyPDFLoader             │
│ Extração do texto       │
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│ RecursiveCharacter     │
│ TextSplitter            │
│ Chunks: 1000 caracteres │
│ Overlap: 200 caracteres │
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│ Google Embeddings       │
│ Conversão em vetores    │
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│ FAISS                   │
│ Índice vetorial         │
└─────────────────────────┘
```

### 2. Recuperação e geração das respostas

```text
┌─────────────────────────┐
│ Usuário                 │
│ Pergunta no Streamlit   │
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│ Embedding da pergunta   │
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│ FAISS                   │
│ Recuperação semântica   │
│ dos trechos relevantes  │
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│ Prompt + contexto       │
│ ChatGoogleGenerativeAI  │
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│ Resposta contextualizada│
│ Exibida no Streamlit    │
└─────────────────────────┘
```

### Fluxo principal

1. A aplicação carrega o documento `politica_rh.pdf` por meio do PyPDFLoader.
2. O texto extraído é dividido em trechos utilizando RecursiveCharacterTextSplitter, com 1.000 caracteres e sobreposição de 200 caracteres.
3. Cada trecho é convertido em embedding utilizando o modelo de embeddings do Google.
4. Os vetores são armazenados em um índice FAISS em memória.
5. O usuário envia uma pergunta pela interface Streamlit.
6. A pergunta é convertida em embedding e utilizada para recuperar os trechos mais relevantes.
7. Os trechos recuperados são enviados ao Gemini junto ao prompt.
8. A resposta gerada é apresentada na interface de chat.

## Tecnologias utilizadas

| Tecnologia | Finalidade |
|---|---|
| Python | Linguagem principal da aplicação. |
| Streamlit | Desenvolvimento da interface web de chat. |
| LangChain | Integração e composição do pipeline RAG. |
| PyPDFLoader | Carregamento e extração de texto do PDF. |
| RecursiveCharacterTextSplitter | Divisão do documento em trechos menores. |
| Google Gemini | Geração de respostas e criação de embeddings. |
| FAISS | Indexação vetorial e recuperação semântica. |
| python-dotenv | Gerenciamento de variáveis de ambiente. |

**Modelos utilizados:**

- Embeddings: `gemini-embedding-001`.
- Geração de respostas: `gemini-3.5-flash-lite`.

## Pré-requisitos

- Python 3.10 ou superior.
- Chave de API do Google Gemini.
- Acesso à internet para comunicação com os serviços do Google.

## Como executar o projeto

### 1. Clonar o repositório

```bash
git clone URL_DO_REPOSITORIO
cd NOME_DO_PROJETO
```

### 2. Criar e ativar um ambiente virtual

**Linux/macOS:**

```bash
python3 -m venv .venv
source .venv/bin/activate
```

**Windows PowerShell:**

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
```

### 3. Instalar as dependências

```bash
python -m pip install -r requirements.txt
```

### 4. Configurar a chave de API

Crie um arquivo `.env` na raiz do projeto, seguindo o modelo de `.env.example`:

```env
GOOGLE_API_KEY=sua_chave_do_google_gemini
```

Não compartilhe sua chave de API nem versione o arquivo `.env`. Ele deve estar incluído no `.gitignore`.

### 5. Executar a aplicação

Na raiz do projeto, execute:

```bash
streamlit run RH_AI.py
```

A aplicação ficará disponível no endereço:

`http://localhost:8501`

## Exemplos de perguntas e respostas

Os exemplos abaixo representam consultas relacionadas ao documento de demonstração.

### Pergunta 1 — Home office

**Pergunta:** Quantos dias de home office um colaborador elegível pode fazer por semana?

**Resposta esperada:** Colaboradores elegíveis podem trabalhar remotamente até 3 dias por semana. A presença no escritório é obrigatória às terças e quintas-feiras.

### Pergunta 2 — Auxílio home office

**Pergunta:** Qual é o valor do auxílio home office?

**Resposta esperada:** O auxílio é de R$ 200,00 por mês para colaboradores em regime híbrido e de R$ 350,00 por mês para colaboradores em regime 100% remoto, quando aprovado pela diretoria.

### Pergunta 3 — Plano de saúde

**Pergunta:** Como funciona o plano de saúde para dependentes?

**Resposta esperada:** Podem ser incluídos cônjuge ou companheiro(a) com união estável documentada, filhos naturais ou adotivos até 21 anos e filhos universitários até 24 anos, mediante comprovação semestral. O custo é de R$ 180,00 por dependente ao mês, descontado em folha.

### Pergunta 4 — Banco de horas

**Pergunta:** Qual é o limite do banco de horas?

**Resposta esperada:** O limite é de 40 horas positivas ou 20 horas negativas. A compensação deve ocorrer em até 90 dias corridos. Horas positivas acima de 40 horas são pagas no mês seguinte com adicional de 50%, conforme a política.

### Pergunta 5 — Ausência de informação

**Pergunta:** A empresa reembolsa qualquer curso que eu queira fazer?

**Comportamento esperado:** O assistente informa que não encontrou informações suficientes no contexto recuperado e orienta o usuário a consultar o RH ou verificar o manual.

## Limitações conhecidas

- A aplicação utiliza um único documento PDF como base de conhecimento.
- O índice FAISS é armazenado em memória e reconstruído durante a inicialização.
- O processamento do documento e a geração dos embeddings são realizados novamente quando a aplicação é iniciada.
- A qualidade das respostas depende da qualidade do texto extraído do PDF e da relevância dos trechos recuperados.
- Como toda aplicação baseada em LLM, o sistema pode gerar respostas incorretas. O uso de RAG e de instruções no prompt reduz riscos, mas não garante ausência de alucinações.

## Estrutura do projeto

```text
.
├── RH_AI.py
├── politica_rh.pdf
├── requirements.txt
├── .env.example
├── .gitignore
└── README.md
```
