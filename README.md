# Bot de Clima no Telegram (n8n Workflow)

Este repositório contém um fluxo de trabalho (workflow) do **n8n** que implementa um chatbot para o Telegram. O bot consulta a previsão do tempo para uma cidade fornecida pelo usuário e utiliza Inteligência Artificial (Google Gemini) para gerar uma resposta meteorológica amigável, natural e personalizada.

## 📋 Descrição do Chatbot

O fluxo funciona da seguinte maneira:
1. **Telegram Trigger**: O bot recebe uma mensagem de texto enviada pelo usuário no Telegram contendo o nome de uma cidade (ex: *São Paulo, BR* ou *Paris, FR*).
2. **Tratamento de Dados (Set/Edit Fields)**: A mensagem é normalizada (removendo acentos e espaços extras) para garantir uma busca precisa.
3. **Consulta de Clima (HTTP Request)**: É feita uma chamada à API do **OpenWeather** para obter as condições climáticas atuais em tempo real.
4. **Validação (If)**: O fluxo verifica se a cidade existe e se os dados foram retornados com sucesso:
   - **Caso de Sucesso**:
     - Os dados são formatados em JavaScript.
     - Um modelo de IA (**Google Gemini**) reescreve a mensagem meteorológica de forma descontraída, adicionando emojis adequados ao clima e informando a temperatura e a sensação térmica atual.
     - O bot responde ao usuário com o texto humanizado gerado pela IA (ou um modelo padrão em caso de falha da IA).
   - **Caso de Falha**:
     - O bot envia uma mensagem de erro instruindo o usuário a tentar novamente informando o nome correto da cidade.

---

## 🚀 Como Importar o Fluxo no n8n

Siga os passos abaixo para importar o fluxo para a sua instância do n8n:

1. Baixe o arquivo do workflow deste repositório: [workflow-telegram-chatbot.json].
2. Abra a interface do seu **n8n**.
3. No painel lateral, clique em **Workflows** e depois em **Add workflow** (ou abra um workflow vazio).
4. No canto superior direito da tela de edição do fluxo, clique nos **três pontos (...)** e selecione **Import from file**.
5. Selecione o arquivo `workflow-telegram-chatbot.json` baixado.
6. O fluxo será carregado na tela. Lembre-se de clicar em **Save** para salvar no seu ambiente.

---

## 🔑 Configuração de Credenciais e Variáveis

Para que o fluxo funcione corretamente, você precisará configurar as credenciais do Telegram e do OpenWeather, além da API Key do Google Gemini.

### 1. Telegram (`TELEGRAM_BOT_TOKEN`)

Você precisa criar um bot no Telegram e configurar suas credenciais no n8n.

1. No Telegram, fale com o [@BotFather](https://t.me/BotFather) e envie o comando `/newbot`.
2. Siga as instruções para dar um nome e um username ao seu bot.
3. O `@BotFather` gerará um token de acesso (que representará o seu **`TELEGRAM_BOT_TOKEN`**).
4. No **n8n**:
   - Abra as configurações de qualquer um dos nós do Telegram (como o *Telegram Trigger* ou *Send a text message*).
   - Na seção **Credential for Telegram API**, selecione **- Create New Credential -** (ou selecione uma existente se já possuir).
   - Insira o token gerado pelo BotFather no campo **Access Token**.
   - Salve a credencial com um nome amigável (por exemplo, `Telegram account`).

### 2. OpenWeather (`OPENWEATHER_API_KEY`)

O nó de consulta de clima utiliza uma variável de ambiente do sistema para autenticar as requisições na API do OpenWeather de forma segura.

1. Crie uma conta gratuita no portal do [OpenWeather](https://openweathermap.org/).
2. Vá até a seção de chaves de API e gere a sua chave (esta será a **`OPENWEATHER_API_KEY`**).
3. **No ambiente onde o n8n está rodando** (Docker, PM2, máquina local, etc.), defina a seguinte variável de ambiente:
   ```env
   OPENWEATHER_API_KEY=sua_chave_aqui
   ```
   *Nota: O fluxo está configurado para ler automaticamente esta variável usando a expressão `{{ $env.OPENWEATHER_API_KEY }}`.*
4. Caso você não consiga definir variáveis de ambiente no servidor do seu n8n, você pode alternativamente:
   - Abrir o nó **HTTP Request** no n8n.
   - Ir até os parâmetros da query (`queryParameters`).
   - Localizar o parâmetro de nome `appid` e alterar o valor de `={{ $env.OPENWEATHER_API_KEY }}` diretamente para a sua chave de API (entre aspas ou texto simples). *Atenção: evite exportar o JSON se fizer isso para não expor a sua chave publicamente.*

### 3. Google Gemini (Opcional - para mensagens humanizadas)

Para usar a IA na reescrita das mensagens, o fluxo utiliza um **AI Agent** conectado a um modelo **Google Gemini Chat Model** e a um **Structured Output Parser**. Siga os passos para configurar a credencial do Gemini:

1. Obtenha uma chave de API no [Google AI Studio](https://aistudio.google.com/).
2. No nó **Google Gemini Chat Model** no n8n:
   - Em **Credential for Google Gemini(PaLM) API**, selecione **- Create New Credential -**.
   - Cole a sua chave de API gerada no Google AI Studio.
   - Salve a credencial.

---

## 🐳 Executando o Ambiente com Docker Compose

Para rodar todo o ecossistema localmente ou em produção (n8n editor, runners para execução segura de código JS e ngrok para expor os webhooks), você pode utilizar o arquivo [docker-compose.yml] fornecido.

### 📋 Pré-requisitos e Variáveis de Ambiente (`.env`)

Todos os dados sensíveis foram removidos do arquivo `docker-compose.yml` e devem ser definidos em um arquivo `.env` no mesmo diretório. 

1. Crie uma cópia do arquivo [.env.example] e renomeie-a para `.env`:
   ```bash
   cp .env.example .env
   ```
2. Abra o arquivo `.env` e preencha as variáveis de ambiente seguindo as orientações abaixo:

#### 🗝️ Como obter as Credenciais e Configurações:

*   **`OPENWEATHER_API_KEY`**: 
    1. Crie uma conta gratuita no portal do [OpenWeather](https://openweathermap.org/).
    2. Vá até a seção de chaves de API e gere a sua chave.
*   **`NGROK_AUTHTOKEN`**:
    1. Crie uma conta gratuita em [ngrok](https://ngrok.com/).
    2. Vá ao seu painel em **Your Authtoken** e copie a chave gerada.
*   **`NGROK_DOMAIN`** e **`WEBHOOK_URL`**:
    1. No painel do ngrok, vá para **Cloud Edge** -> **Domains**.
    2. Crie ou reserve um domínio estático gratuito (exemplo: `seu-subdominio.ngrok-free.dev`).
    3. Defina a variável `NGROK_DOMAIN` com este valor (ex: `seu-subdominio.ngrok-free.dev`).
    4. Defina a variável `WEBHOOK_URL` como `https://seu-subdominio.ngrok-free.dev`.
*   **`N8N_RUNNERS_AUTH_TOKEN`**:
    1. É uma chave secreta e aleatória para autenticar a conexão gRPC segura entre o editor n8n e o executor de tarefas (`n8n-runner`).
    2. Você pode gerar este token executando o comando no seu terminal:
       ```bash
       openssl rand -hex 32
       ```

---

### 🚀 Comandos para Inicialização

Você pode rodar os containers utilizando o terminal (CLI) ou através do Portainer integrado ao Docker Desktop.

#### Opção A: Pelo Terminal (CLI)

Com o arquivo `.env` configurado, inicialize os containers em segundo plano executando:

```bash
docker compose up -d
```

Para verificar se todos os containers estão rodando perfeitamente:

```bash
docker compose ps
```

Para monitorar os logs em tempo real:

```bash
docker compose logs -f
```

#### Opção B: Pelo Portainer (Docker Desktop)

Se você gerencia seus containers visualmente através do **Portainer**:

1. Abra o painel do seu Portainer (geralmente em `http://localhost:9000` ou `http://localhost:9443`).
2. No menu lateral esquerdo do ambiente local, clique em **Stacks** e depois em **Add stack**.
3. Dê um nome para a stack (ex: `n8n-weather-bot`).
4. Na opção de build, escolha **Web editor** e cole o conteúdo do arquivo [docker-compose.yml].
5. Logo abaixo do editor, na seção **Environment variables**:
   * Clique em **Advanced mode** (Modo Avançado).
   * Copie todo o conteúdo configurado do seu arquivo `.env` e cole diretamente no campo de texto.
6. Clique em **Deploy the stack** no final da página.
7. O Portainer criará os containers do `n8n-editor`, `n8n-runner` e `ngrok` agrupados na mesma rede automaticamente.

---

### 📌 Versões Recomendadas do n8n

*   **Imagem Base Utilizada**: `n8nio/n8n:2.30.3` (e `n8nio/runners:2.30.3` para o runner de tarefas).
*   **Recomendação**: Os executores de tarefas externos (task runners) foram integrados nativamente nas versões mais recentes (como a série `2.x` do n8n). É essencial manter as imagens do editor (`n8nio/n8n`) e dos executores (`n8nio/runners`) na mesma tag de versão (`2.30.3` neste exemplo) para garantir que a comunicação gRPC e a API interna estejam totalmente compatíveis.
