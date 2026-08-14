# Chatbot Telegram para previsão do tempo

Este projeto reúne um bot do Telegram, um workflow no n8n e integrações com OpenWeatherMap e Gemini para responder perguntas sobre clima. A ideia central é simples: receber uma cidade, buscar os dados meteorológicos, montar uma resposta útil e devolver tudo ao usuário em poucos segundos.

## O que o projeto faz

O fluxo criado permite que o bot:

- receba mensagens com nomes de cidades brasileiras;
- consulte informações atuais de clima via OpenWeatherMap;
- organize os dados em uma estrutura mais legível;
- gerar respostas mais humanas com Gemini, quando a integração estiver disponível;
- devolver ao usuário temperatura, sensação térmica, umidade, vento e uma dica prática.

## Stack principal

- n8n para orquestrar o workflow (2.9.4)
- Telegram Bot API para comunicação com o usuário;
- OpenWeatherMap API para obtenção dos dados climáticos;
- Google Gemini para transformar os dados em mensagens mais amigáveis;
- ngrok para expor o webhook localmente com HTTPS.

## Estrutura do repositório

```text
chatbot-telegram/
├── .env                          # Variáveis de ambiente
├── .env.example                  # Modelo de configuração
├── .gitignore                    # Arquivos excluídos do versionamento
├── workflow-chatbot-telegram.json # Workflow exportado do n8n
└── README.md                     # Documentação do projeto
```

## Requisitos

Antes de começar, você precisará de:

1. um token de bot do Telegram criado no [@BotFather](https://t.me/botfather);
2. uma chave da OpenWeatherMap;
3. uma chave do Google Gemini, se quiser usar a geração de texto com IA;
4. um token do ngrok, caso queira expor o webhook localmente.

## Preparando o ambiente

### 1. Entrar na pasta do projeto

```bash
cd chatbot-telegram
```

### 2. Criar o arquivo de configuração

```bash
cp .env.example .env
```

### 3. Ajustar as variáveis

Edite o arquivo `.env` com os valores corretos. A configuração mínima inclui:

```bash
N8N_ENCRYPTION_KEY=chavePessoal
N8N_PORT=5678
N8N_PROTOCOL=http
N8N_HOST=localhost
WEBHOOK_URL=http://localhost:5678/
GENERIC_TIMEZONE=America/Sao_Paulo

NGROK_AUTHTOKEN=ngrokTokenPessoal
TELEGRAM_BOT_TOKEN=telegramTokenPessoal
OPENWEATHER_API_KEY=openWeatherToken
```

### Iniciar o n8n localmente

Para executar o n8n em desenvolvimento, rode:

```bash
npx n8n start
```

### Expor localmente (ngrok / localtunnel) e fornecer a API key

O n8n precisa de um `WEBHOOK_URL` público (HTTPS) para registrar webhooks no Telegram, e do `OPENWEATHER_API_KEY` disponível no processo que inicia o n8n. Como você está rodando sem Docker, a forma mais simples é exportar as variáveis no shell antes de iniciar o n8n.

- Usando ngrok (recomendado):

```bash
# inicie o ngrok (porta 5678)
ngrok http 5678
# copie o URL HTTPS que o ngrok gerar, ex: https://abcd1234.ngrok.io
```

- Em seguida, inicie o n8n no mesmo terminal (ou exporte as variáveis antes):

```bash
export OPENWEATHER_API_KEY="sua_chave_aqui"
export WEBHOOK_URL="https://abcd1234.ngrok.io/"
npx n8n start
```

ou em apenas um comando:

```bash
OPENWEATHER_API_KEY="sua_chave_aqui" WEBHOOK_URL="https://abcd1234.ngrok.io/" npx n8n start
```

- Alternativa sem conta (localtunnel):

```bash
npx localtunnel --port 5678
# ou com subdomínio opcional
npx localtunnel --port 5678 --subdomain meu-subdominio
```

Notas importantes:
- Garanta que o túnel (ngrok/localtunnel) esteja rodando **antes** de ativar o workflow no n8n — o `WEBHOOK_URL` precisa estar definido no processo do n8n quando você ativa o workflow, pois o n8n registra o webhook junto ao Telegram durante a ativação.
- Não coloque a `OPENWEATHER_API_KEY` diretamente no `workflow-chatbot-telegram.json` (evite vazar chaves no repositório). Prefira variáveis de ambiente ou credenciais do n8n.



### Variáveis principais

`TELEGRAM_BOT_TOKEN` autenticação do bot no Telegram
`OPENWEATHER_API_KEY` consulta dos dados meteorológicos
`N8N_ENCRYPTION_KEY` proteção das credenciais internas do n8n


## Importando o workflow no n8n

### 1. Acesse a interface do n8n

Depois que os serviços estiverem no ar, abra o endereço:

```text
http://localhost:5678
```

### 2. Crie a conta inicial

Na primeira abertura, o n8n pedirá para criar um usuário administrador. Complete esse processo antes de continuar.

### 3. Importe o workflow

1. No painel do n8n selecione "Import from File".
3. Importe o arquivo `workflow-chatbot-telegram.json`.

### 4. Configure as credenciais

#### Telegram

1. Abra o nó "Entrada do Telegram".
2. Crie uma credencial.
3. Informe o `TELEGRAM_BOT_TOKEN` obtido via @BotFather.
4. Salve a configuração.

Repita o processo para os nós "Enviar Resposta" e "Aviso de Erro".

#### OpenWeatherMap

1. Abra o nó "Consulta Climática OpenWeather".
2. Crie uma nova credencial do tipo "OpenWeatherMap API".
3. Informe a `OPENWEATHER_API_KEY`e salve.

#### Google Gemini

1. Abra o nó "Criar Mensagem IA".
2. Crie uma credencial do Gemini.
3. Informe a chave da API, se quiser usar a geração de texto com IA.
4. Salve a configuração.


### 5. Ative o workflow

No canto superior direito, ative o workflow para registrar o webhook do Telegram.

## Como o fluxo funciona

O workflow segue uma sequência simples:

1. o Telegram recebe a mensagem do usuário;
2. o nó "Processar Dados" prepara os dados para o processamento;
3. a API do OpenWeatherMap traz as informações do clima;
4. o nó "Validar Resposta" valida o resultado;
5. o nó "Criar Mensagem IA" monta uma resposta mais natural, quando disponível;
6. o nó "Enviar Resposta" envia a resposta de volta ao usuário.

## Testando o chatbot

Depois do workflow estar ativo, você pode conversar com o bot no Telegram e enviar mensagens como:

```text
São Paulo, SP
```

ou

```text
Porto Alegre, RS
```

A resposta deve trazer dados do clima e uma mensagem com contexto prático.

## Troubleshooting

Alguns problemas comuns podem ser resolvidos verificando:

- se o arquivo `.env` está preenchido corretamente;
- se o token do Telegram está válido;
- se a chave do OpenWeatherMap está ativa;
- se o workflow foi ativado no n8n;
- se o `WEBHOOK_URL` aponta para o endpoint correto.

