# Projeto Final — Pipeline para Leitura Automatizada do Número de Identificação de Veículos (VIN)

## Descrição Geral

Este projeto tem como objetivo o **reconhecimento automatizado do Número de Identificação de Veículos (VIN)** a partir de imagens fotográficas reais.  
O pipeline desenvolvido é composto por **três etapas principais**, utilizando modelos da família **YOLOv11** e uma **Rede Neural ResNet18** para refino de predições.

A abordagem visa identificar, localizar e classificar corretamente todos os **17 caracteres alfanuméricos** que compõem o VIN, mesmo em condições adversas como iluminação irregular, ruídos visuais e diferentes materiais de gravação.  

Os 17 caracteres de um VIN são compostos pelos números de 0 até 9, e dos caracteres de A até Z, excluindo I, O e Q, de acordo com as normas ISO 3779 e NBR 6066.  
<img width="669" height="125" alt="vin_structure" src="https://github.com/user-attachments/assets/e1218830-9dcc-4d32-9dc9-332fa6e890c5" />

O link abaixo apresenta as variações do WMI presentes em veículos e tem uma maior explicação sobre o VIN em sua totalidade.  
<https://en.wikipedia.org/wiki/Vehicle_identification_number>

---

## 🧠 Pipeline de Inferência

A figura abaixo ilustra uma amostra real de VIN (note que os asteriscos presentes nas laterais não fazem parte dos 17 caracteres):
<img width="1024" height="715" alt="chassi" src="https://github.com/user-attachments/assets/c628cb0c-8fd6-494d-a1e1-1c588151baee" />

O fluxo de execução é o seguinte:

1. **Entrada:** imagem de um veículo contendo o VIN.  
2. **Pré-processamento:** essa imagem é convertida para escala de cinza, passa por CLAHE, suavização e sharpening.  
3. **Detecção da ROI:** o YOLOv11 localiza e recorta a região contendo o VIN (acontece na mesma Etapa 2).  
4. **Detecção de caracteres:** o YOLOv11 detecta as bounding boxes dos caracteres.  
5. **Classificação e fusão de resultados:** cada caractere detectado é classificado pela ResNet18; em caso de conflito entre os modelos, prevalece a classe com maior confiança.  
6. **Reconstrução do VIN:** os caracteres são ordenados da esquerda para a direita, gerando o número completo.  

---

## Estrutura do Pipeline

O pipeline completo é dividido em **três fases principais**, conforme a figura abaixo:

<img width="791" height="591" alt="etapa5" src="https://github.com/user-attachments/assets/856f547b-af5f-4f95-9ef0-d5ebefc6a297" />

### **1️⃣ Detecção da Região de Interesse (ROI)**

- **Modelo:** YOLOv11s (ROI Detector)  
- **Objetivo:** identificar e recortar automaticamente a área onde o VIN está presente na imagem original.  
- **Dataset:** composto por amostras positivas (contendo VIN) e negativas (sem VIN) para aumentar a robustez do modelo.  

- A Figura abaixo ilustra os resultados obtidos no treinamento do modelo.
<img width="2400" height="1200" alt="results" src="https://github.com/user-attachments/assets/5cdf7dca-14ea-4105-9766-697aeeab67a7" />
- Esses resultados indicam que o modelo tem **alta capacidade de localizar corretamente a área do VIN**, com poucas detecções incorretas.

---

- A matriz de confusão abaixo ilustra a capacidade do modelo de reconhecer imagens com VINs e sem VINs.  
- Curiosamente, os casos negativos não estão sendo mostrados nessa matriz de confusão, mas eles existem e compõem cerca de 20-40% das imagens utilizadas.
<img width="3000" height="2250" alt="confusion_matrix" src="https://github.com/user-attachments/assets/7ff3954d-fde7-4b8a-88ce-84848681d81e" />

---

### **2️⃣ Detecção de Caracteres**

- **Modelo:** YOLOv11s (Character Detector)  
- **Objetivo:** detectar individualmente cada caractere (0–9, A–Z) dentro da ROI detectada.  
- O modelo é responsável por gerar bounding boxes correspondentes a cada caractere.  
- Foi aplicado um processo de **aumento de dados (data augmentation)** para melhorar a generalização do modelo.
- **Resultados:**
  - Precisão: 0.9512  
  - Recall: 0.8936  
  - mAP@0.50: 0.9552  
  - mAP@0.50-0.95: 0.7077  
  - Fitness: 0.7325  

- A Figura abaixo ilustra os valores alcançados pelo modelo modelo de Detecção de Caracteres utilizado.
<img width="2400" height="1200" alt="results" src="https://github.com/user-attachments/assets/c2d56e41-7e28-4658-ac50-c1a324bdaccc" />

---

- A matriz de confusão abaixo ilustra os valores previstos pelo modelo, perante os valores (caracteres) reais.
<img width="3000" height="2250" alt="confusion_matrix_normalized" src="https://github.com/user-attachments/assets/6f06eb9b-392f-48a7-97a4-15f95ba60d13" />

---

### **3️⃣ Classificação de Caracteres**

- **Modelo:** ResNet18  
- **Função:** atuar como refinador das predições do YOLO, corrigindo erros de classificação em caracteres visualmente semelhantes (como “X” e “K”, “B” e “8”, “Z” e “2”).  
- A rede foi treinada sobre um dataset de caracteres alfanuméricos (0–9, A–Z, excluindo I, O e Q), atingindo:
  - Precisão Macro: **97.77%**

- A matriz de confusão abaixo ilustra a capacidade do modelo de reconhecer os caracteres corretamente.
<img width="788" height="701" alt="resnet18_confusion" src="https://github.com/user-attachments/assets/c31b9702-f6bf-4688-a39c-0320da711e0b" />


Durante a inferência, caso o YOLO e a ResNet18 **concordem na predição**, a classe é mantida.  
Se houver **divergência**, é escolhida a classe com **maior grau de confiança**.

---

## 🧪 Resultados do Pipeline

O pipeline completo foi avaliado sobre o conjunto de validação e teste, gerando um relatório detalhado (`vin_results_report.csv`).  
As métricas indicam **alto desempenho geral** e excelente robustez no reconhecimento completo do VIN.

<img width="687" height="471" alt="taxa_erro_acumulada" src="https://github.com/user-attachments/assets/787031ce-f6a6-4053-8143-c5d47e60ceb4" />

| Métrica | Valor |
|----------|--------|
| Acurácia de leitura completa (0 erros) | 87.5% |
| ≤1 erros por VIN | 91.5% |
| ≤2 erros por VIN | 92.7% |

Esses resultados mostram que o sistema é capaz de **ler corretamente a grande maioria dos VINs completos**, mesmo em ambientes com ruído, reflexo e variação de foco.  

A Figura abaixo ilustra a frequência de VINs que apresentaram nenhum ou mais que 1 erro de leitura, permitindo analisar aonde estão concentrados os maiores erros do modelo.  
<img width="790" height="490" alt="distribuicao_erros" src="https://github.com/user-attachments/assets/8d9c1596-77a1-43e6-90bb-be6b93ad7e86" />

---

## ⚠️ Disclaimer do autor
- É importante salientar que as bases de dados pré-processadas e prontas para uso não estão adicionados ao projeto no GitHub, 
além disso, nem todos os pré-processamentos, filtros, manipulações, dentre outros, 
estão presentes no código do "projeto_final.ipynb" e "resnet18_classifier.ipynb".  
- Alguns processos nos datasets foram aplicados de forma manual, por meio de websites como o CVAT, impactando na reprodutibilidade do projeto.  
- Os datasets originais utilizados são descritos no arquivo "projeto_final.ipynb" e estão disponíveis no website Roboflow.
