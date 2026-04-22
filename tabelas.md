# 💧 APP DE CONTROLE DE INGESTÃO DE ÁGUA

Este documento descreve a estrutura do banco de dados para um aplicativo de controle de ingestão de água, incluindo cadastro de usuários, registros de consumo e metas diárias.

---

## 🧩 Estrutura do Banco de Dados

### 👤 Tabela: usuarios

```sql
CREATE TABLE usuarios (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    nome VARCHAR(150) NOT NULL,
    email VARCHAR(150) UNIQUE,
    peso DECIMAL(5,2),
    meta_diaria_ml INTEGER,
    criado_em TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

### 💧 Tabela: ingestao_agua

```sql
CREATE TABLE ingestao_agua (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    usuario_id UUID NOT NULL,
    quantidade_ml INTEGER NOT NULL,
    data_hora TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_usuario
        FOREIGN KEY (usuario_id)
        REFERENCES usuarios(id)
        ON DELETE CASCADE
);
```

---

### 🎯 Tabela: metas_diarias

```sql
CREATE TABLE metas_diarias (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    usuario_id UUID NOT NULL,
    meta_ml INTEGER NOT NULL,
    data DATE NOT NULL,

    CONSTRAINT fk_usuario_meta
        FOREIGN KEY (usuario_id)
        REFERENCES usuarios(id)
        ON DELETE CASCADE,

    UNIQUE(usuario_id, data)
);
```

---

## 📊 View: resumo_diario_agua

```sql
CREATE VIEW resumo_diario_agua AS
SELECT 
    usuario_id,
    DATE(data_hora) AS data,
    SUM(quantidade_ml) AS total_ml
FROM ingestao_agua
GROUP BY usuario_id, DATE(data_hora);
```

---

## 🔔 Tabela: lembretes

```sql
CREATE TABLE lembretes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    usuario_id UUID NOT NULL,
    horario TIME NOT NULL,
    ativo BOOLEAN DEFAULT TRUE,

    FOREIGN KEY (usuario_id)
        REFERENCES usuarios(id)
        ON DELETE CASCADE
);
```

---

## 🏆 Tabela: sequencia_consumo

```sql
CREATE TABLE sequencia_consumo (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    usuario_id UUID NOT NULL,
    dias_consecutivos INTEGER DEFAULT 0,
    ultima_data DATE,

    FOREIGN KEY (usuario_id)
        REFERENCES usuarios(id)
        ON DELETE CASCADE
);
```
