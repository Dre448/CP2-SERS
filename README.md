# Energia renovável e aprendizado de máquina

Avaliação sobre o uso de APIs públicas de energia renovável e de modelos de aprendizado de máquina. O trabalho tem duas tarefas:

1. **Classificação:** descobrir se um empreendimento da ANEEL é **Solar**, **Eólico** ou **Hidráulico**, usando a potência e a localização.
2. **Regressão:** estimar a **radiação solar** (em W/m²) em **Petrolina (PE)**, usando o clima e a hora do dia.

Em cada tarefa foram treinados **três algoritmos diferentes**, e os resultados foram comparados.

---

## Sumário

1. [Objetivo](#1-objetivo)
2. [Fontes dos dados](#2-fontes-dos-dados)
3. [Estrutura do repositório](#3-estrutura-do-repositório)
4. [Como executar](#4-como-executar)
5. [Bibliotecas usadas](#5-bibliotecas-usadas)
6. [Tarefa 1 — Classificação (ANEEL)](#6-tarefa-1--classificação-aneel)
7. [Tarefa 2 — Regressão (Open-Meteo)](#7-tarefa-2--regressão-open-meteo)
8. [Resultados](#8-resultados)
9. [Conclusões](#9-conclusões)
10. [Limitações](#10-limitações)
11. [Segurança e credenciais](#11-segurança-e-credenciais)

---

## 1. Objetivo

Praticar o caminho completo de um projeto de dados:

- consultar APIs públicas;
- organizar os dados em tabelas (CSV);
- explorar e visualizar os dados;
- separar treino e teste do jeito certo;
- treinar e comparar três algoritmos em cada tarefa;
- interpretar os resultados e entender os limites de cada modelo.

**Perguntas do trabalho**

| Tarefa | Pergunta | Tipo de problema |
|---|---|---|
| 1 | A partir da potência e da localização, é possível dizer se um empreendimento é solar, eólico ou hidráulico? | Classificação (o alvo é uma categoria) |
| 2 | Dadas as condições do tempo e a hora, qual é a radiação solar média naquela hora em Petrolina? | Regressão (o alvo é um número) |

---

## 2. Fontes dos dados

Nenhuma das duas APIs exige login, token ou chave. As consultas são públicas.

### ANEEL — SIGA (Sistema de Informações de Geração da ANEEL)

- Site do conjunto de dados: https://dadosabertos.aneel.gov.br/dataset/siga-sistema-de-informacoes-de-geracao-da-aneel
- API usada: CKAN / DataStore (`datastore_search`)
- Identificador do recurso: `11ec447d-698d-4ab8-977f-b424d5deee6a`
- Tipos de geração consultados (`SigTipoGeracao`):

| Sigla | Significado | Classe usada |
|---|---|---|
| `UFV` | Usina solar fotovoltaica | Solar |
| `EOL` | Usina eólica | Eólica |
| `UHE` | Usina hidrelétrica | Hidráulica |
| `PCH` | Pequena central hidrelétrica | Hidráulica |
| `CGH` | Central geradora hidrelétrica | Hidráulica |

- Limite de 1200 linhas por sigla em cada consulta. Por isso, **as quantidades não representam a participação real de cada fonte na matriz elétrica brasileira**.

### Open-Meteo — Histórico do tempo

- Documentação: https://open-meteo.com/en/docs/historical-weather-api
- API usada: `https://archive-api.open-meteo.com/v1/archive`
- Local: Petrolina (PE), latitude `-9.39` e longitude `-40.50`
- **Período:** de **01/04/2025** a **30/06/2025** (datas incluídas)
- Fuso horário: `America/Recife` (a hora dos dados é a hora local)
- Dados horários, mantidos apenas das **7h às 17h**
- São dados estimados por modelos (reanálise), e **não** leituras de um sensor do local.

---

## 3. Estrutura do repositório

```
.
├── README.md                                        Este arquivo
├── Aula_APIs_Energia_Renovavel_ML.ipynb             Notebook original: consulta às APIs e geração dos CSVs
├── Aula_APIs_Energia_Renovavel_ML_resolvido.ipynb   Notebook com a análise e os modelos
├── aneel_classificacao_orange.csv                   Dados da Tarefa 1
├── meteo_regressao_orange.csv                       Dados da Tarefa 2
└── figuras/                                         Gráficos salvos (opcional)
```

> Ajuste esta lista se os nomes ou as pastas do seu repositório forem diferentes.

---

## 4. Como executar

### Opção A — Google Colab (usada no trabalho)

1. Abra o notebook `Aula_APIs_Energia_Renovavel_ML.ipynb` no Colab.
2. Execute todas as células **na ordem**. Elas consultam as duas APIs e geram os dois arquivos CSV.
3. Confirme que os arquivos `aneel_classificacao_orange.csv` e `meteo_regressao_orange.csv` aparecem na pasta `/content/` (painel de arquivos à esquerda).
4. Abra o notebook `Aula_APIs_Energia_Renovavel_ML_resolvido.ipynb` no mesmo ambiente.
5. Execute todas as células **na ordem** (Ambiente de execução → Executar tudo).

### Opção B — Computador local

1. Tenha o Python 3.9 ou mais novo.
2. Instale as bibliotecas:

   ```bash
   pip install pandas numpy matplotlib seaborn scikit-learn notebook
   ```

3. Abra o notebook original e execute todas as células para gerar os CSVs.
4. No notebook resolvido, troque os caminhos de leitura, que começam com `/content/`, pelo caminho onde os CSVs foram salvos. Exemplo:

   ```python
   dados = pd.read_csv('aneel_classificacao_orange.csv')
   dados_meteo = pd.read_csv('meteo_regressao_orange.csv')
   ```

5. Execute todas as células **na ordem**.

### Se não houver internet

Os dois CSVs estão no repositório. Basta pular a etapa de consulta às APIs e rodar apenas o notebook resolvido.

---

## 5. Bibliotecas usadas

O notebook resolvido usa **somente** estas importações:

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import accuracy_score, precision_score , confusion_matrix
```

| Biblioteca | Para que serve neste trabalho |
|---|---|
| `pandas` | Ler os CSVs, ver tipos, valores ausentes e montar as tabelas de resultado |
| `numpy` | Contas com números: padronização, distâncias, KNN, regressão linear, MAE, MSE e R² |
| `matplotlib` e `seaborn` | Gráficos |
| `train_test_split` | Divisão estratificada de treino e teste (Tarefa 1) |
| `LogisticRegression` | Primeiro classificador da Tarefa 1 |
| `accuracy_score`, `precision_score`, `confusion_matrix` | Métricas da Tarefa 1 |

Como o conjunto de importações é pequeno, os outros algoritmos (KNN, centroide mais próximo, regressão linear, polinomial e KNN de regressão) foram escritos com `numpy`, em código simples com laços `for`. Recall, F1, MAE, MSE e R² também foram calculados manualmente.

---

## 6. Tarefa 1 — Classificação (ANEEL)

### 6.1 Dados

Cada linha do arquivo `aneel_classificacao_orange.csv` é **um empreendimento**.

| Coluna | Significado | Papel |
|---|---|---|
| `potencia_kw` | Potência outorgada (autorizada), em quilowatts | Entrada (X) |
| `latitude` | Latitude aproximada, em graus decimais | Entrada (X) |
| `longitude` | Longitude aproximada, em graus decimais | Entrada (X) |
| `fonte` | Solar, Eólica ou Hidráulica | Alvo (y) |

Foram descartadas as linhas sem potência ou sem coordenadas.

**Colunas que NÃO foram usadas como entrada:** `SigTipoGeracao`, nome do empreendimento, código CEG e descrições de combustível. Elas revelam a resposta e fariam o modelo "colar" em vez de aprender.

### 6.2 Exploração

- Verificação de valores ausentes, tamanho da tabela e tipos das colunas.
- Contagem de empreendimentos por classe, para ver se há desequilíbrio.
- Gráfico de barras com a quantidade por fonte.
- Histograma da potência por fonte, com escala logarítmica.
- Mapa de pontos (longitude × latitude) colorido pela fonte.

### 6.3 Preparação

1. **Transformação da potência:** foi usado `log(1 + potência)`, porque a potência vai de poucos kW a milhares de kW. Sem isso, os valores muito grandes dominariam as distâncias.
2. **Divisão treino/teste:** 80% para treino e 20% para teste, com `random_state=42` e **estratificada** (as classes mantêm a mesma proporção nos dois grupos).
3. **Padronização:** cada entrada foi transformada em `(valor − média) / desvio padrão`. A média e o desvio foram calculados **somente com o treino** e aplicados ao teste. Isso evita usar informação do teste durante o treino.

### 6.4 Os três classificadores

| # | Algoritmo | Como funciona | Por que foi escolhido |
|---|---|---|---|
| 1 | **Regressão Logística** | Aprende fronteiras retas para separar as classes | Modelo simples e muito usado como ponto de partida |
| 2 | **KNN (k = 5)** | Acha os 5 empreendimentos mais parecidos do treino e escolhe a classe mais comum entre eles | Consegue formar regiões de formato livre, o que ajuda com dados de localização |
| 3 | **Centroide mais próximo** | Calcula o ponto médio de cada classe e escolhe a classe com o ponto médio mais perto | Modelo bem simples, bom para comparar com os outros dois |

Os três usam **exatamente a mesma divisão** de treino e teste.

### 6.5 Métricas

| Métrica | O que mede |
|---|---|
| **Accuracy** | Porcentagem de acertos no total |
| **Precision** | De tudo o que o modelo previu como uma classe, quanto estava certo |
| **Recall** | De tudo o que realmente era de uma classe, quanto o modelo encontrou |
| **F1** | Combina precision e recall em um único número |

**Tipo de média:** Precision, Recall e F1 foram calculados por classe e depois combinados com a média **`macro`**. Na média macro, todas as classes têm o mesmo peso, mesmo que uma tenha menos exemplos.

Também foi feita a **matriz de confusão** de cada modelo. Nela, as linhas são as classes reais e as colunas são as classes previstas. Os valores fora da diagonal mostram os erros.

---

## 7. Tarefa 2 — Regressão (Open-Meteo)

### 7.1 Dados

Cada linha do arquivo `meteo_regressao_orange.csv` é **uma hora** entre 7h e 17h. As linhas ficam em ordem de data e hora.

| Coluna | Significado | Papel |
|---|---|---|
| `data_hora` | Data e hora local | Só para ordenar e separar treino e teste (não entra em X) |
| `temperatura_c` | Temperatura do ar a 2 m, em °C | Entrada (X) |
| `umidade_pct` | Umidade relativa a 2 m, em % | Entrada (X) |
| `nuvens_pct` | Cobertura de nuvens, em % | Entrada (X) |
| `vento_kmh` | Velocidade do vento a 10 m, em km/h | Entrada (X) |
| `hora` | Hora local, de 7 a 17 | Entrada (X) |
| `radiacao_w_m2` | Radiação solar global horizontal média da hora anterior, em W/m² | Alvo (y) |

Nenhum valor de `radiacao_w_m2` foi usado para criar uma entrada.

### 7.2 Exploração

- Verificação de valores ausentes, tamanho da tabela e tipos das colunas.
- Histograma da radiação solar.
- Boxplot da radiação por hora do dia.
- Mapa de correlação entre as variáveis.

### 7.3 Preparação

1. **Divisão no tempo:** as primeiras 80% das horas foram usadas para treino e as últimas 20% para teste. **Sem embaralhar.** Assim, o teste simula prever o "futuro" a partir do "passado".
2. **Padronização:** média e desvio padrão calculados apenas com o treino e aplicados ao teste.

### 7.4 Os três algoritmos de regressão

| # | Algoritmo | Como funciona | Por que foi escolhido |
|---|---|---|---|
| 1 | **Regressão Linear** | Ajusta uma reta (um plano) entre as entradas e a radiação | Modelo simples e fácil de interpretar |
| 2 | **Regressão Polinomial (grau 2)** | Igual à linear, mas também usa as entradas ao quadrado | Permite formar curvas, como o "morro" da radiação ao longo do dia |
| 3 | **KNN Regressor (k = 5)** | Prevê a média da radiação das 5 horas mais parecidas do treino | Não assume um formato fixo para a relação |

Os três usam **exatamente a mesma divisão** de treino e teste.

### 7.5 Métricas

| Métrica | Unidade | O que mede |
|---|---|---|
| **MAE** | W/m² | Erro médio, em valor absoluto. É a métrica mais fácil de entender |
| **MSE** | (W/m²)² | Média dos erros ao quadrado. Pune mais os erros grandes |
| **R²** | sem unidade | Quanto da variação da radiação o modelo explica. Perto de 1 é melhor; perto de 0, o modelo não é melhor que a média |

Foi feito um gráfico de **valores reais × valores previstos** para os três modelos. Quanto mais perto da linha vermelha tracejada, melhor a previsão.

Também foram calculados os coeficientes da regressão linear (para ver o peso de cada entrada) e o erro médio por hora do dia.

---

## 8. Resultados

> Preencha as tabelas abaixo com os números que aparecerem ao executar o notebook. Os valores dependem dos dados consultados nas APIs.

### 8.1 Classificação

Média das métricas por classe: **macro**.

| Modelo | Accuracy | Precision (macro) | Recall (macro) | F1 (macro) |
|---|---|---|---|---|
| Regressão Logística |  |  |  |  |
| KNN (k = 5) |  |  |  |  |
| Centroide mais próximo |  |  |  |  |

- **Modelo escolhido (maior F1 macro):** _______
- **Classes mais confundidas:** _______ previstas como _______ (número de casos: ___)
- **Quantidade de exemplos por classe:** Solar ___ · Eólica ___ · Hidráulica ___

Matrizes de confusão: veja o notebook (célula `#5.`) ou a pasta `figuras/`.

### 8.2 Regressão

| Modelo | MAE (W/m²) | MSE ((W/m²)²) | R² |
|---|---|---|---|
| Regressão Linear |  |  |  |
| Regressão Polinomial (grau 2) |  |  |  |
| KNN Regressor (k = 5) |  |  |  |

- **Melhor modelo (maior R²):** _______
- **Radiação média da amostra (W/m²):** _______
- **Horas com maior erro:** _______
- **Entrada com maior peso na regressão linear:** _______

Gráfico real × previsto: veja o notebook (célula `#11.`) ou a pasta `figuras/`.

---

## 9. Conclusões

### Classificação

- O modelo escolhido é o de maior **F1 macro**, porque essa métrica junta precision e recall e dá o mesmo peso a todas as classes, mesmo quando uma delas tem menos exemplos.
- O notebook mostra qual par de classes é mais confundido. Isso indica quais fontes têm potência e localização parecidas.
- **Potência e localização não bastam para uma aplicação real.** Usinas de fontes diferentes podem ter potências parecidas e ficar na mesma região. Faltam informações importantes, como tecnologia usada, data de operação, relevo, presença de rio e potencial de vento e sol do local.
- A potência usada é a **outorgada** (autorizada), e não a energia realmente gerada.
- Como a consulta tem limite por tipo, as proporções entre as fontes **não** refletem o Brasil.

### Regressão

- A radiação muda muito ao longo do dia: é baixa de manhã cedo, alta perto do meio-dia e cai no fim da tarde. Por isso a **`hora` tem peso grande** na previsão.
- Uma regressão linear só consegue desenhar retas. Já a regressão polinomial e o KNN conseguem seguir a curva do dia, e por isso tendem a errar menos.
- **Nuvens** reduzem a radiação: quanto mais nuvens, menor a radiação.
- **Estimar a radiação não prevê automaticamente a energia produzida por um sistema fotovoltaico.**
  - A radiação é medida em W/m² (potência por área). A energia é medida em kWh (potência ao longo do tempo).
  - A geração real depende também do tamanho e da eficiência dos painéis, da inclinação, da sujeira, da temperatura do painel, das perdas do inversor e da rede elétrica.

---

## 10. Limitações

- Os dados da ANEEL têm limite de linhas por tipo de geração, então não representam o país todo.
- As classes podem estar desequilibradas. Por isso foram usadas métricas macro e uma divisão estratificada.
- As coordenadas da ANEEL são aproximadas.
- A Tarefa 2 usa apenas **três meses** (abril a junho de 2025) e **um único local**. O modelo pode não funcionar bem em outras épocas do ano ou em outras cidades.
- Os dados de radiação são estimativas de modelo, e não medições de sensor.
- A hora do dia foi restrita a 7h–17h, o que deixa de fora a noite.
- Os hiperparâmetros (`k = 5`, grau 2) foram fixados e não foram otimizados.
- Cada tarefa usa uma única divisão de treino e teste. Os resultados podem mudar com outra divisão.

---

## 11. Segurança e credenciais

As duas APIs usadas são públicas e **não exigem token ou chave**. Nenhuma senha ou credencial está neste repositório. Se você usar outra API que exija credenciais, **não as publique** no GitHub.

---

## Fontes

- ANEEL — SIGA: https://dadosabertos.aneel.gov.br/dataset/siga-sistema-de-informacoes-de-geracao-da-aneel
- ANEEL — recurso e campos usados: https://dadosabertos.aneel.gov.br/dataset/siga-sistema-de-informacoes-de-geracao-da-aneel/resource/11ec447d-698d-4ab8-977f-b424d5deee6a
- Open-Meteo — histórico e unidades: https://open-meteo.com/en/docs/historical-weather-api
