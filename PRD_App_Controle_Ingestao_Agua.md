# PRD — App de Controle de Ingestão de Água

**Autor:** Dário Rego — FICR
**Versão:** 1.0
**Data:** 29/04/2026
**Status:** Pronto para desenvolvimento

---

## 1. Visão Geral

Aplicativo móvel/web que ajuda usuários a manter uma rotina saudável de hidratação por meio do registro do consumo diário de água, definição de metas personalizadas, lembretes inteligentes e acompanhamento de progresso. O sistema gamifica a constância através de sequências de dias consecutivos atingindo a meta, incentivando a manutenção do hábito a longo prazo.

## 2. Objetivos

**Objetivos do produto:**
- Permitir o registro rápido e contínuo da ingestão diária de água.
- Calcular e sugerir metas personalizadas com base no peso do usuário.
- Manter o usuário engajado por meio de lembretes e indicadores de progresso.
- Gerar histórico e estatísticas de consumo para análise comportamental.

**Métricas de sucesso (KPIs):**
- Percentual de usuários atingindo a meta diária (≥ 60%).
- Retenção D7 e D30.
- Média de dias consecutivos de consumo (sequência).
- Número de registros de ingestão por usuário/dia (meta: ≥ 6).

## 3. Público-Alvo

- Adultos preocupados com saúde, bem-estar e hábitos diários.
- Praticantes de atividades físicas que precisam controlar hidratação.
- Pessoas com rotinas corridas que esquecem de beber água.

## 4. Stack Tecnológica Sugerida

- **Backend:** Node.js + Express (ou NestJS) / Python + FastAPI.
- **Banco de Dados:** PostgreSQL (a estrutura já usa `UUID` e `gen_random_uuid()` do PostgreSQL).
- **Frontend:** React (web) + React Native (mobile) ou Flutter para abordagem multiplataforma única.
- **Autenticação:** JWT + bcrypt para hash de senhas.
- **Notificações:** Push notifications (FCM / OneSignal) para os lembretes.

---

## 5. Modelo de Dados

O banco de dados é composto por **5 tabelas** e **1 view**, todas com chaves primárias do tipo `UUID` e relacionamentos `ON DELETE CASCADE` para garantir integridade referencial.

### 5.1. Tabela `usuarios`
Armazena os dados de cadastro e a meta padrão do usuário.

| Campo            | Tipo            | Descrição                                  |
|------------------|-----------------|--------------------------------------------|
| `id`             | UUID (PK)       | Identificador único.                       |
| `nome`           | VARCHAR(150)    | Nome completo. **Obrigatório.**            |
| `email`          | VARCHAR(150)    | E-mail único do usuário.                   |
| `peso`           | DECIMAL(5,2)    | Peso em kg (usado para cálculo da meta).   |
| `meta_diaria_ml` | INTEGER         | Meta padrão de consumo diário em ml.       |
| `criado_em`      | TIMESTAMP       | Data e hora do cadastro (default: NOW).    |

### 5.2. Tabela `ingestao_agua`
Registra cada vez que o usuário consome água.

| Campo            | Tipo       | Descrição                                       |
|------------------|------------|-------------------------------------------------|
| `id`             | UUID (PK)  | Identificador único do registro.                |
| `usuario_id`     | UUID (FK)  | Referência ao usuário (`ON DELETE CASCADE`).    |
| `quantidade_ml`  | INTEGER    | Quantidade ingerida em ml. **Obrigatório.**     |
| `data_hora`      | TIMESTAMP  | Momento do consumo (default: NOW).              |

### 5.3. Tabela `metas_diarias`
Permite registrar metas específicas por dia (sobrescrevem a meta padrão de `usuarios`).

| Campo         | Tipo       | Descrição                                          |
|---------------|------------|----------------------------------------------------|
| `id`          | UUID (PK)  | Identificador único.                               |
| `usuario_id`  | UUID (FK)  | Referência ao usuário (`ON DELETE CASCADE`).       |
| `meta_ml`     | INTEGER    | Meta de consumo do dia em ml. **Obrigatório.**     |
| `data`        | DATE       | Dia da meta. **Obrigatório.**                      |

> **Restrição:** `UNIQUE(usuario_id, data)` — um usuário só pode ter uma meta por dia.

### 5.4. Tabela `lembretes`
Horários configurados pelo usuário para receber notificações.

| Campo         | Tipo       | Descrição                                          |
|---------------|------------|----------------------------------------------------|
| `id`          | UUID (PK)  | Identificador único.                               |
| `usuario_id`  | UUID (FK)  | Referência ao usuário (`ON DELETE CASCADE`).       |
| `horario`     | TIME       | Horário do lembrete. **Obrigatório.**              |
| `ativo`       | BOOLEAN    | Se o lembrete está ativo (default: TRUE).          |

### 5.5. Tabela `sequencia_consumo`
Controla a gamificação por dias consecutivos atingindo a meta.

| Campo                | Tipo       | Descrição                                       |
|----------------------|------------|-------------------------------------------------|
| `id`                 | UUID (PK)  | Identificador único.                            |
| `usuario_id`         | UUID (FK)  | Referência ao usuário (`ON DELETE CASCADE`).    |
| `dias_consecutivos`  | INTEGER    | Contagem da sequência atual (default: 0).       |
| `ultima_data`        | DATE       | Último dia em que a meta foi atingida.          |

### 5.6. View `resumo_diario_agua`
Agrega o consumo total por usuário por dia. Usada para gráficos e cálculo de metas atingidas.

```sql
CREATE VIEW resumo_diario_agua AS
SELECT usuario_id, DATE(data_hora) AS data, SUM(quantidade_ml) AS total_ml
FROM ingestao_agua
GROUP BY usuario_id, DATE(data_hora);
```

---

## 6. Funcionalidades (Requisitos Funcionais)

### RF01 — Cadastro e Autenticação
- Cadastro com nome, e-mail (único) e senha.
- Login com e-mail e senha.
- Recuperação de senha por e-mail.
- Edição de perfil: nome, peso, foto.

### RF02 — Definição da Meta Diária
- Ao cadastrar o peso, o sistema sugere automaticamente uma meta (35 ml por kg de peso corporal).
- O usuário pode editar a meta padrão (`usuarios.meta_diaria_ml`).
- O usuário pode definir uma meta específica para um dia (`metas_diarias`), por exemplo em dias de treino intenso.

### RF03 — Registro de Ingestão
- Botões de adição rápida com volumes pré-definidos (100 ml, 200 ml, 250 ml, 500 ml).
- Opção de inserir volume customizado.
- Edição e exclusão de registros do dia atual.
- Histórico de registros do dia visível na tela inicial.

### RF04 — Acompanhamento e Dashboard
- **Tela inicial:** barra circular de progresso mostrando percentual da meta atingida hoje.
- Total consumido vs. meta do dia.
- Gráfico de barras dos últimos 7 dias.
- Gráfico de linha mensal (consumo médio).
- Indicador de quanto falta para atingir a meta.

### RF05 — Lembretes
- Cadastro de múltiplos horários de lembrete (`lembretes`).
- Ativar/desativar lembretes individualmente.
- Notificação push no horário configurado, se o usuário ainda não atingiu a meta.

### RF06 — Sequência de Consumo (Gamificação)
- A cada dia que o usuário atinge a meta, `sequencia_consumo.dias_consecutivos` é incrementado.
- Se o usuário falhar em um dia, a sequência é zerada.
- Exibição da sequência atual em destaque na tela inicial (ex.: "🔥 7 dias seguidos!").
- Marcos comemorativos: 7, 15, 30, 60, 100 dias.

### RF07 — Histórico
- Calendário com indicação visual dos dias em que a meta foi atingida.
- Detalhe ao clicar em um dia: total consumido, meta do dia, todos os registros.

---

## 7. Requisitos Não-Funcionais

| Categoria        | Requisito                                                                 |
|------------------|---------------------------------------------------------------------------|
| **Performance**  | Tempo de resposta da API < 300 ms para operações comuns.                  |
| **Segurança**    | Senhas com hash bcrypt; comunicação via HTTPS; JWT com expiração.         |
| **Disponibilidade** | 99,5% de uptime.                                                       |
| **Usabilidade**  | Registrar uma ingestão deve levar no máximo 2 toques.                     |
| **Acessibilidade** | Conformidade com WCAG 2.1 nível AA (contraste, leitor de tela, fontes). |
| **Privacidade**  | Conformidade com a LGPD; opção de exclusão de conta com remoção total dos dados (`ON DELETE CASCADE` já garante isso). |
| **Responsividade** | Suporte mobile-first; layouts adaptados para tablet e desktop.          |

---

## 8. Telas Principais

1. **Splash / Login / Cadastro**
2. **Onboarding** — coleta peso e calcula meta sugerida.
3. **Home (Dashboard)** — progresso do dia, botões de adição rápida, sequência atual.
4. **Histórico** — calendário e gráficos.
5. **Lembretes** — listagem e cadastro de horários.
6. **Perfil** — dados pessoais, meta padrão, exclusão de conta.

---

## 9. Endpoints da API (Sugestão REST)

| Método | Endpoint                              | Descrição                                  |
|--------|---------------------------------------|--------------------------------------------|
| POST   | `/auth/register`                      | Cadastro de usuário.                       |
| POST   | `/auth/login`                         | Login.                                     |
| GET    | `/usuarios/me`                        | Dados do usuário autenticado.              |
| PUT    | `/usuarios/me`                        | Atualiza perfil.                           |
| POST   | `/ingestoes`                          | Registra um consumo.                       |
| GET    | `/ingestoes?data=YYYY-MM-DD`          | Lista ingestões de um dia.                 |
| DELETE | `/ingestoes/:id`                      | Remove um registro.                        |
| GET    | `/resumo/diario?data=YYYY-MM-DD`      | Total consumido vs. meta do dia.           |
| GET    | `/resumo/semanal`                     | Consumo dos últimos 7 dias.                |
| POST   | `/metas`                              | Cria/atualiza meta de um dia específico.   |
| GET    | `/lembretes`                          | Lista lembretes.                           |
| POST   | `/lembretes`                          | Cria lembrete.                             |
| PATCH  | `/lembretes/:id`                      | Ativa/desativa lembrete.                   |
| GET    | `/sequencia`                          | Retorna sequência atual do usuário.        |

---

## 10. Regras de Negócio

- **RN01:** A meta sugerida é `peso × 35 ml`. Pode ser sobrescrita pelo usuário.
- **RN02:** Para o dia atual, se existir registro em `metas_diarias`, ele tem precedência sobre `usuarios.meta_diaria_ml`.
- **RN03:** A sequência é atualizada à meia-noite via job/cron: se o usuário atingiu a meta no dia anterior, incrementa; senão, zera.
- **RN04:** Lembretes só disparam se o usuário ainda não atingiu a meta do dia.
- **RN05:** Ao excluir um usuário, todos os dados relacionados são removidos automaticamente via `ON DELETE CASCADE`.
- **RN06:** Volume mínimo por registro: 10 ml. Volume máximo: 2000 ml por registro.

---

## 11. Roadmap de Entregas

**MVP (Sprint 1–2):**
- RF01 (cadastro/login), RF02 (meta), RF03 (registro), RF04 (dashboard básico).

**V1.1 (Sprint 3):**
- RF05 (lembretes), RF06 (sequência), RF07 (histórico).

**V2.0 (Futuro):**
- Integração com smartwatches (Apple Health, Google Fit).
- Compartilhamento social da sequência.
- Suporte a outras bebidas (chá, café) com fator de hidratação.
- IA para ajuste dinâmico da meta com base em clima e atividade física.

---

## 12. Anexo — DDL Completo do Banco

```sql
-- Tabela: usuarios
CREATE TABLE usuarios (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    nome VARCHAR(150) NOT NULL,
    email VARCHAR(150) UNIQUE,
    peso DECIMAL(5,2),
    meta_diaria_ml INTEGER,
    criado_em TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Tabela: ingestao_agua
CREATE TABLE ingestao_agua (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    usuario_id UUID NOT NULL,
    quantidade_ml INTEGER NOT NULL,
    data_hora TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT fk_usuario FOREIGN KEY (usuario_id)
        REFERENCES usuarios(id) ON DELETE CASCADE
);

-- Tabela: metas_diarias
CREATE TABLE metas_diarias (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    usuario_id UUID NOT NULL,
    meta_ml INTEGER NOT NULL,
    data DATE NOT NULL,
    CONSTRAINT fk_usuario_meta FOREIGN KEY (usuario_id)
        REFERENCES usuarios(id) ON DELETE CASCADE,
    UNIQUE(usuario_id, data)
);

-- Tabela: lembretes
CREATE TABLE lembretes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    usuario_id UUID NOT NULL,
    horario TIME NOT NULL,
    ativo BOOLEAN DEFAULT TRUE,
    FOREIGN KEY (usuario_id)
        REFERENCES usuarios(id) ON DELETE CASCADE
);

-- Tabela: sequencia_consumo
CREATE TABLE sequencia_consumo (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    usuario_id UUID NOT NULL,
    dias_consecutivos INTEGER DEFAULT 0,
    ultima_data DATE,
    FOREIGN KEY (usuario_id)
        REFERENCES usuarios(id) ON DELETE CASCADE
);

-- View: resumo_diario_agua
CREATE VIEW resumo_diario_agua AS
SELECT usuario_id, DATE(data_hora) AS data, SUM(quantidade_ml) AS total_ml
FROM ingestao_agua
GROUP BY usuario_id, DATE(data_hora);
```
