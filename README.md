# 🏃 Human Activity Recognition (HAR)
### Processamento Digital de Sinais + Classificação Preditiva

![Python](https://img.shields.io/badge/Python-3.x-blue?logo=python)
![Scikit-Learn](https://img.shields.io/badge/Scikit--Learn-ML-orange?logo=scikit-learn)
![XGBoost](https://img.shields.io/badge/XGBoost-Classifier-green)
![Status](https://img.shields.io/badge/Status-Concluído-brightgreen)

---

## 📋 Sobre o Projeto

Projeto desenvolvido para a disciplina de **Processamento Digital de Sinais** na **Universidade Federal do Pará (UFPA)**.

O objetivo é reconhecer automaticamente atividades humanas — **Caminhar**, **Correr** e **Pular** — a partir de um dataset com dados brutos coletados de um acelerômetro, combinando um pipeline completo de **Processamento Digital de Sinais** com modelos de **Aprendizado de Máquina**, visando fazer uma análise a cerca de seus  desempenhos.

---

## 🎯 Pipeline do Projeto

```
Dados Brutos (CSV)
      │
      ▼
Validação da Taxa de Amostragem (fs ≈ 98.93 Hz)
      │
      ▼
Filtragem Butterworth em Cascata
  ├── Passa-Alta  (fc = 0.3 Hz) → remove componente DC da gravidade
  └── Passa-Baixa (fc = 20 Hz)  → remove ruído de alta frequência
      │
      ▼
Janelamento Deslizante (Sliding Window)
  └── Overlap de 75% → captura adequada da classe minoritária (salto)
      │
      ▼
Extração de Características (Feature Engineering)
  ├── Estatísticas: Média, Desvio Padrão, Autocorrelação
  └── Espectrais:   Pico FFT, Entropia Espectral
      │
      ▼
Treinamento e Validação (StratifiedKFold, K=5)
  ├── SVM
  ├── Random Forest
  └── XGBoost  ← melhor modelo (F1 Macro: 0.83)
```

---

## 📊 Dataset

| Atributo          | Valor                        |
|-------------------|------------------------------|
| Fonte             | Acelerômetro inercial        |
| Total de amostras | 4.305 registros              |
| Taxa de amostragem| ≈ 98.93 Hz                   |
| Eixos             | X, Y, Z (triaxial)           |
| Classes           | Caminhar, Correr, Pular      |

**Distribuição das classes:**
- Caminhar: 2.276 amostras
- Correr: 1.484 amostras
- Pular: 545 amostras

---

## ⚙️ Pré-processamento

### Filtragem Butterworth

A análise direta de sinais brutos de acelerômetro é ineficiente pois o sinal é composto por três componentes sobrepostos:

- **Aceleração dinâmica do corpo** → sinal de interesse (0.3 Hz – 20 Hz)
- **Gravidade estática** → componente DC (~0 Hz), removida pelo filtro passa-alta
- **Ruído e vibrações espúrias** → altas frequências (>20 Hz), removidas pelo filtro passa-baixa

### Janelamento Deslizante (Sliding Window)

- Segmentação dos sinais filtrados em janelas temporais
- Overlap de **75%** adotado para maximizar a captura da classe minoritária (salto)
  - Com 50% de overlap: apenas 8 janelas de salto
  - Com 75% de overlap: **15 janelas de salto** (evitando colapso do F1 Macro na validação cruzada)

### Extração de Características

Cada janela é convertida em um vetor de **15 atributos** (5 features × 3 eixos):

| Feature             | Domínio    | Captura                              |
|---------------------|------------|--------------------------------------|
| Média               | Temporal   | Nível médio de aceleração            |
| Desvio Padrão       | Temporal   | Intensidade/energia do movimento     |
| Autocorrelação      | Temporal   | Periodicidade e repetição do padrão  |
| Pico FFT            | Frequencial| Frequência dominante do movimento    |
| Entropia Espectral  | Frequencial| Complexidade/irregularidade do sinal |

---

## 🤖 Modelos e Resultados

Validação com **StratifiedKFold (K=5)**, métrica principal: **F1 Macro**.

| Modelo        | F1 Macro (média) | Característica principal              |
|---------------|-----------------|---------------------------------------|
| SVM           | ~0.74           | Dificuldade com sobreposição espectral |
| Random Forest | ~0.80           | Maior estabilidade (menor variância)  |
| **XGBoost**   | **~0.83**       | **Melhor teto preditivo**             |

### Análise das Matrizes de Confusão

- **SVM**: comportamento conservador; classificou 12 corridas como caminhadas e acertou apenas 8 de 15 saltos
- **Random Forest**: mais consistente; melhorou a classificação de corrida e salto em relação ao SVM
- **XGBoost**: melhor discriminação entre caminhar e correr (40/46 corridas corretas); manteve 10/15 saltos corretos

---

## 🛠️ Tecnologias Utilizadas

```
Python 3.x
├── numpy
├── pandas
├── matplotlib
├── seaborn
├── scipy
│   ├── signal   (filtros Butterworth, FFT)
│   └── stats    (entropia)
├── scikit-learn
│   ├── RandomForestClassifier
│   ├── SVC
│   ├── StandardScaler
│   ├── StratifiedKFold
│   └── cross_val_score / cross_val_predict
└── xgboost
    └── XGBClassifier
```

---

## 🚀 Como Executar

**1. Clone o repositório**
```bash
git clone https://github.com/Josafha-Pereira/projeto-har.git
cd projeto-har
```

**2. Instale as dependências**
```bash
pip install numpy pandas matplotlib seaborn scipy scikit-learn xgboost
```

**3. Adicione o dataset**

Coloque o arquivo `05_rotulado.csv` na raiz do projeto. O dataset deve conter as colunas:
- `time_s` — tempo em segundos
- `acc_x`, `acc_y`, `acc_z` — aceleração triaxial
- `comportamento` — rótulo da atividade (Caminhar / Correr / Pular)

**4. Execute**
```bash
python projeto_har.py
```

> O projeto foi originalmente desenvolvido no **Google Colab**. Para execução local, substituir `display()` por `print()` se necessário.

---

## 📁 Estrutura do Repositório

```
projeto-har/
│
├── HAR
│   ├── 05_rotulado.csv      # Dataset
│   ├── projeto_har.py       # Pipeline completo: PDS + ML    
└── README.md
```

---

## 📌 Conclusões

- A filtragem (Butterworth passa-alta + passa-baixa) isolou com êxito a aceleração corporal pura, eliminando gravidade e ruídos
- O ajuste do overlap de 50% para **75%** foi de grande importância para o desempenho na classe minoritária (salto)
- A extração de features estatísticas e espectrais gerou um **espaço de características com alta separabilidade**
- O **XGBoost** obteve o melhor desempenho geral (F1 Macro ≈ 0.83), enquanto o **Random Forest** se destacou como a escolha mais robusta e estável para uso em produção

---

## 👨‍💻 Autor

**Josafha Pereira de Carvalho**
Engenharia da Computação — UFPA 
[![GitHub](https://img.shields.io/badge/GitHub-Josafha--Pereira-black?logo=github)](https://github.com/Josafha-Pereira)