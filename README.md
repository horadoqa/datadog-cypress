# datadog-cypress

## DATADOG PARA QA COM CYPRESS

### Guia completo de monitoramento e testes

Você testa em staging com Cypress e tudo funciona perfeitamente.

Mas em produção? Aí começa a quebrar sem você saber por quê.

**Datadog** é a ferramenta que faltava para você **VER** o que está acontecendo de verdade lá.

---

## O que é Datadog?

Datadog é uma plataforma de **observabilidade** que coleta dados de tudo que acontece no seu sistema em tempo real:

* Performance da aplicação
* Erros e exceções
* Latência de requisições
* Logs de tudo que acontece
* Uso de CPU e memória
* Banco de dados lento
* Requisições falhando

### Como QA usando Cypress, você consegue:

* ✅ Rodar teste no staging (tudo OK)
* ✅ Ver em tempo real no Datadog o que está acontecendo
* ✅ Identificar gargalos (aplicação, banco, infraestrutura)
* ✅ Criar alertas automáticos
* ✅ Gerar relatórios com dados reais

---

## PASSO 1: Instalar e configurar Datadog

### Criar conta gratuita

Acesse:
👉 [https://www.datadoghq.com/free](https://www.datadoghq.com/free)

Benefícios da conta gratuita:

* 100GB/mês de logs
* 7 dias de retenção
* Dashboards básicos
* Alertas básicos
* APM grátis

Após criar a conta, você receberá uma **API_KEY**. Guarde bem.

---

### Instalar Datadog Agent

O Datadog funciona com um **agente** que coleta dados do servidor.

#### Linux (Ubuntu/Debian)

```bash
DD_AGENT_MAJOR_VERSION=7 DD_API_KEY=SUA_API_KEY DD_SITE=datadoghq.com bash -c "$(curl -L https://s3.amazonaws.com/dd-agent/scripts/install_agent.sh)"
```

#### macOS

```bash
DD_AGENT_MAJOR_VERSION=7 DD_API_KEY=SUA_API_KEY DD_SITE=datadoghq.com bash -c "$(curl -L https://s3.amazonaws.com/dd-agent/scripts/install_agent.sh)"
```

#### Verificar se está rodando

```bash
sudo systemctl status datadog-agent
```

Se aparecer `active (running)`, está funcionando ✅

---

## PASSO 2: Integrar Cypress com Datadog

### Instalar plugins

```bash
npm install --save-dev @datadog/browser-rum @datadog/browser-logs
```

### Criar arquivo de configuração

**cypress/support/datadog.js**

```js
import { datadogRum } from '@datadog/browser-rum'
import { datadogLogs } from '@datadog/browser-logs'

datadogRum.init({
  applicationId: 'SEU_APPLICATION_ID',
  clientToken: 'SEU_CLIENT_TOKEN',
  site: 'datadoghq.com',
  service: 'cypress-tests',
  env: 'staging',
  sessionSampleRate: 100,
  sessionReplaySampleRate: 100,
  trackUserInteractions: true,
  trackResources: true,
  trackLongTasks: true,
  defaultPrivacyLevel: 'mask-user-input',
})

datadogLogs.init({
  clientToken: 'SEU_CLIENT_TOKEN',
  site: 'datadoghq.com',
  service: 'cypress-tests',
  forwardErrorsToLogs: true,
  sessionSampleRate: 100,
})

datadogRum.startSessionReplayRecording()
```

### Importar no Cypress

**cypress/support/e2e.js**

```js
import './datadog.js'
```

---

## PASSO 3: Criar primeiro teste com monitoramento

### Teste básico: Login

```js
describe('Login Feature', () => {
  it('Should login successfully', () => {
    cy.visit('https://seu-app.com/login')

    cy.get('[data-testid="email"]').type('teste@email.com')
    cy.get('[data-testid="password"]').type('123456')

    cy.get('[data-testid="submit"]').click()

    cy.url().should('include', '/dashboard')
    cy.contains('Bem-vindo').should('be.visible')
  })
})
```

### Rodar teste

```bash
npx cypress run cypress/e2e/login.cy.js
```

### O que o Datadog coleta

* Tempo de carregamento de páginas
* Requisições feitas
* Erros de console
* Performance do navegador
* Latência da API

---

## PASSO 4: Adicionar logs customizados

```js
import { datadogLogs } from '@datadog/browser-logs'

describe('Login Feature com Monitoramento', () => {
  it('Should login and verify dashboard', () => {
    datadogLogs.logger.info('Test Started: Login Flow')

    cy.visit('https://seu-app.com/login')

    cy.get('[data-testid="email"]').type('teste@email.com')
    cy.get('[data-testid="password"]').type('123456')

    cy.intercept('POST', '/api/auth/login').as('loginRequest')
    cy.get('[data-testid="submit"]').click()

    cy.wait('@loginRequest').then((interception) => {
      datadogLogs.logger.info('Login API responded', {
        statusCode: interception.response.statusCode,
        duration: interception.response.responseTime,
      })
    })
  })
})
```

---

## PASSO 5: Monitorar requisições com `cy.intercept()`

* Captura status code
* Mede latência
* Detecta falhas reais de API
* Registra erros no Datadog

---

## PASSO 6: Medir performance

```js
const navTiming = window.performance.getEntriesByType('navigation')[0]
const pageLoadTime = navTiming.loadEventEnd - navTiming.fetchStart

expect(pageLoadTime).to.be.lessThan(3000)
```

---

## PASSO 7: Criar dashboards no Datadog

### Cypress Tests Overview

Gráficos sugeridos:

* Tests executados por hora
* Taxa de erro
* Latência média
* Requisições por status code

---

## PASSO 8: Criar alertas automáticos

### Exemplos de alertas

* ❌ Teste falhando
* 🐢 API lenta
* 🔥 Taxa de erro alta

---

## PASSO 9: Analisar logs e encontrar problemas

### Cenário 1: Teste passa, mas sistema lento

Conclusão: **Banco de dados lento**

### Cenário 2: Teste falha

Conclusão: **Infra/cache indisponível**

### Cenário 3: Vazamento de memória

Conclusão: **Memory leak**

---

## PASSO 10: Integração completa — exemplo prático

* Jornada completa do e-commerce
* Métricas por etapa
* Logs estruturados
* Performance real medida

Resultado final no Datadog:

```
TOTAL: 2.2 segundos
```

✅ Tudo perfeito!

---

## DICAS DE OURO

* Use tags customizadas (`testType`, `env`, `browser`)
* Sempre adicione contexto aos erros
* Meça tempo entre etapas
* Crie alertas automáticos

---

## Conclusão

Com **Cypress + Datadog** você:

* ✅ Vê exatamente o que acontece em cada teste
* ✅ Identifica gargalos reais
* ✅ Cria alertas automáticos
* ✅ Gera relatórios confiáveis
* ✅ Economiza horas de debug
* ✅ Aumenta sua credibilidade no time

### Próximos passos

1. Criar conta no Datadog
2. Instalar o agent
3. Configurar Cypress
4. Adicionar logs
5. Criar dashboards
6. Rodar testes e observar tudo em tempo real

🚀 **Você nunca mais vai rodar Cypress sem Datadog.**

---

Se quiser, posso:

* Ajustar para **README.md**
* Simplificar para **artigo de blog**
* Transformar em **ebook**
* Ou adaptar para **LinkedIn / Medium**

Só dizer 😄
