# ZeusDeBolsoBot

Bot de clima no Telegram desenvolvido com n8n para o desafio prático de IA e Automação da Rocketseat. Consulta dados meteorológicos em tempo real na OpenWeather e usa o Google Gemini para transformá-los em pequenas histórias cômicas narradas por Zeus, o senhor dos céus.

O Gemini é opcional: o workflow possui uma resposta determinística com cidade e temperatura, sem depender de IA.

**Telegram:** [@ZeusDeBolsoBot](https://t.me/ZeusDeBolsoBot)

## Tecnologias

- [n8n](https://n8n.io/) — orquestração do workflow e tratamento de erros.
- [Telegram Bot API](https://core.telegram.org/bots/api) — recebimento e envio de mensagens.
- [OpenWeather](https://openweathermap.org/api/current) — dados meteorológicos atuais.
- [Google Gemini](https://ai.google.dev/) — geração opcional de respostas personalizadas.
- JavaScript — normalização, validação e resposta determinística.

## Como funciona

1. O Telegram Trigger recebe a mensagem e o node `Input` salva a cidade na variável `queue`.
2. A entrada é validada e consultada na OpenWeather, com temperatura em Celsius e idioma PT-BR.
3. O workflow valida os dados meteorológicos e envia uma mensagem de erro se a consulta falhar.
4. O node `Message a model` (Gemini), após `Validate Success`, gera uma resposta na personalidade de Zeus.
5. Se a IA falhar ou retornar uma saída não reconhecida, o node `Default Response` gera a mensagem padrão. A resposta é enviada ao chat original.

## Como executar

### 1. Importe o workflow

Tenha uma instância do n8n com suporte aos nodes Telegram, HTTP Request, Code e Google Gemini. No editor do n8n, crie um workflow, abra o menu de opções e selecione **Import from File**. Importe o arquivo [`workflow-chatbot-telegram.json`](./workflow-chatbot-telegram.json).

Para receber mensagens reais, a instância deve possuir um webhook público HTTPS acessível pelo Telegram. Mantenha o workflow desativado até concluir a configuração.

### 2. Configure as credenciais

O desafio define os seguintes nomes para os segredos:

| Segredo | Finalidade |
| --- | --- |
| `TELEGRAM_BOT_TOKEN` | Token do bot criado no Telegram. |
| `OPENWEATHER_API_KEY` | Chave da API de clima. |

**Telegram:** crie um bot com o [@BotFather](https://t.me/BotFather). No n8n, crie uma credencial do tipo **Telegram API**, informe o valor de `TELEGRAM_BOT_TOKEN` no campo Access Token e selecione essa credencial em todos os nodes Telegram, incluindo o Trigger. Não é necessário colocar o token diretamente no JSON.

**OpenWeather:** obtenha uma chave em [API Keys](https://home.openweathermap.org/api_keys) e defina a variável de ambiente `OPENWEATHER_API_KEY` no ambiente em que o n8n é executado. Se utilizar Docker Compose, adicione ao serviço existente do n8n:

```yaml
environment:
  OPENWEATHER_API_KEY: ${OPENWEATHER_API_KEY}
```

No arquivo `.env` local (não versionado), informe:

```dotenv
OPENWEATHER_API_KEY=insira_sua_chave_aqui
```

Recrie o container para aplicar a configuração. Se utilizar outra forma de instalação, defina a variável no ambiente do processo do n8n e reinicie-o.

O node `OpenWeather` utiliza **Authentication: None** e **Send Query Parameters** habilitado:

| Parâmetro | Valor |
| --- | --- |
| `q` | `{{ $json.queue }}` |
| `units` | `metric` |
| `lang` | `pt_br` |
| `appid` | `{{ $env.OPENWEATHER_API_KEY }}` |

A chave é obtida em tempo de execução, sem ser inserida diretamente no workflow. Caso o acesso a `$env` esteja bloqueado, configure `N8N_BLOCK_ENV_ACCESS_IN_NODE=false` somente em uma instância confiável. Essa configuração permite que editores de workflows acessem variáveis de ambiente. Consulte a [documentação de segurança do n8n](https://docs.n8n.io/hosting/configuration/environment-variables/security/).

### 3. Configure o Gemini (opcional)

Obtenha uma chave no [Google AI Studio](https://aistudio.google.com/apikey), crie uma credencial **Google Gemini(PaLM) API** no n8n e selecione-a no node `Message a model`. Escolha um modelo disponível na sua conta. O prompt já está configurado para responder em português, com saída JSON contendo `message` e `ok`.

**Sem Gemini:** desative o node `Message a model` e substitua a conexão da saída verdadeira de `Validate Success` por uma conexão direta com `Default Response`, que já está conectado a `Send Response`. Assim, o workflow pode ser executado sem credenciais ou custos de IA, inclusive em uma instalação nova. Para reativar a IA, restaure a conexão original e habilite o node com uma credencial válida.

### 4. Execute e teste

Salve e ative/publique o workflow. Abra seu bot no Telegram e envie uma cidade no formato `Cidade,UF,BR` ou `Cidade,País`.

Exemplo de entrada:

```text
São Paulo,SP,BR
```

Exemplo ilustrativo de resposta do Zeus:

> Fui trocar uma lâmpada no Olimpo e esqueci de desligar a energia. Daí os 22°C e as trovoadas em São Paulo. Não conte pra Hera!

Sem IA, a resposta segue o formato:

> 🌤️ A temperatura em São Paulo é de 22°C.

Para testar o workflow, consulte `São Paulo,SP,BR`, `Rio de Janeiro,RJ,BR` e `Lisboa,PT`. Em seguida, envie `CidadeInexistenteXYZ,SP,BR` e confira a mensagem de erro:

> ❌ Cidade não encontrada. Use o formato Cidade,UF,BR (ex.: São Paulo,SP,BR).

Os valores exibidos são ilustrativos. Compare a temperatura real com o retorno da OpenWeather e repita o teste com o Gemini desativado. Se a busca de uma cidade brasileira com UF falhar, tente `Cidade,BR`: a OpenWeather documenta a busca por estado apenas para os Estados Unidos.

## Segurança

Mantenha a chave da OpenWeather em uma variável de ambiente e os tokens do Telegram e Gemini no gerenciador de credenciais do n8n. Configure esses segredos novamente após a importação em outra instância. Não publique tokens, chaves de API ou arquivos `.env` com valores reais. Use um bot próprio para testar a importação, evitando conflitos de webhook com o bot original.

---

Projeto desenvolvido como parte do desafio **Bot de Clima no Telegram com N8N**, da Rocketseat.
