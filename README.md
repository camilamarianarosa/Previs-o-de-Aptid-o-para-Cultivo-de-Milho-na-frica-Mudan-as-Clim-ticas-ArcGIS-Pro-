# Previsão de Aptidão para Cultivo de Milho na África – Mudanças Climáticas (ArcGIS Pro)

Projeto desenvolvido durante o curso *GIS for Climate Action* (Esri). O objetivo é modelar a aptidão atual e futura para o cultivo de milho em uma região da África Subsaariana utilizando o modelo de machine learning **Presence-Only Prediction (MaxEnt)**, comparando variáveis bioclimáticas baseline com o cenário climático **SSP3 7.0 para 2055**.

---

## Cenário

Em muitas partes da África, a segurança alimentar e de renda está diretamente ligada à produção agrícola do milho. Com as mudanças climáticas, temperaturas mais extremas e secas mais frequentes ameaçam essa segurança para populações vulneráveis.

Como analista GIS de uma ONG global, fui responsável por construir um modelo preditivo que identificasse onde o milho pode ser cultivado hoje e onde essa aptidão será comprometida até 2055, auxiliando ministros da agricultura a planejar ações de resiliência climática.

---

## Objetivos

- Preparar o dataset de treinamento com pontos de presença de milho e variáveis explicativas
- Treinar o modelo MaxEnt com 19 variáveis bioclimáticas baseline
- Executar o modelo com variáveis projetadas para 2055 (SSP3 7.0)
- Revisar os diagnósticos de desempenho do modelo
- Comparar visualmente a aptidão baseline com a futura usando o Swipe Tool

---

## Estrutura dos Dados

**Pontos de presença (PresencePoints):**
- 100.000 pontos aleatórios dentro da área de estudo (7 países da África Subsaariana)
- Identificação de presença cruzando dois datasets:
  - GFSAD.tif (NASA) – identificação de culturas dominantes por satélite
  - Africa Crop Maize – Harvested Area (SPAM) – área colhida de milho (1999–2001)
- Campo `MaizePresent`: 1 = presença confirmada em ambos os datasets, 0 = ausência

**Variáveis explicativas (ArcGIS Living Atlas – CHELSA):**
- 19 variáveis bioclimáticas baseline (1981–2010)
- 19 variáveis bioclimáticas projetadas para 2055 (SSP3 7.0 / SSP370)
- Subconjuntos extraídos via *Subset Multidimensional Raster*

**Pontos de predição (PredictionPoints):**
- ~5,2 milhões de pontos gerados a partir do raster GFSAD clipado para a área de estudo

---

## Metodologia

### 1. Preparação dos dados de presença (Exercise 1)

- Exploração das variáveis bioclimáticas CHELSA no ArcGIS Living Atlas
- Subsetting dos rasters multidimensionais para isolar baseline (1995) e projeção (SSP370, 2055)
- Geração de 100.000 pontos aleatórios na área de estudo com *Create Spatial Sampling Locations*
- Reclassificação dos datasets GFSAD e SPAM para identificar presença de milho (valores 1/0)
- Extração dos valores raster para os pontos com *Extract Multi Values To Points*
- Criação do campo `MaizePresent` por *Select by Attributes* + *Calculate Field*
- Adição das 19 variáveis bioclimáticas baseline ao dataset de treinamento

### 2. Criação dos pontos de predição (Exercise 2 – Step 1 e 2)

- Clip do GFSAD_Maize para a área de estudo com *Extract By Mask*
- Conversão do raster para pontos com *Raster To Point* (~5,2 milhões de pontos)
- Adição das 19 variáveis baseline + 19 variáveis projetadas 2055 aos pontos de predição

### 3. Treinamento do modelo (Step 3)

- Ferramenta: *Presence-Only Prediction (MaxEnt)* (Spatial Statistics Tools)
- Input: `PresencePoints` com campo `MaizePresent` e opção *Contains Background Points*
- 19 variáveis baseline como *Explanatory Training Variables*
- Expansões de variáveis: Linear, Quadrática, Produto e Hinge
- *Apply Spatial Thinning*: distância mínima de 5 km
- Peso relativo Presença/Background: 100 (padrão)
- Outputs de treinamento: `TrainedFeatures`, `CurveTable`, `SensitivityTable`

### 4. Execução do modelo com projeções futuras (Step 4)

- Reexecução da ferramenta MaxEnt com as variáveis projetadas para 2055 (SSP370)
- Match das variáveis: BioclimateProjections01–19_2055 ↔ BaselineBioclimate01–19
- Output: `FutureMaizeSuitability`

### 5. Diagnósticos do modelo (Step 5)

- **Partial Response of Continuous Variables**: análise do impacto de cada variável na probabilidade de presença
- **BaselineBioclimate07** (amplitude térmica anual): probabilidade máxima entre 10–15°C, declínio acima de 25°C
- **Classification Result Percentages (Cutoff = 0.5)**:
  - Presença corretamente classificada: ~72%
  - Background unchanged: ~81%
  - Desempenho considerado adequado para o propósito da análise

### 6. Conversão e comparação dos resultados (Steps 6 e 7)

- Conversão dos layers de ponto para raster com *Point To Raster* (campo: Probability of Presence, cellsize: GFSAD.tif)
- Simbologia Yellow-Green (Continuous), Stretch Type: Minimum Maximum
- Basemap alterado para Imagery para maior contraste visual
- Comparação visual com **Swipe Tool** entre `BaselineMaizeSuitability_Raster` e `FutureMaizeSuitability_Raster`

---

## Resultados

| Camada | Descrição |
|---|---|
| `BaselineMaizeSuitability_Raster` | Aptidão atual (baseline 1981–2010) |
| `FutureMaizeSuitability_Raster` | Aptidão projetada para 2055 (SSP3 7.0) |

A comparação entre os dois rasters evidencia uma **retração significativa das áreas aptas para o cultivo de milho** até meados do século, especialmente nas regiões de menor altitude e maior exposição ao estresse térmico. As áreas com maior aptidão futura concentram-se em zonas de transição climática, onde as condições permanecem dentro do intervalo ótimo de temperatura para o milho.

A análise permite que ministros da agricultura identifiquem onde priorizar ações de adaptação, redistribuição de cultivos e planejamento de segurança alimentar.

---

## Imagens

### Aptidão Baseline
<img width="1568" height="640" alt="image" src="https://github.com/user-attachments/assets/c0543e61-1bf3-4901-b308-2814916df0e9" />


### Aptidão Futura (SSP3 7.0 – 2055)
<img width="1568" height="661" alt="image" src="https://github.com/user-attachments/assets/0ffbc959-0333-40a9-8037-f467ae8b94fb" />


---

## 🛠️ Ferramentas Utilizadas

- **ArcGIS Pro 3.6** – geoprocessamento, machine learning, simbologia
- **ArcGIS Spatial Analyst Extension** – Extract By Mask, Reclassify, Point To Raster
- **ArcGIS Spatial Statistics Tools** – Presence-Only Prediction (MaxEnt)
- **ArcGIS Living Atlas** – variáveis bioclimáticas CHELSA (baseline e projeções SSP)
- **ArcGIS Online** – dados SPAM (Africa Crop Maize – Harvested Area)

---

## Autora

**Camila Mariana Neri Rosa** – Graduação em Oceanografia (UERJ)

[LinkedIn](#) | [GitHub](#)
