# Classificação de qualidade de vinhos com Machine Learning

Projeto acadêmico do **SENAI CIMATEC – Ford Enter Inteligência Artificial**: um fluxo completo de ciência de dados que vai da configuração do ambiente em nuvem até a implantação de um serviço de inferência com monitoramento e governança.

[![Abrir no Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/SEU_USUARIO/SEU_REPOSITORIO/blob/main/notebooks/wine_quality_colab.ipynb)

> **Autor:** Gideão Victor Bitencourt Silva

>**Professora:** Auriane do Carmo Barbosa

> **Instituição:** SENAI CIMATEC – Ford Enter Inteligência Artificial (Salvador, 2026)

---

## Sumário

1. [Sobre o projeto](#sobre-o-projeto)
2. [Base de dados](#base-de-dados)
3. [Arquitetura](#arquitetura)
4. [Estrutura do repositório](#estrutura-do-repositório)
5. [Como executar](#como-executar)
6. [Etapas do notebook](#etapas-do-notebook)
7. [Resultados](#resultados)
8. [Serviço de inferência (API)](#serviço-de-inferência-api)
9. [Monitoramento](#monitoramento)
10. [Governança e reprodutibilidade](#governança-e-reprodutibilidade)
11. [Limitações](#limitações)
12. [Melhorias futuras](#melhorias-futuras)
13. [Referências](#referências)

---

## Sobre o projeto

O cenário simulado é o de uma vinícola que quer apoiar o laboratório na triagem de amostras. A partir de medições físico-químicas, o modelo estima a probabilidade de um vinho ser de **alta qualidade** (nota sensorial ≥ 7), para que os especialistas priorizem as amostras que merecem a prova sensorial.

- **Tipo de problema:** classificação binária supervisionada.
- **Ambiente de computação:** Google Colab (runtime CPU).
- **Armazenamento:** Google Drive.
- **Modelo final:** floresta aleatória (Random Forest), escolhida pela validação cruzada.
- **Implantação:** API FastAPI executada no próprio Colab.
- **Monitoramento:** latência, taxa de erro, deriva dos dados (KS e PSI) e distribuição das previsões.

> Este é um experimento acadêmico. O modelo apoia a decisão de um especialista e **não deve ser usado sozinho** em decisões comerciais.

## Base de dados

**Wine Quality Dataset** (UCI Machine Learning Repository), com vinhos verdes portugueses tintos e brancos.

| Item | Valor |
|---|---|
| Registros originais | 6.497 (1.599 tintos e 4.898 brancos) |
| Duplicados removidos | 1.177 |
| Registros após limpeza | 5.320 |
| Atributos | 11 físico-químicos + tipo do vinho (`wine_type`) |
| Alvo | `high_quality` = 1 se `quality` ≥ 7, senão 0 |
| Proporção de alta qualidade | 18,97% (classe minoritária) |
| Treino / teste | 4.256 / 1.064 (divisão estratificada 80/20, semente 42) |

**Atributos:** acidez fixa, acidez volátil, ácido cítrico, açúcar residual, cloretos, dióxido de enxofre livre, dióxido de enxofre total, densidade, pH, sulfatos, álcool e tipo (0 = tinto, 1 = branco).

Fonte: <https://archive.ics.uci.edu/dataset/186/wine+quality>

## Arquitetura da solução


| Componente | Função |
|---|---|
| Google Colab | Computação: EDA, treino, validação e execução da API |
| Google Drive | Armazenamento de dados brutos e tratados, modelos, logs, relatórios e evidências |
| scikit-learn | Pipelines, validação cruzada, busca em grade e métricas |
| FastAPI + Uvicorn | Serviço de inferência com validação de entradas |
| Logs JSONL + KS/PSI | Monitoramento operacional, de dados e do modelo |
| Model card + hashes SHA-256 | Governança, rastreabilidade e auditoria |

## Estrutura do repositório

Sugestão de organização para o GitHub:

```
.
├── README.md
├── notebooks/
│   └── wine_quality_colab.ipynb      # pipeline completo (Passos 1, 2 e 4 a 9)
├── docs/
│   ├── Relatorio_Wine_Quality_ABNT.docx
│   └── arquitetura_solucao.png
├── reports/                          # figuras e tabelas geradas pelo notebook
│   ├── fig_target.png
│   ├── fig_confusion.png
│   ├── fig_roc.png
│   ├── fig_monitoring.png
│   ├── cv_results.csv
│   └── test_results.csv
└── evidence/                         # evidências da execução
    ├── environment.json
    ├── data_quality.json
    ├── inference_tests.json
    ├── monitoring_report.json
    └── model_card_v1.0.0.json
```

Durante a execução, o notebook cria no **Google Drive** a pasta `wine_quality_project/` com `data/raw`, `data/processed`, `models`, `logs`, `reports` e `evidence`. Os arquivos acima são cópias dos resultados dessa execução.

## Como executar

1. Abra o notebook `notebooks/wine_quality_colab.ipynb` no Google Colab (botão no topo deste README, depois de trocar `SEU_USUARIO` e `SEU_REPOSITORIO` pelo seu usuário e nome do repositório).
2. Execute **Ambiente de execução → Executar tudo**.
3. Autorize o acesso ao Google Drive quando solicitado.
4. Acompanhe as saídas de cada etapa. Os artefatos são salvos em `MyDrive/wine_quality_project/`.

**Requisitos:** conta Google, acesso à internet no Colab (o notebook baixa a base da UCI) e Python 3 (o próprio Colab fornece). O notebook instala `fastapi`, `uvicorn`, `scikit-learn`, `pandas`, `matplotlib`, `scipy`, `joblib` e `requests`. Versões usadas na execução documentada: Python 3.13.18, scikit-learn 1.6.1, pandas 2.2.3, NumPy 2.4.6, SciPy 1.16.3 e Matplotlib 3.10.0 (ver `evidence/environment.json`).

## Etapas do notebook

| Passo | O que acontece |
|---|---|
| 1 | Configuração do ambiente e registro das versões e do hardware |
| 2 | Montagem do Google Drive e criação da estrutura de pastas |
| 4 | Seleção e download da base pública, com cálculo do SHA-256 |
| 5 | Verificações de qualidade, remoção de duplicados, análise exploratória e divisão treino/teste |
| 6 | Treino e avaliação de três modelos com validação cruzada e busca de hiperparâmetros |
| 7 | Implantação da API FastAPI e testes de funcionamento |
| 8 | Monitoramento: latência, erros, deriva (KS e PSI) e alertas |
| 9 | Manifesto de evidências, cartão do modelo e encerramento do serviço |

**Modelagem:** regressão logística, floresta aleatória e gradient boosting, todos em *pipelines* com padronização (`StandardScaler`) ajustada só no treino de cada partição, para evitar vazamento de informação. Os hiperparâmetros foram escolhidos por `GridSearchCV` com `StratifiedKFold` de 5 partições, usando o **F1 da classe positiva** como métrica. O critério de seleção do modelo foi definido antes de usar o conjunto de teste.

## Resultados

**Validação cruzada (F1 da classe positiva):**

| Modelo | Hiperparâmetros escolhidos | F1 médio | Desvio-padrão |
|---|---|---|---|
| Regressão logística | C = 0,01 | 0,5391 | 0,0199 |
| **Floresta aleatória** | n_estimators = 400; max_depth = 10; min_samples_leaf = 1 | **0,5579** | 0,0295 |
| Gradient boosting | n_estimators = 200; learning_rate = 0,1; max_depth = 3 | 0,4631 | 0,0203 |

**Conjunto de teste (1.064 registros):**

| Modelo | Acurácia | Precisão | Revocação | F1 | ROC AUC | PR AUC |
|---|---|---|---|---|---|---|
| Regressão logística | 0,7350 | 0,4005 | 0,7970 | 0,5331 | 0,8167 | 0,4736 |
| **Floresta aleatória** | 0,8224 | 0,5259 | 0,6535 | **0,5828** | **0,8636** | **0,5864** |
| Gradient boosting | 0,8440 | 0,6500 | 0,3861 | 0,4845 | 0,8497 | 0,5585 |

**Como interpretar:**

- A floresta aleatória foi selecionada pela validação cruzada e também teve o melhor F1 e as maiores áreas ROC e PR no teste.
- A acurácia **não** é a métrica adequada: como só ~19% dos vinhos são de alta qualidade, um classificador que sempre respondesse "não" teria cerca de 81% de acurácia, e a floresta aleatória (82,2%) supera isso por pouco.
- Os modelos têm perfis de erro diferentes: a regressão logística encontra mais vinhos de alta qualidade (maior revocação) com muitos falsos positivos; o gradient boosting é mais preciso, mas deixa passar mais de 60% dos vinhos de alta qualidade.
- Desempenho moderado (F1 de 0,58): o modelo captura parte do sinal físico-químico, mas ainda erra com frequência.

Gráficos em [`reports/`](reports/): distribuição do alvo, matrizes de confusão, curvas ROC e painel de monitoramento.

## Serviço de inferência (API)

A API é carregada a partir do modelo salvo no Drive e roda no próprio Colab em `http://127.0.0.1:8000` (acessível apenas de dentro da sessão do Colab).

| Método | Rota | Descrição |
|---|---|---|
| GET | `/health` | Estado do serviço, nome e versão do modelo |
| POST | `/predict` | Predição para uma amostra |
| POST | `/predict_batch` | Predição para até 500 amostras (`{"items": [...]}`) |

**Exemplo de requisição** (valores ilustrativos):

```bash
curl -X POST http://127.0.0.1:8000/predict \
  -H "Content-Type: application/json" \
  -d '{
    "fixed_acidity": 7.0,
    "volatile_acidity": 0.3,
    "citric_acid": 0.3,
    "residual_sugar": 2.0,
    "chlorides": 0.05,
    "free_sulfur_dioxide": 30,
    "total_sulfur_dioxide": 110,
    "density": 0.994,
    "pH": 3.2,
    "sulphates": 0.55,
    "alcohol": 11.5,
    "wine_type": 1
  }'
```

**Formato da resposta:**

```json
{
  "alta_qualidade": true,
  "probabilidade": 0.0,
  "versao_modelo": "1.0.0"
}
```

(`probabilidade` é a probabilidade estimada da classe de alta qualidade; `alta_qualidade` é verdadeiro quando ela é ≥ 0,5.)

**Validação de entrada:** cada atributo tem limites plausíveis definidos com Pydantic (por exemplo, pH entre 2 e 5 e teor alcoólico entre 5 e 20). Valores fora do intervalo ou campos ausentes retornam **HTTP 422**.

**Testes realizados** (`evidence/inference_tests.json`): predição unitária, lote de 50 amostras, entrada inválida e campo ausente (422 esperado), consistência entre API e modelo local (diferença máxima de 4,9 × 10⁻⁵) e latência em 100 chamadas (p50 = 85,4 ms; p95 = 98,8 ms).

## Monitoramento

Cada requisição gera um registro em `logs/requests.jsonl`. O relatório de monitoramento cobre três dimensões:

| Dimensão | O que é acompanhado | Limite de alerta |
|---|---|---|
| Operacional | Latência (p50 e p95) e taxa de erro HTTP | p95 > 500 ms; erro > 5% |
| Dados | Deriva por atributo contra uma amostra de referência do treino (KS e PSI) | PSI > 0,25 e p-valor do KS < 0,01 |
| Modelo | Taxa de predições positivas | Fora da faixa de 2% a 60% |

**Resultado da simulação** (tráfego com deriva deliberada em álcool, acidez volátil e açúcar residual): 257 requisições, latência p50 = 81,8 ms e p95 = 93,8 ms, taxa de erro de 0,78% (as duas entradas inválidas enviadas de propósito nos testes) e **alerta de deriva para o teor alcoólico** (PSI = 0,31). A acidez volátil ficou no limite (PSI = 0,2495) e o açúcar residual não alertou, o que mostra que o monitor detecta deslocamentos grandes, mas tem sensibilidade limitada para desvios moderados. Os limiares são pontos de partida e devem ser calibrados com tráfego real.

Detalhes em [`evidence/monitoring_report.json`](evidence/monitoring_report.json).

## Governança e reprodutibilidade

- **Cartão do modelo** (`model_card_v1.0.0.json`): versão, parâmetros, métricas, dados de origem, ambiente, uso pretendido e limitações.
- **Hashes SHA-256** dos dados brutos e de todos os artefatos, reunidos em um manifesto (`manifest.json`, gerado no Drive).
- **Semente fixa** (42) e versões das bibliotecas registradas em `environment.json`.
- **Camadas de dados separadas:** bruta, tratada, modelos e logs.
- **Uso pretendido explícito:** apoio à triagem, com decisão final de um especialista.

## Limitações

- A nota sensorial é subjetiva.
- A base é histórica e restrita a vinhos verdes de Portugal.
- A classe positiva é minoritária (~19%) e o F1 do modelo é moderado.
- Não há validação externa com dados novos.
- O Google Colab é adequado a prototipagem, mas não oferece alta disponibilidade, réplicas, autenticação gerenciada nem endpoint público estável. A API deixa de existir quando a sessão é encerrada; os artefatos permanecem no Drive.

## Melhorias futuras

- Implantar a API em serviço gerenciado (por exemplo, Cloud Run, Vertex AI, Amazon SageMaker ou Azure ML), com contêiner, escalonamento automático, autenticação e TLS.
- Armazenar dados e modelos em serviço de objetos com controle de acesso e criptografia.
- Automatizar o treino e registrar experimentos e modelos (por exemplo, com MLflow).
- Painéis e alertas automáticos de monitoramento, com coleta de rótulos reais para acompanhar o desempenho em produção.
- Estender a grade da regressão logística (o melhor C foi o menor valor testado), aplicar pesos de classe ao gradient boosting e escolher o limiar de decisão por custo de erro.
- Calibrar probabilidades, usar validação aninhada e várias sementes, e testar outros algoritmos.
- Calibrar o monitoramento de deriva com janelas de tempo e níveis de atenção e de alerta.
- Ampliar a base com amostras recentes e representativas da vinícola.

