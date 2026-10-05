# APIs de Energia Renovável e Aprendizado de Máquina

Checkpoint 02 — SERS | FIAP | Ciências da Computação (1CC)

## Integrantes

| Nome | RM |
|---|---|
| Eduardo Novaki Santos Coelho | 572649 |
| Gabriel dos Santos Siqueira | 572200 |
| Pedro Andreassa Zamai | 569318 |
| Pedro Yoshikado Garcia | 570449 |
| Rafael Ferreirinha Quaresma | 571949 |
| Thiago Maluf Hofmann | 569852 |

## Objetivo

Consultar duas APIs públicas de dados de energia, organizar os dados e resolver duas tarefas de aprendizado de máquina, treinando e comparando **três algoritmos em cada tarefa**:

1. **Classificação (ANEEL):** a partir da potência e da localização de um empreendimento, classificar sua fonte como **Solar, Eólica ou Hidráulica**.
2. **Regressão (Open-Meteo):** a partir de condições meteorológicas e da hora local em Petrolina (PE), estimar a **radiação solar horizontal** (W/m²).

## Fontes e período dos dados

| Tarefa | Fonte | Dados utilizados | Período |
|---|---|---|---|
| Classificação | [ANEEL — SIGA (API CKAN/DataStore)](https://dadosabertos.aneel.gov.br/dataset/siga-sistema-de-informacoes-de-geracao-da-aneel) | Potência outorgada (kW), latitude e longitude de empreendimentos `UFV` (solar), `EOL` (eólica) e `UHE`, `PCH`, `CGH` (agrupadas como hidráulica) | Cadastro vigente no momento da consulta, com limite de 1200 registros por tipo |
| Regressão | [Open-Meteo — API histórica](https://open-meteo.com/en/docs/historical-weather-api) | Temperatura, umidade, cobertura de nuvens, vento e radiação de onda curta em Petrolina (PE), latitude -9,39 e longitude -40,50, das 7h às 17h, fuso `America/Recife` | 1º de abril a 30 de junho de 2025 |

As duas consultas são públicas e **não exigem login, token ou chave de API**.

Observações sobre os dados:
- A quantidade de empreendimentos por tipo é limitada pela consulta e **não representa a matriz energética brasileira**.
- Os dados do Open-Meteo são estimativas de modelos/reanálise, não leituras de um sensor.
- Radiação em W/m² **não equivale** à energia em kWh nem à geração de um painel solar.

## Como executar o notebook

### No Google Colab (recomendado)

1. Abra o notebook `Aula_APIs_Energia_Renovavel_ML.ipynb` no Colab (**Arquivo → Abrir notebook → GitHub**, colando o link do repositório).
2. Clique em **Ambiente de execução → Executar tudo** (ou **Reiniciar sessão e executar tudo**).
3. As células de consulta às APIs geram os dois arquivos CSV na sessão. Em seguida, o notebook lê esses CSVs, treina os modelos e exibe tabelas, gráficos e conclusões.

### Localmente

1. Instale o Python 3.9 ou superior e as dependências:
   ```bash
   pip install pandas numpy matplotlib seaborn scikit-learn jupyter
   ```
2. Clone o repositório e abra o notebook:
   ```bash
   git clone <link-do-repositório>
   cd <pasta-do-repositório>
   jupyter notebook Aula_APIs_Energia_Renovavel_ML.ipynb
   ```
3. Execute todas as células em ordem. É necessário acesso à internet para as consultas às APIs.

> Os CSVs já estão no repositório. Se alguma API estiver indisponível, pule as células de consulta e execute a partir da célula de imports: o notebook lê os CSVs da mesma pasta.

## Metodologia

**Tarefa 1 — Classificação.** Entradas: `potencia_kw`, `latitude` e `longitude`. Alvo: `fonte`. Divisão estratificada 80% treino e 20% teste, com `random_state=42`. KNN e Regressão Logística usam `StandardScaler` dentro de um `Pipeline`, ajustado apenas com o treino. Métricas por classe com média **macro**.

**Tarefa 2 — Regressão.** Entradas: `temperatura_c`, `umidade_pct`, `nuvens_pct`, `vento_kmh` e `hora`. Alvo: `radiacao_w_m2`. Divisão **temporal**, sem embaralhar: as primeiras 80% das horas para treino e as 20% finais para teste.

## Resumo dos resultados

### Tarefa 1 — Classificação (ANEEL)

| Algoritmo | Accuracy | Precision (macro) | Recall (macro) | F1 (macro) |
|---|---|---|---|---|
| KNN (com escala) | 0,9652 | 0,9663 | 0,9636 | 0,9648 |
| Regressão Logística (com escala) | 0,8247 | 0,8282 | 0,8214 | 0,8197 |
| **Random Forest** | **0,9755** | **0,9769** | **0,9741** | **0,9753** |

- O **Random Forest** teve o melhor desempenho e é o modelo escolhido. Ele errou 19 dos 776 exemplos de teste.
- A classe Solar foi a que mais gerou erros no Random Forest (7 previstas como Hidráulica e 5 como Eólica). A Regressão Logística confunde principalmente Solar com Eólica.
- A Regressão Logística teve o menor desempenho, o que indica que a fronteira entre as fontes não é linear.
- **Limitação:** potência e localização não bastam para uma aplicação real. Além disso, alguns registros têm coordenadas (0, 0), que parecem ser valor de preenchimento do cadastro.

### Tarefa 2 — Regressão (Open-Meteo)

| Algoritmo | MAE (W/m²) | MSE ((W/m²)²) | R² |
|---|---|---|---|
| Regressão Linear | 145,20 | 30034,20 | 0,3598 |
| Decision Tree | 88,79 | 15291,16 | 0,6741 |
| **Random Forest** | **66,80** | **7307,42** | **0,8442** |

- O **Random Forest** teve o menor erro e o maior R². Seu MAE de ~67 W/m² corresponde a cerca de 14% da radiação média do período (~470 W/m²).
- A Regressão Linear teve desempenho fraco porque a radiação não varia linearmente com a hora: sobe até o meio-dia e cai depois.
- A variável `hora` foi a mais importante (~0,49), seguida de temperatura (~0,30) e umidade (~0,17). A cobertura de nuvens teve importância baixa (~0,02).
- **Limitação:** o modelo estima a radiação solar, e não a energia elétrica gerada. A geração de um sistema fotovoltaico também depende de potência instalada, eficiência dos painéis, área, orientação, inclinação e perdas.

As conclusões completas, as matrizes de confusão e o gráfico real × previsto estão no notebook.
