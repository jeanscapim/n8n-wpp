# Integrando o WhatsApp ao Google Calendar através do N8N

## Introdução

Este tutorial irá guiar você através do processo de instalação e configuração da nossa estrutura para criação de um Agente de IA Local, incluindo os componentes:

- **WAHA**: API de WhatsApp gratuita
- **N8N**: Plataforma de automação
- **Redis**: Banco de dados em memória para registro de chats

## Pré-requisitos

- Ter o Docker Compose ou Docker Desktop instalado
- Sistema operacional Windows 10, Windows 11 ou MacOs

## Passo a Passo

### 1. Faça o download deste repositório em seu computador
Acesse a opção "<> Code" e escolha a opção "Download ZIP"

### 2. Extraia os arquivos e acesse a pasta raiz do projeto
Certifique-se estar na mesma pasta do arquivo "docker-compose.yml"

### 3. Abra um terminal na pasta raiz
Digite o comando: ```docker-compose up -d``` e aguarde o docker baixar e subir os containers dos aplicativos

*Esse procedimento irá baixar e instalar todos os programas localmente em seu computador*

### 4. Acessando os programas:
Você poderá acessar os programas no seu navegador clicando no link do container através do seu aplicativo Docker Desktop ou através do passo a passo abaixo:

#### 4.1 Configurando o Waha (Parte 1)

Acesse o portal através do link http://localhost:3000/dashboard/

Na área de "Sessions" clique em "Start" para inicializar o serviço

Conecte seu WhatsApp no WAHA através do QR Code gerado

#### 4.2 Configurando o N8N

Acesse o portal através do link http://localhost:5678/setup e realize o cadastro do seu perfil

Instale os nodes do WAHA no N8N clicando em:
- Clique em "Settings"
- Clique em "Community nodes"
- Clique em "Install a community node"
- No campo "npm Package Name" digite: ```n8n-nodes-waha```
- Aceite os termos
- Clique em "Install"

#### 4.3 Criando um workflow no N8N (Parte 1)

Acesse o portal através do link: http://localhost:5678/home/workflows

Na página inicial do N8N crie um novo workflow

Adicione um nome para o seu workflow

Adicione um trigger de "Webhook" e o configure da seguinte forma:
- Em "HTTP Method" selecione a opção "POST"
- Em "Path" adicione o texto "webhook"
- Copie a URL de teste que aparece logo acima ```http://host.docker.internal:5678/webhook-test/webhook```

#### 4.4 Configurando o Waha (Parte 2)
Retorne para  o portal através do link http://localhost:3000/dashboard

Na área de "Sessions" clique em "Configuration" para adicionar o webhook
- Clique em "+ Webhook" para adicionar um novo webhook, cole a URL copiada
- Em "Events" deixe apenas o "message"
- Clique em "Update"

#### 4.5 Criando um workflow no N8N (Parte 2)

Retorne para o portal através do link: http://localhost:5678

Clique para esperar um evento de teste e envie uma mensagem para o seu número conectado no WAHA
- Não utilize o seu número pessoal, envie através de OUTRO NÚMERO, por exemplo, de algum familiar

Após receber os dados, adicione um node "Set" conectado ao "Webhook" , salve e crie os seguintes campos:
- Renomei para "Dados"
- session = `````` {{ $json.body.session }} ``````
- chatId = ```{{ $json.body.payload.from }}```
- pushName = ```{{ $json.body.payload._data.Info.PushName }}```
- payload_id = ```{{ $json.body.payload.id }}```
- event = ```{{ $json.body.event }}```
- message = ```{{ $json.body.payload.body }}``` 
- fromMe = ```{{ $json.body.payload.fromMe }}```
Realize um teste clicando em "Execute step"

Após receber os dados, adicione um node "Switch" em frente ao node "Set", salve e crie as seguintes regras:
- Neste switch apenas faça a comparação, se o ```{{ $json.event }}``` for igual a "message"

Adicione o node do "AI Agent" e faça as configurações:
- Em "Source for Prompt" selecione a oção "Define below"
- Em "Prompt" adicione a mensagem do "Switch" ```{{ $json.message }}```
- Em "System Message" adicione o prompt ```Você é uma esteticista especializada em manicure e pedicure```

Adicione o node do "Google Gemini" em baixo ao node "AI Agent" e adicione os seguintes parametros:
- Em "Credential to connect with" clique em "+ Create a new credential"
- Pegue a chave de API gratuita do Google Gemini aqui: https://aistudio.google.com/app/apikey
- Adicione a chave gerada no campo "API KEY"
- Altere o modelo em "Model" para o modelo "models/gemini-2.0-flash"
- Em "Options" selecione a opçao "Sampling Temperature" com o valor 0,4

Adicione o node do "Redis Chat Memory" em baixo ao node "AI Agent" e adicione os seguintes parametros:
- A configuração do node do Redis Chat Memory é a seguinte:
- Em "Password" adicione o texto "default"
- Em "Host" adicione o texto "host.docker.internal"
Configure o "Session id" selecionando "Define Beloow"
- Em "Key" adicione o ID do chat dos Dados ```{{ $('Dados').item.json.chatId }}```
- Em "Session time to live" adicione o número "3600"
- Em "Context Window Length" adicione o número "10"

Adicione o node do Waha "Send Seen" em frente ao node "AI Agent" e adicione as seguintes configurações:
- Em "Parameters" clique em "+ Create a new credential"
- Em "Host URL" adicione ```http://host.docker.internal:3000```
- Em API KEY" remova o conteúdo
- Em "Session" adicione ```{{ $('Dados').item.json.session }}```
- Em "Chat Id" adicione ```{{ $('Dados').item.json.chatId }}```
- Em "Message Id" adicione ```{{ $('Dados').item.json.payload_id }}```
- Em "Participant" deixe vazio

Adicione o node do Waha "Send a text message" em frente ao node do  Waha "Send Seen" e adicione as seguintes configurações:
- Em "Session" adicione ```{{ $('Dados').item.json.session }}```
- Em "Chat Id" adicione ```{{ $('Dados').item.json.chatId }}```
- Em "Reply To" deixe vazio
- Em "Text" customize a sua resposta combinando a resposta da AI ```{{ $('AI Agent').item.json.output }}```
