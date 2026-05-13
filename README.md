# 📱 Kotlin CryptoMonitor — Arquitetura Imperativa

Aplicativo Android desenvolvido em **Kotlin** que exibe a cotação atual do **Bitcoin (BTC)** em reais, consumindo a API pública do [Mercado Bitcoin](https://www.mercadobitcoin.net/). O projeto foi construído com foco em aprendizado da **arquitetura imperativa** no Android, utilizando XML para layout e `AppCompatActivity` para controle direto da UI.

---

## 🖥️ Funcionalidades

- Exibe o valor atual (last) do Bitcoin formatado em moeda brasileira (R$)
- Exibe a data e hora da última atualização da cotação
- Botão de atualização manual que dispara a chamada à API
- Tratamento de erros HTTP (400, 401, 403, 404) com feedback via Toast
- Tratamento de exceções de rede com mensagem descritiva

---

## 🏗️ Arquitetura

O projeto segue a abordagem **imperativa**, onde a `Activity` concentra as responsabilidades de controle de UI, chamada de rede e atualização dos componentes visuais — sem uso de ViewModel, LiveData ou StateFlow.

```
carreiras.com.github.cryptomonitor/
├── MainActivity.kt                          # Ponto de entrada: UI + lógica de negócio
├── model/
│   └── TickerResponse.kt                    # Data classes que mapeiam o JSON da API
└── service/
    ├── MercadoBitcoinService.kt             # Interface Retrofit (contrato do endpoint)
    └── MercadoBitcoinServiceFactory.kt      # Factory que constrói a instância Retrofit
```

---

## 🔄 Fluxo de Dados

```
Usuário toca em "Atualizar"
         ↓
btn_refresh → setOnClickListener → makeRestCall()
         ↓
CoroutineScope(Dispatchers.Main).launch { }
         ↓
MercadoBitcoinServiceFactory().create()     ← instancia o Retrofit
         ↓
MercadoBitcoinService.getTicker()           ← GET api/BTC/ticker/
         ↓
Response<TickerResponse>                    ← Gson desserializa o JSON
         ↓
  ┌──────────────────┬──────────────────────┐
  ✅ isSuccessful    ❌ erro HTTP            💥 Exception
  ↓                  ↓                       ↓
Formata valor       Mapeia código           Toast com
e data              (400/401/403/404)       e.message
  ↓
Atualiza lbl_value e lbl_date via findViewById
```

---

## 🧩 Componentes de Layout

O layout da tela principal é composto por arquivos XML independentes montados via `<include>`:

```
activity_main.xml
├── component_toolbar_main.xml       → Toolbar superior com título e cor primária
└── component_quote_information.xml  → Painel central com valor e data da cotação
        └── component_button_refresh.xml   → Botão "Atualizar"
```

A comunicação entre o XML e o Kotlin é feita através de `setContentView(R.layout.activity_main)` e `findViewById`, o padrão central da arquitetura imperativa.

---

## 🌐 API Utilizada

**Mercado Bitcoin — API Pública**

| Campo     | Valor                                      |
|-----------|--------------------------------------------|
| Base URL  | `https://www.mercadobitcoin.net/`          |
| Endpoint  | `GET api/BTC/ticker/`                      |
| Formato   | JSON                                       |
| Auth      | Não requerida                              |

**Exemplo de resposta:**
```json
{
  "ticker": {
    "high": "320000.00",
    "low": "305000.00",
    "vol": "12.34567",
    "last": "312500.00",
    "buy": "312000.00",
    "sell": "313000.00",
    "date": 1713358440
  }
}
```

---

## 🛠️ Tecnologias e Dependências

| Tecnologia / Biblioteca             | Versão  | Finalidade                              |
|-------------------------------------|---------|-----------------------------------------|
| Kotlin                              | —       | Linguagem principal                     |
| Android AppCompat                   | 1.7.0   | Base da Activity e Toolbar              |
| Retrofit                            | 2.9.0   | Cliente HTTP para consumo da API REST   |
| Gson Converter                      | 2.9.0   | Desserialização automática do JSON      |
| Kotlin Coroutines (Android)         | 1.5.2   | Chamadas assíncronas sem bloquear a UI  |

---

## 📁 Descrição dos Arquivos

### `MainActivity.kt`
Classe central do app. Herda de `AppCompatActivity` e concentra toda a lógica:
- `onCreate` → infla o layout, configura a Toolbar e define o listener do botão
- `configureToolbar()` → define título, cor do texto e background via `colors.xml` e `strings.xml`
- `makeRestCall()` → executa a chamada à API dentro de uma coroutine, formata os dados recebidos e atualiza os `TextView`s diretamente

### `TickerResponse.kt`
Duas classes simples que espelham a estrutura JSON da API:
- `TickerResponse` → envelope com um campo `ticker: Ticker`
- `Ticker` → campos `high`, `low`, `vol`, `last`, `buy`, `sell` (`String`) e `date` (`Long` em timestamp Unix)

### `MercadoBitcoinService.kt`
Interface Retrofit com um único método `suspend fun getTicker(): Response<TickerResponse>`, anotado com `@GET("api/BTC/ticker/")`.

### `MercadoBitcoinServiceFactory.kt`
Classe responsável por construir e entregar a instância do Retrofit configurada com a `baseUrl` e o `GsonConverterFactory`.

---

## ⚠️ Características da Abordagem Imperativa

Este projeto foi desenvolvido intencionalmente com arquitetura imperativa para fins de estudo. Os pontos abaixo são características dessa abordagem:

- **Sem ViewModel** — a Activity carrega e manipula dados diretamente
- **Sem LiveData/StateFlow** — a UI é atualizada de forma manual via `TextView.text = ...`
- **`CoroutineScope` criada inline** — sem ciclo de vida gerenciado (pode vazar se a Activity for destruída durante uma chamada em andamento)
- **`MercadoBitcoinServiceFactory` instanciada a cada clique** — sem singleton ou injeção de dependência
- **`findViewById` chamado dentro da coroutine** — funciona pois roda na `Dispatchers.Main`, mas não é o padrão recomendado para projetos em produção

---

## 📚 Objetivo de Aprendizado

Este projeto é parte de um estudo comparativo entre estilos arquiteturais no Android. A abordagem imperativa serve como ponto de partida para compreender os problemas que motivaram padrões modernos como **MVVM**, **Clean Architecture** e **Jetpack Compose**.

---

## 📄 Licença

Projeto de uso educacional, baseado no repositório original [carreiras/android-crypto-monitor](https://github.com/carreiras/android-crypto-monitor).
