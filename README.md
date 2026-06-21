# Serviço de Pagamento

Serviço desenvolvido em Node.js para gerenciar pagamentos com armazenamento de histórico e categorização automática de valores.

---

## Funcionalidades

- **`pagar(codigoBarras, valor)`** — Registra um novo pagamento e retorna o objeto criado.
- **`consultarUltimoPagamento()`** — Retorna o último pagamento registrado ou `null` se não houver nenhum.
- **Categorização automática** — Valores acima de R$ 100,00 são marcados como `"cara"`, os demais como `"padrão"`.

### Estrutura do objeto de pagamento

```js
{
  codigoBarras: String,   // Código de barras informado
  empresa: String,        // Sempre "Nike"
  valor: Number,          // Valor em reais
  categoria: String       // "cara" ou "padrão"
}
```

---

## Como executar localmente

**Pré-requisito:** Node.js 18 ou superior instalado.

```bash
# Instalar dependências
npm install

# Rodar os testes no terminal
npm test

# Gerar relatório HTML em reports/test-report.html
npm run test:report
```

---

## Pipeline de Integração Contínua (CI)

Este projeto utiliza **GitHub Actions** para executar os testes automatizados e gerar relatórios a cada alteração no repositório.

### Arquivo de configuração

`.github/workflows/ci.yml`

### Gatilhos configurados

| Gatilho | Descrição |
|---|---|
| `push` | Executa em qualquer push, em qualquer branch |
| `workflow_dispatch` | Execução manual pelo painel do GitHub Actions |
| `schedule (cron)` | Execução automática toda segunda-feira às 08h UTC |

### Etapas da pipeline

```
1. Checkout do código       → Baixa o repositório no runner
2. Configurar Node.js 20    → Instala o Node com cache de dependências npm
3. Instalar dependências    → npm ci (instalação limpa e reproduzível)
4. Executar testes          → npm test (saída visível nos logs da pipeline)
5. Gerar relatório HTML     → npm run test:report (mochawesome)
6. Publicar artefato        → Upload do relatório para 30 dias de retenção
```

> As etapas 5 e 6 são executadas mesmo quando os testes falham (`if: always()`), garantindo que o relatório esteja sempre disponível para diagnóstico.

### Relatório de testes

O relatório é gerado pelo **mochawesome** em formato HTML e publicado como artefato da execução. Para acessá-lo:

1. Acesse a aba **Actions** no repositório do GitHub.
2. Clique na execução desejada.
3. Na seção **Artifacts**, baixe o arquivo `relatorio-de-testes`.
4. Abra `reports/test-report.html` no navegador.

---

## Conceitos utilizados

### Integração Contínua (CI)

Prática de integrar e validar o código de forma automática e frequente. A cada push, a pipeline executa os testes, garantindo que o código novo não quebre o que já funcionava.

### GitHub Actions

Plataforma de automação nativa do GitHub. Permite criar workflows declarativos em YAML que reagem a eventos do repositório (push, pull request, schedule, etc.) e executam jobs em runners gerenciados pela plataforma.

### Workflow

Arquivo YAML em `.github/workflows/` que define quando e como a automação é executada. Composto por:
- **`on`** — os gatilhos de execução
- **`jobs`** — os grupos de trabalho (cada um roda em um runner)
- **`steps`** — os passos sequenciais dentro de um job

### Gatilhos (triggers)

- **`push`** — reage a commits enviados ao repositório.
- **`workflow_dispatch`** — habilita execução manual com um clique na interface do GitHub.
- **`schedule`** — execução agendada por expressão cron (ex: `0 8 * * 1` = toda segunda às 8h UTC).

### Actions reutilizáveis

Blocos prontos publicados no GitHub Marketplace:
- **`actions/checkout@v4`** — faz o clone do repositório no runner.
- **`actions/setup-node@v4`** — instala e configura o Node.js com cache.
- **`actions/upload-artifact@v4`** — armazena arquivos gerados na pipeline como artefatos acessíveis por 30 dias.

### `npm ci`

Alternativa ao `npm install` otimizada para ambientes de CI: instala exatamente as versões travadas no `package-lock.json`, sem gerar alterações no lockfile, garantindo builds reproduzíveis.

### Artefatos

Arquivos gerados durante a execução da pipeline e armazenados no GitHub. Permitem inspecionar relatórios, logs ou binários produzidos sem precisar re-executar a pipeline.

### Mochawesome

Reporter do Mocha que gera relatórios HTML visuais com detalhes de cada teste (aprovados, reprovados, duração, mensagens de erro). Usado com a flag `--reporter mochawesome`.

---

## Testes automatizados

Framework: **Mocha** com módulo nativo `assert` do Node.js.

| Suite | Testes | Cobertura |
|---|---|---|
| `pagar()` | 5 | Criação do objeto, categorização, múltiplos pagamentos |
| `consultarUltimoPagamento()` | 5 | Último pagamento, lista vazia, isolamento do resultado |
| **Total** | **10** | |

---

## Estrutura do projeto

```
Servico-de-pagamento/
├── .github/
│   └── workflows/
│       └── ci.yml              # Pipeline do GitHub Actions
├── src/
│   └── servicoDePagamento.js   # Classe principal
├── test/
│   └── servicoDePagamento.test.js  # Testes com Mocha
├── package.json
└── README.md
```

---

## Tecnologias

- **Node.js 20**
- **Mocha 10** — framework de testes
- **Mochawesome 7** — geração de relatório HTML
- **GitHub Actions** — CI/CD
