# Priorização da Fiscalização do Desmatamento na Amazônia Legal (Power BI & Análise de Dados)

**Equipe:** Diego Feitosa & Diego Braga  
**Cliente:** Secretaria Estadual de Meio Ambiente  
**Disciplina:** Power BI e Análise de Dados  

---

## 1. Descrição e Objetivo do Projeto

A fiscalização ambiental na Amazônia Legal enfrenta o desafio de atuar em escala continental com equipes e orçamento limitados. Este projeto desenvolve uma solução completa de **Inteligência Analítica e Modelagem Multicritério** para responder à pergunta central:

> *"Quais municípios da Amazônia Legal concentraram mais alertas de desmatamento nos últimos 12 meses e devem ser prioritários para a fiscalização?"*

### Perguntas de Negócio (P1 a P5)
- **P1 (Concentração):** Quais municípios tiveram mais alertas e maior área desmatada (km²) nos últimos 12 meses?
- **P2 (Tendência):** Quais municípios estão com tendência de crescimento (variação anual e inclinação da regressão linear mensal)?
- **P3 (Áreas Protegidas):** Qual o percentual de alertas localizados a até 10 km de Unidades de Conservação (UCs) ou Terras Indígenas (TIs)?
- **P4 (Priorização):** Qual a melhor combinação de critérios para hierarquizar os municípios (Índice de Priorização de 0 a 1)?
- **P5 (Sazonalidade):** Quais meses apresentam os maiores picos de desmatamento e qual o gatilho de mobilização preventiva?

---

## 2. Fontes Oficiais de Dados

| Conjunto de Dados | Órgão / Sistema | Acesso / Repositório Oficial |
| :--- | :--- | :--- |
| **Avisos Diários de Desmatamento (DETER)** | INPE / TerraBrasilis | https://terrabrasilis.dpi.inpe.br/downloads/ |
| **Malha Municipal da Amazônia Legal (772 municípios)** | IBGE / Diretoria de Geociências | https://geoftp.ibge.gov.br/organizacao_do_territorio/malhas_territoriais/malhas_municipais/ |
| **Unidades de Conservação (UCs)** | MMA / ICMBio (CNUC) | https://dados.gov.br/dados/conjuntos-dados/unidadesdeconservacao |
| **Terras Indígenas (TIs)** | FUNAI (DGeo) | https://geoserver.funai.gov.br/ |

---

## 3. Limpeza e Mineração de Dados (Pipeline ETL)

1. **Filtro Temporal (Janela Móvel de 24 Meses):** Retenção dos últimos 12 meses (análise atual) e 12 meses anteriores (base comparativa para variação anual).
2. **Filtro de Classes Alvo:** Seleção de `DESMATAMENTO_CR`, `DESMATAMENTO_VEG` e `MINERACAO` (remoção de `NUVEM` e `RESIDUO_MASCARA`).
3. **Remoção de Ruído Espacial e Duplicatas:** Descarte de *sliver polygons* menores que `0,03 km²` (3 hectares) e deduplicação por `GID`.
4. **Padronização de Chaves:** Uso do Código IBGE de 7 dígitos (`CD_MUN`) como chave primária e normalização de textos (`Trim`, `Upper`, UTF-8).
5. **Mineração Espacial, Temporal e Multicritério:**
   - Cálculo de distância mínima para UCs/TIs e flag binária `FLAG_BUFFER_10KM` (`<= 10 km`).
   - Regressão Linear por Mínimos Quadrados Ordinários (OLS) para estimar a inclinação mensal (Beta).
   - Normalização estatística Min-Max `[0, 1]` para composição do Índice de Priorização.

---

## 4. Regras de Negócio (RN-P1 a RN-P5)

- **RN-P1 (Concentração — Peso 40%):** Total de alertas (12m), Área Desmatada (km²) e Participação no Total (%).
- **RN-P2 (Tendência — Peso 30%):** Variação Anual (%) e Inclinação Mensal da Regressão Linear ($eta$).
- **RN-P3 (Áreas Protegidas — Peso 30%):** Percentual de alertas no buffer de até 10 km de UCs ou TIs.
- **RN-P4 (Índice de Priorização Multicritério):**
  - Fórmula: `IP = (0,40 * P1_norm) + (0,30 * P2_norm) + (0,30 * P3_norm)`
  - **Faixas Operacionais:**
    - `IP >= 0,75`: Prioridade Crítica (Força-Tarefa Permanente)
    - `0,50 <= IP < 0,75`: Prioridade Alta (Operações Táticas)
    - `IP < 0,50`: Monitoramento (Remoto e Ações Locais)
- **RN-P5 (Gatilho Sazonal):** Quando a Variação Mensal (MoM %) ultrapassa `+35%` na transição chuva-seca (Maio, com `+50,0%`), deflagra-se a mobilização preventiva antes do pico absoluto de Agosto (`1.620 alertas`).

---
