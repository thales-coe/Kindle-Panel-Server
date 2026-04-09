# Kindle Panel Server

Painel web em Python para dispositivos e-ink (como Kindle), com foco em baixa complexidade operacional, atualização periódica e leitura rápida de informações essenciais.

O projeto expõe uma página HTTP local que consolida:

- hora e data do sistema;
- clima de São Paulo via API pública;
- notícias por RSS;
- eventos do Google Calendar em modo somente leitura;
- métricas básicas do host (CPU, RAM, disco, uptime, IP e hostname).

> **Importante:** este repositório **não deve versionar credenciais**. Arquivos como `credentials.json`, `token.json`, caches locais e quaisquer segredos devem permanecer fora do Git.

---

## 1. Visão geral

O servidor é implementado em um único arquivo Python e usa `HTTPServer` da biblioteca padrão. A renderização é feita em HTML gerado dinamicamente, com CSS otimizado para tela monocromática e atualização automática configurável.

Arquivo principal atual:

- `server_kindle_google_agenda.py`

Baseado no código enviado pelo usuário, o projeto roda por padrão em `0.0.0.0:8000`, atualiza a página a cada 30 segundos, consome feeds RSS, consulta a API `met.no` para clima e lê múltiplos calendários do Google via OAuth 2.0 em escopo somente leitura. fileciteturn0file0

---

## 2. Funcionalidades

### Interface
- Layout em HTML/CSS com tipografia monoespaçada.
- Compatibilidade com navegação simples em navegador embutido.
- Atualização automática por meta refresh.
- Renderização server-side sem JavaScript crítico.

### Clima
- Consulta à API `met.no` usando coordenadas fixas.
- Cache em memória.
- Persistência do último clima válido em arquivo local (`weather_last.json`).
- Fallback quando o serviço externo fica indisponível. fileciteturn0file0

### Notícias
- Leitura de múltiplos feeds RSS.
- Deduplicação simples por título.
- Rotação de manchetes conforme o ciclo de refresh. fileciteturn0file0

### Agenda
- Integração com Google Calendar em modo `readonly`.
- Leitura de múltiplos calendários configurados em lista.
- Exibição compacta dos eventos de hoje e amanhã. fileciteturn0file0

### Status do host
- Uso de CPU por leitura de `/proc/stat`.
- Uso de memória por leitura de `/proc/meminfo`.
- Uso de disco por `shutil.disk_usage`.
- Uptime, hostname e IP local. fileciteturn0file0

---

## 3. Stack técnica

- **Linguagem:** Python 3.11+
- **Servidor HTTP:** `http.server.HTTPServer`
- **Integrações externas:**
  - Google Calendar API
  - RSS/XML feeds
  - API de clima `met.no`
- **Bibliotecas de terceiros:**
  - `google-auth`
  - `google-auth-oauthlib`
  - `google-api-python-client`

---

## 4. Estrutura sugerida do repositório

```text
.
├── server_kindle_google_agenda.py
├── README.md
├── .gitignore
├── .env.example
├── requirements.txt
├── SECURITY.md
├── CONTRIBUTING.md
├── CHANGELOG.md
└── docs/
    ├── ARCHITECTURE.md
    └── DEPLOYMENT.md
```

---

## 5. Requisitos

- Python 3.11 ou superior
- Conta Google com acesso aos calendários desejados
- `credentials.json` obtido no Google Cloud Console
- Rede local acessível pelo dispositivo que exibirá o painel

---

## 6. Instalação local

### 6.1. Clonar o repositório

```bash
git clone <URL_DO_REPOSITORIO>
cd <NOME_DO_REPOSITORIO>
```

### 6.2. Criar ambiente virtual

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 6.3. Instalar dependências

```bash
pip install -r requirements.txt
```

### 6.4. Adicionar credenciais locais

Coloque o arquivo **real** `credentials.json` na raiz do projeto **sem commitar no Git**.

Na primeira execução, o OAuth abrirá o fluxo de autenticação e gerará `token.json`, que também **não deve ser versionado**.

### 6.5. Rodar o servidor

```bash
python3 server_kindle_google_agenda.py
```

Saída esperada:

```text
Servidor rodando em http://0.0.0.0:8000
```

---

## 7. Configuração sensível e segredos

O código enviado contém pontos que exigem cuidado antes de subir para GitHub:

### Não versionar
- `credentials.json`
- `token.json`
- `weather_last.json`
- arquivos `.env`
- qualquer dump, log ou backup com dados pessoais

### Revisar antes do primeiro push
- IDs de calendários pessoais e de terceiros
- e-mail usado em `User-Agent`
- nomes internos de calendários
- IPs, hostnames, caminhos locais e prints de tela

### Recomendação prática
Antes de publicar:
1. mova valores sensíveis para variáveis de ambiente ou arquivo `.env` local;
2. substitua exemplos reais por placeholders;
3. confira `git status` antes de cada commit;
4. use `.gitignore` desde o primeiro commit.

---

## 8. Exemplo de variáveis de ambiente

Mesmo que o código atual ainda use constantes internas, esta é a direção recomendada para endurecimento do projeto:

```env
HOST=0.0.0.0
PORT=8000
REFRESH_SECONDS=30
REQUEST_TIMEOUT=4
CITY_QUERY=Sao Paulo
PANEL_TITLE=Kindle Panel
GOOGLE_CREDENTIALS_FILE=credentials.json
GOOGLE_TOKEN_FILE=token.json
WEATHER_STATE_FILE=weather_last.json
MET_USER_AGENT=KindlePainel/1.0 (contact: seu-email-ou-alias)
```

---

## 9. Segurança mínima para GitHub

### Checklist antes de publicar
- [ ] `.gitignore` criado e validado
- [ ] nenhum segredo aparece em `git diff`
- [ ] nenhum token ou credencial foi commitado antes
- [ ] nomes pessoais foram revisados
- [ ] endpoints internos foram revisados
- [ ] arquivo principal abre sem depender de segredo hardcoded além dos arquivos locais ignorados

### Se um segredo já tiver sido commitado
1. revogue a credencial imediatamente no provedor;
2. gere nova credencial;
3. remova o arquivo do histórico, não apenas do último commit;
4. faça rotação de tokens associados.

---

## 10. Execução em servidor Linux

### Execução simples
```bash
source .venv/bin/activate
python3 server_kindle_google_agenda.py
```

### Execução em background com systemd
Ver `docs/DEPLOYMENT.md` para um exemplo completo de unit file.

---

## 11. Observações de arquitetura

- O projeto usa cache em memória para reduzir chamadas externas.
- A medição de CPU usa uma janela curta com `sleep(0.10)`, o que é suficiente para exibição simples, mas não para observabilidade rigorosa. fileciteturn0file0
- O layout é gerado integralmente a cada request.
- O uso de uma única thread com `HTTPServer` é adequado para cenário doméstico/local, mas não para tráfego alto.

---

## 12. Próximos passos recomendados

1. Extrair configurações para `.env`.
2. Separar coleta de dados, renderização HTML e servidor em módulos distintos.
3. Adicionar logs estruturados.
4. Criar testes unitários para parsers e helpers.
5. Criar rota `/health` para monitoramento.
6. Adicionar serviço `systemd` e reinício automático.
7. Sanitizar identificadores pessoais antes de tornar o repositório público.

---

## 13. Licença

Defina a licença do projeto antes de publicar. Se você ainda não decidiu, mantenha o repositório sem licença até escolher explicitamente uma.

---

## 14. Aviso

Este projeto foi documentado a partir do arquivo Python enviado pelo usuário. A documentação técnica acima reflete o comportamento observável do código-base fornecido. fileciteturn0file0
