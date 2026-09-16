# 🏃 Human Activity Recognition (HAR)
### Processamento Digital de Sinais + Classificação Preditiva

![Python](https://img.shields.io/badge/Python-3.x-blue?logo=python)
![Scikit-Learn](https://img.shields.io/badge/Scikit--Learn-ML-orange?logo=scikit-learn)
![XGBoost](https://img.shields.io/badge/XGBoost-Classifier-green)
![Status](https://img.shields.io/badge/Status-Concluído-brightgreen)

---

## 📋 Sobre o Projeto

Projeto desenvolvido para a disciplina de **Processamento Digital de Sinais** na **Universidade Federal do Pará (UFPA)**.

O objetivo é reconhecer por classificação atividades humanas (**Caminhada**, **Corrida** e **Salto**) a partir de um dataset com dados brutos coletados de um acelerômetro, combinando um pipeline completo de **Processamento Digital de Sinais** com modelos de **Aprendizado de Máquina**, visando fazer uma análise acerca de seus desempenhos.

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
  ├── Passa-Alta  (fc = 0.3 Hz) → atenua componente DC da gravidade
  └── Passa-Baixa (fc = 20 Hz)  → atenua ruído de alta frequência
      │
      ▼
Janelamento Deslizante (Sliding Window)
  └── Overlap de 75% → aumenta a quantidade de janelas da classe minoritária (salto)
      │
      ▼
Extração de Características (Feature Engineering)
  ├── Estatísticas: Média, Desvio Padrão, Autocorrelação
  └── Espectrais:   Pico FFT, Entropia Espectral
      │
      ▼
Treinamento e Validação (5 folds por blocos temporais)
  ├── SVM
  ├── Random Forest  ← maior F1 Macro médio (0.77)
  └── XGBoost
```

---

## 📊 Dataset

| Atributo          | Valor                        |
|-------------------|------------------------------|
| Fonte             | Acelerômetro inercial        |
| Total de amostras | 4.305 registros              |
| Taxa de amostragem| ≈ 98.93 Hz                   |
| Eixos             | X, Y, Z (triaxial)           |
| Classes           | Caminhada, Corrida, Salto    |

**Distribuição das classes:**
- Caminhada: 2.276 amostras
- Corrida: 1.484 amostras
- Salto: 545 amostras

---

## ⚙️ Pré-processamento

### Filtragem Butterworth

A análise direta de sinais brutos de acelerômetro é ineficiente pois o sinal é composto por três componentes sobrepostos:

- **Aceleração dinâmica do corpo** → sinal de interesse (0.3 Hz – 20 Hz)
- **Gravidade estática** → componente DC (~0 Hz), atenuada pelo filtro passa-alta
- **Ruído e vibrações espúrias** → altas frequências (>20 Hz), atenuadas pelo filtro passa-baixa

### Janelamento Deslizante (Sliding Window)

- Segmentação dos sinais filtrados em janelas temporais, separadamente em cada trecho contínuo de uma mesma atividade
- Overlap de **75%** adotado para reduzir a chance de um movimento ficar dividido na borda entre janelas e aumentar a quantidade de exemplos da classe minoritária (salto)
- Foram geradas **125 janelas**: 68 de caminhada, 43 de corrida e 14 de salto

### Extração de Características

Cada janela é convertida em um vetor de **15 atributos** (5 features × 3 eixos):

| Feature             | Domínio    | Captura                              |
|---------------------|------------|--------------------------------------|
| Média               | Temporal   | Nível médio de aceleração            |
| Desvio Padrão       | Temporal   | Intensidade/energia do movimento     |
| Autocorrelação      | Temporal   | Periodicidade e repetição do padrão  |
| Pico FFT            | Frequencial| Frequência dominante do movimento    |
| Entropia Espectral  | Frequencial| desorganização do sinal |

---

## 🤖 Modelos e Resultados

Validação com 5 folds formados por blocos temporais dentro de cada classe, removendo do treino janelas que compartilham amostras com a validação. Métrica principal: **F1 Macro**.

| Modelo            | F1 Macro (média) | Desvio padrão | Característica principal |
|-------------------|------------------|---------------|--------------------------|
| SVM               | 0.68             | 0.189         | Menor desempenho médio |
| **Random Forest** | **0.77**         | 0.204         | Maior F1 Macro médio |
| XGBoost           | 0.73             | 0.183         | Desempenho intermediário |

A variação entre os folds foi alta nos três modelos.


---

## 🛠️ Tecnologias Utilizadas

```
Python 3.x
├── numpy
│   └── fft
├── pandas
├── matplotlib
├── seaborn
├── scipy
│   ├── signal   (filtros Butterworth)
│   └── stats    (entropia)
├── scikit-learn
│   ├── RandomForestClassifier
│   ├── SVC
│   ├── StandardScaler / LabelEncoder
│   └── f1_score / confusion_matrix
└── xgboost
    └── XGBClassifier
```

Ambiente de desenvolvimento: **VS Code + Jupyter Notebook**, com dependências isoladas em `.venv`.

---

## 🚀 Como Executar

**1. Clone o repositório**
```bash
git clone https://github.com/Josafha-pereira/Reconhecimento-de-Atividades-Humanas-HAR.git
cd Reconhecimento-de-Atividades-Humanas-HAR
```

**2. Crie o ambiente virtual**
```bash
python3 -m venv .venv
```

**3. Ative o ambiente**

Em Bash ou Zsh:
```bash
source .venv/bin/activate
```

No Fish:
```fish
source .venv/bin/activate.fish
```

**4. Instale as dependências**
```bash
python -m pip install numpy pandas matplotlib seaborn scipy scikit-learn xgboost ipykernel
```

**5. Abra o projeto no VS Code**
```bash
code .
```

**6. Execute o notebook**

Abra `notebook/HAR.ipynb`, selecione `.venv/bin/python` como kernel e execute as células em ordem.

O notebook lê o dataset em `dataset/05_rotulado.csv`, portanto a estrutura de pastas deve ser mantida.

---

## 📁 Estrutura do Repositório

```
Reconhecimento-de-Atividades-Humanas-HAR/
├── dataset/
│   └── 05_rotulado.csv
├── notebook/
│   └── HAR.ipynb
├── .gitignore
└── README.md
```

O diretório `.venv/` é criado localmente e não faz parte do repositório.

---

## 📌 Conclusões

- A filtragem Butterworth atenuou componentes fora da faixa definida antes do janelamento
- O overlap de **75%** aumentou a quantidade de janelas da classe salto, mantendo mais exemplos da classe minoritária para avaliação
- A extração de features estatísticas e espectrais reduziu cada janela de 128 × 3 para **15 características** utilizadas na classificação
- O **Random Forest** apresentou o maior F1 Macro médio, seguido pelo XGBoost e pelo SVM
- A variação entre os folds foi alta nos três modelos, mostrando que o desempenho depende do trecho temporal utilizado na validação
- Para este conjunto de dados e este pipeline de processamento, o **Random Forest** apresentou o melhor desempenho médio na F1 Macro e, apesar da variação alta, a maioria dos seus valores ficou acima de **0.74**
- A quantidade reduzida de janelas, principalmente da classe salto, ainda limita a estabilidade da avaliação e deve ser considerada

---

## 👨‍💻 Autor

**Josafha Pereira de Carvalho**
Engenharia da Computação — UFPA 
[![GitHub](https://img.shields.io/badge/GitHub-Josafha--Pereira-black?logo=github)](https://github.com/Josafha-Pereira)