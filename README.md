<p align="center">
  <img src="./avatar.png" width="140" alt="ZeusDeBolsoBot">
</p>

<h1 align="center">ZeusDeBolsoBot</h1>

<p align="center">
  <a href="https://t.me/ZeusDeBolsoBot">@ZeusDeBolsoBot</a>
</p>

Bot de clima no Telegram desenvolvido com n8n para o desafio de Agentes e Automação da Rocketseat. Consulta a temperatura atual na OpenWeather e utiliza o Google Gemini para criar pequenas histórias cômicas narradas por Zeus.

## Como executar

1. Importe o arquivo [`workflow-chatbot-telegram.json`](./workflow-chatbot-telegram.json) no n8n pela opção **Import from File**.
2. Configure as credenciais abaixo.
3. Salve e ative o workflow. A instância do n8n precisa ter um webhook HTTPS público para receber mensagens do Telegram.

### Credenciais

- **`TELEGRAM_BOT_TOKEN`**: crie um bot com o [@BotFather](https://t.me/BotFather), insira o token em uma credencial **Telegram API** do n8n e selecione-a nos nodes Telegram.
- **`OPENWEATHER_API_KEY`**: obtenha a [chave da OpenWeather](https://home.openweathermap.org/api_keys) e configure-a como variável de ambiente da instância do n8n. O node `OpenWeather` utiliza `{{ $env.OPENWEATHER_API_KEY }}` no parâmetro `appid`. Reinicie a instância após configurar a variável. Se o acesso a `$env` estiver bloqueado, configure `N8N_BLOCK_ENV_ACCESS_IN_NODE=false` apenas em uma instância confiável.

Não publique chaves, tokens ou arquivos `.env` com segredos reais.

### Google Gemini (opcional)

O node `Message a model`, após `Validate Success`, gera as respostas personalizadas. Para ativá-lo, obtenha uma [chave do Gemini](https://aistudio.google.com/apikey), configure a credencial **Google Gemini(PaLM) API** no n8n e selecione um modelo disponível.

Em caso de falha da IA ou resposta inválida, o node `Default Response` gera uma mensagem determinística. Para executar sem credenciais Gemini, desative `Message a model` e conecte a saída verdadeira de `Validate Success` diretamente a `Default Response`.

## Como utilizar

Envie uma cidade no formato `Cidade,UF,BR` ou `Cidade,País`.

**Entrada:** `São Paulo,SP,BR`

**Resposta ilustrativa:**
> Fui trocar uma lâmpada no Olimpo e esqueci de desligar a energia. Daí os 22°C e as trovoadas em São Paulo. Não conte pra Hera!

**Sem IA:**
> 🌤️ A temperatura em São Paulo é de 22°C.

Para testar, consulte São Paulo, Rio de Janeiro e Belo Horizonte no formato indicado. Envie também `CidadeInexistenteXYZ,SP,BR` para verificar a mensagem de erro. As temperaturas acima são ilustrativas.
