#Previsão de Aptidão para Cultivo de Milho na África – Mudanças Climáticas (ArcGIS Pro)#

Projeto desenvolvido durante o curso GIS for Climate Action (Esri). O objetivo é modelar a aptidão atual e futura para o cultivo de milho em uma região da África Subsaariana utilizando o modelo de machine learning Presence-Only Prediction (MaxEnt), comparando variáveis bioclimáticas baseline com o cenário climático SSP3 7.0 para 2055.

Cenário
Em muitas partes da África, a segurança alimentar e de renda está diretamente ligada à produção agrícola do milho. Com as mudanças climáticas, temperaturas mais extremas e secas mais frequentes ameaçam essa segurança para populações vulneráveis.
Como analista GIS de uma ONG global, fui responsável por construir um modelo preditivo que identificasse onde o milho pode ser cultivado hoje e onde essa aptidão será comprometida até 2055, auxiliando ministros da agricultura a planejar ações de resiliência climática.

Objetivos

Preparar o dataset de treinamento com pontos de presença de milho e variáveis explicativas
Treinar o modelo MaxEnt com 19 variáveis bioclimáticas baseline
Executar o modelo com variáveis projetadas para 2055 (SSP3 7.0)
Revisar os diagnósticos de desempenho do modelo
Comparar visualmente a aptidão baseline com a futura usando o Swipe Tool


Estrutura dos Dados
Pontos de presença (PresencePoints):

100.000 pontos aleatórios dentro da área de estudo (7 países da África Subsaariana)
Identificação de presença cruzando dois datasets:

GFSAD.tif (NASA) – identificação de culturas dominantes por satélite
Africa Crop Maize – Harvested Area (SPAM) – área colhida de milho (1999–2001)


Campo MaizePresent: 1 = presença confirmada em ambos os datasets, 0 = ausência

Variáveis explicativas (ArcGIS Living Atlas – CHELSA):

19 variáveis bioclimáticas baseline (1981–2010)
19 variáveis bioclimáticas projetadas para 2055 (SSP3 7.0 / SSP370)
Subconjuntos extraídos via Subset Multidimensional Raster

Pontos de predição (PredictionPoints):

~5,2 milhões de pontos gerados a partir do raster GFSAD clipado para a área de estudo


Metodologia
1. Preparação dos dados de presença 

Exploração das variáveis bioclimáticas CHELSA no ArcGIS Living Atlas
Subsetting dos rasters multidimensionais para isolar baseline (1995) e projeção (SSP370, 2055)
Geração de 100.000 pontos aleatórios com Create Spatial Sampling Locations
Reclassificação dos datasets GFSAD e SPAM para identificar presença de milho (valores 1/0)
Extração dos valores raster para os pontos com Extract Multi Values To Points
Criação do campo MaizePresent por Select by Attributes + Calculate Field
Adição das 19 variáveis bioclimáticas baseline ao dataset de treinamento

2. Criação dos pontos de predição 
Clip do GFSAD_Maize para a área de estudo com Extract By Mask
Conversão do raster para pontos com Raster To Point (~5,2 milhões de pontos)
Adição das 19 variáveis baseline + 19 variáveis projetadas 2055 aos pontos de predição

3. Treinamento do modelo 

Ferramenta: Presence-Only Prediction (MaxEnt) (Spatial Statistics Tools)
Input: PresencePoints com campo MaizePresent e opção Contains Background Points
19 variáveis baseline como Explanatory Training Variables
Expansões de variáveis: Linear, Quadrática, Produto e Hinge
Apply Spatial Thinning: distância mínima de 5 km
Outputs de treinamento: TrainedFeatures, CurveTable, SensitivityTable

4. Execução do modelo com projeções futuras 

Reexecução da ferramenta MaxEnt com as variáveis projetadas para 2055 (SSP370)
Match das variáveis: BioclimateProjections01–19_2055 ↔ BaselineBioclimate01–19
Output: FutureMaizeSuitability

5. Conversão e comparação dos resultados 
Conversão dos layers de ponto para raster com Point To Raster
Simbologia Yellow-Green (Continuous), Stretch Type: Minimum Maximum
Basemap alterado para Imagery para maior contraste visual
Comparação visual com Swipe Tool entre baseline e futuro


Resultados
CamadaDescriçãoBaselineMaizeSuitability_RasterAptidão atual (baseline 1981–2010)FutureMaizeSuitability_RasterAptidão projetada para 2055 (SSP3 7.0)
A comparação evidencia uma retração significativa das áreas aptas para o cultivo de milho até meados do século, especialmente nas regiões de menor altitude e maior exposição ao estresse térmico.
Aptidão Baseline (1981–2010)

<img width="1568" height="640" alt="image" src="https://github.com/user-attachments/assets/55c42944-4009-4f71-a35d-5ca15eee934a" />

Aptidão Futura – SSP3 7.0 (2055)
<img width="1568" height="661" alt="image" src="https://github.com/user-attachments/assets/a1d497d4-fc26-4f03-abe8-e81d8ce233c3" />


Diagnósticos do Modelo
Resposta Parcial de Variáveis Contínuas

<img width="1397" height="810" alt="image" src="https://github.com/user-attachments/assets/c3dd70c6-c975-49a0-9dd1-bfb8939234c8" />

Cada mini-gráfico mostra o impacto de uma das 19 variáveis bioclimáticas na probabilidade de presença do milho. Em destaque, a variável BaselineBioclimate07 (amplitude térmica anual): a probabilidade sobe rapidamente entre 0°C e ~15°C, mantém-se elevada até ~25°C e cai abruptamente acima disso — comportamento coerente com a fisiologia do milho.

Gráfico de ROC

<img width="1392" height="810" alt="image" src="https://github.com/user-attachments/assets/8a864758-2ca0-461f-a1d7-7cbb8d811ec8" />

A curva ROC avalia a capacidade do modelo de distinguir presença de ausência. A curva bem acima da diagonal indica bom poder discriminativo — o modelo identifica áreas aptas muito além do acaso.

Porcentagens de Resultados de Classificação (Cortar = 0,5)
<img width="1397" height="802" alt="image" src="https://github.com/user-attachments/assets/6538c533-2c58-4357-9548-16dd44676056" />


Barra Presença: ~71% azul (Correctly Classified) — maioria dos pontos de presença corretamente identificada
Barra Background: ~80% cinza (Unchanged) — áreas sem presença corretamente mantidas como ausência

Desempenho considerado satisfatório para os propósitos desta análise exploratória.

🛠️ Ferramentas Utilizadas

ArcGIS Pro 3.6 – geoprocessamento, machine learning, simbologia
ArcGIS Spatial Analyst Extension – Extract By Mask, Reclassify, Point To Raster
ArcGIS Spatial Statistics Tools – Presence-Only Prediction (MaxEnt)
ArcGIS Living Atlas – variáveis bioclimáticas CHELSA (baseline e projeções SSP)
ArcGIS Online – dados SPAM (Africa Crop Maize – Harvested Area)


Autora
Camila Mariana Neri Rosa – Graduação em Oceanografia (UERJ)
LinkedIn | GitHub
