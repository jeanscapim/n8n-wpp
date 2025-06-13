# 🤖 Agente de IA para WhatsApp com N8N

## 📋 Visão Geral

Este guia te ajudará a criar um assistente virtual inteligente que responde automaticamente mensagens no WhatsApp. Nosso agente utiliza:

- **🔗 WAHA**: Conecta seu WhatsApp à automação
- **⚡ N8N**: Plataforma que automatiza todo o processo
- **🧠 Google Gemini**: Inteligência artificial para respostas
- **💾 Redis**: Memória para conversas contextuais

---

## 🛠️ Pré-requisitos

Antes de começar, certifique-se de ter:

- [ ] **Docker Desktop** ou **Docker Compose** instalado
- [ ] **Windows 10/11** ou **macOS**
- [ ] **Conta Google** (para API do Gemini - gratuita)
- [ ] **Número de WhatsApp secundário** para testes

> ⚠️ **Importante**: Use um número secundário para testes, não seu WhatsApp pessoal!

---

## 🚀 Instalação e Execução

### Passo 1: Download do Projeto
1. Acesse o repositório do projeto
2. Clique em **"<> Code"** → **"Download ZIP"**
3. Extraia os arquivos em uma pasta de sua escolha
4. Navegue até a pasta que contém o arquivo `docker-compose.yml`

### Passo 2: Inicialização
1. Abra o **terminal/prompt** na pasta do projeto
2. Execute o comando:
   ```bash
   docker-compose up -d
   ```
   *Esse comando iniciará localmente os serviços: WAHA, N8N e Redis.*
3. Aguarde o download e instalação (pode demorar alguns minutos)

---

### 🌐 Acesso aos serviços

| Serviço        | URL                                                                |
| -------------- | ------------------------------------------------------------------ |
| WAHA Dashboard | [http://localhost:3000/dashboard](http://localhost:3000/dashboard) |
| N8N Portal     | [http://localhost:5678](http://localhost:5678)                     |

---

### ⚙️ Configuração

### 🔧 1. Configurando o WAHA (Conexão WhatsApp)

1. **Acesse**: http://localhost:3000/dashboard/
2. Na seção **"Sessions"**, clique em **"Start"**
3. **Escaneie o QR Code** com seu WhatsApp
4. Aguarde a confirmação da conexão

### 🔧 2. Configurando o N8N (Automação)

1. **Acesse**: http://localhost:5678/setup
2. **Crie sua conta** no N8N
3. **Instale o plugin WAHA**:
   - Vá em **Settings** → **Community nodes** → **Install a community node**
   - Digite: `n8n-nodes-waha`
   - Aceite os termos e clique em **"Install"**

### 🔧 3. Criando o Workflow Inteligente

#### 3.1 Configuração Inicial
1. **Acesse**: http://localhost:5678/home/workflows
2. Clique em **"+ Novo Workflow"**
3. Dê um nome ao seu workflow (ex: "Assistente WhatsApp")

#### 3.2 Configurando o Webhook (Receptor de Mensagens)
1. Adicione um nó **"Webhook"**
2. Configure:
   - **HTTP Method**: POST
   - **Path**: webhook
3. **Copie a URL de teste** que aparece (será algo como: http://host.docker.internal:5678/webhook-test/webhook)

#### 3.3 Conectando WAHA ao N8N
1. **Volte ao WAHA**: http://localhost:3000/dashboard
2. Na seção **"Sessions"**, clique em **"Configuration"**
3. Clique em **"+ Webhook"**
4. **Cole a URL** copiada do N8N
5. Em **"Events"**, deixe apenas **"message"** marcado
6. Clique em **"Update"**

#### 3.4 Testando a Conexão
1. **No N8N**, clique em **"Listen for Test Event"** no nó Webhook
2. **Envie uma mensagem** para o número conectado no WAHA (use outro número!)
3. Verifique se os dados chegaram no N8N

### 🔧 4. Configurando a Inteligência Artificial

#### 4.1 Processamento de Dados
Adicione um nó **"Set"** após o Webhook e configure:
```
Nome: "Dados"
Campos:
- session = {{ $json.body.session }}
- chatId = {{ $json.body.payload.from }}
- pushName = {{ $json.body.payload._data.Info.PushName }}
- payload_id = {{ $json.body.payload.id }}
- event = {{ $json.body.event }}
- message = {{ $json.body.payload.body }}
- fromMe = {{ $json.body.payload.fromMe }}
```
Realize um teste clicando em **"Execute step"**

#### 4.2 Filtro de Mensagens
Adicione um nó **"Switch"** com a condição:
- Se `{{ $json.event }}` igual a `"message"`

#### 4.3 Configurando o Google Gemini
1. **Obtenha sua API Key**: https://aistudio.google.com/app/apikey
2. Adicione o nó **"Google Gemini"** com:
   - **API Key**: Cole sua chave gratuita
   - **Model**: `models/gemini-2.0-flash`
   - **Temperature**: 0.4

#### 4.4 Configurando o Agente IA
Adicione o nó **"AI Agent"** com:
- **Prompt**: `{{ $json.message }}`
- **System Message**: 
  ```
  Você é uma esteticista especializada em manicure e pedicure. 
  Seja prestativa, profissional e amigável em suas respostas.
  ```

#### 4.5 Memória de Conversas (Redis)
Adicione o nó **"Redis Chat Memory"** com:
- **Password**: `default`
- **Host**: `host.docker.internal`
- **Session ID**: `{{ $('Dados').item.json.chatId }}`
- **TTL**: `3600` (1 hora)
- **Context Window**: `10` mensagens

### 🔧 5. Enviando Respostas

#### 5.1 Marcar como Visualizada
Adicione o nó **"WAHA Send Seen"** com:
- **Host URL**: `http://host.docker.internal:3000`
- **API KEY**: remova o conteúdo do campo
- **Session**: `{{ $('Dados').item.json.session }}`
- **Chat ID**: `{{ $('Dados').item.json.chatId }}`
- **Message ID**: `{{ $('Dados').item.json.payload_id }}`

#### 5.2 Enviar Resposta
Adicione o nó **"WAHA Send Text Message"** com:
- **Session**: `{{ $('Dados').item.json.session }}`
- **Chat ID**: `{{ $('Dados').item.json.chatId }}`
- **Reply To**: remova o conteúdo do campo
- **Text**: `{{ $('AI Agent').item.json.output }}`

---

## ✅ Testando o Sistema

1. **Ative o workflow** no N8N
2. **Envie uma mensagem** de outro número para o WhatsApp conectado
3. **Aguarde a resposta** automática do agente IA
4. **Teste diferentes perguntas** para verificar a personalidade configurada

---

## 🎯 Dicas de Personalização

### Personalizando o Agente
Modifique o **System Message** do AI Agent para diferentes especialidades:

```
🏥 Atendimento Médico:
"Você é um assistente de uma clínica médica. Seja educado e direcione para agendamentos."

🍕 Restaurante:
"Você é um garçom virtual de uma pizzaria. Ajude com pedidos e informações do cardápio."

🛒 E-commerce:
"Você é um consultor de vendas online. Ajude clientes com dúvidas sobre produtos."
```

### Melhorando as Respostas
- **Temperature baixa (0.2-0.4)**: Respostas mais focadas
- **Temperature alta (0.7-0.9)**: Respostas mais criativas
- **Context Window maior**: Memória mais longa das conversas

---

## 🔧 Solução de Problemas

### Problemas Comuns

| Problema | Solução |
|----------|---------|
| WhatsApp desconecta | Reconecte escaneando o QR Code novamente |
| N8N não recebe mensagens | Verifique se o webhook está configurado corretamente |
| IA não responde | Verifique sua API Key do Google Gemini |
| Erro de conexão Redis | Reinicie os containers: `docker-compose restart` |

### Logs e Monitoramento
- **N8N**: Veja execuções em http://localhost:5678/executions
- **WAHA**: Monitor em http://localhost:3000/dashboard
- **Docker**: Use `docker-compose logs` para ver logs detalhados

---

## 🎉 Próximos Passos

Agora que seu agente está funcionando, você pode:

1. **📅 Integrar com Google Calendar** para agendamentos
2. **📊 Adicionar analytics** para métricas de conversas  
3. **🔄 Criar workflows múltiplos** para diferentes tipos de negócio
4. **📱 Conectar múltiplos WhatsApp** para escalar o atendimento

---

## 📞 Suporte

Se encontrar dificuldades:
- Verifique se todos os containers estão rodando: `docker ps`
- Reinicie o sistema: `docker-compose down && docker-compose up -d`
- Consulte os logs específicos de cada serviço

**🎯 Pronto! Seu agente de IA está funcionando e respondendo automaticamente no WhatsApp!**
