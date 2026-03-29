# 📊 Propensity Score Matching — Escolas Católicas e Desempenho Acadêmico

Análise do efeito causal de frequentar uma escola católica no desempenho em matemática de alunos do ensino fundamental, usando a técnica de **Propensity Score Matching (PSM)**.

---

## 📁 Estrutura do Projeto

```
.
├── propensity_score_matching.ipynb   # Notebook principal com toda a análise
├── ecls.csv                          # Base de dados (Early Childhood Longitudinal Study)
└── README.md                         # Este arquivo
```

---

## ⚙️ Instalação das Dependências

Recomenda-se usar um ambiente virtual antes de instalar:

```bash
python -m venv venv
source venv/bin/activate        # Linux/macOS
venv\Scripts\activate           # Windows
```

Instale os pacotes necessários com:

```bash
pip install numpy pandas scipy scikit-learn matplotlib seaborn jupyter
```

Ou, se preferir usar um arquivo `requirements.txt`:

```bash
pip install -r requirements.txt
```

**`requirements.txt` sugerido:**

```
numpy>=1.24
pandas>=2.0
scipy>=1.10
scikit-learn>=1.3
matplotlib>=3.7
seaborn>=0.12
jupyter>=1.0
```

> ✅ Não são necessários pacotes R ou bibliotecas externas. Tudo roda com o ecossistema padrão do Python científico.

---

## ▶️ Como Executar

1. Clone ou baixe este repositório e coloque `ecls.csv` na mesma pasta do notebook.

2. Inicie o Jupyter:

```bash
jupyter notebook
```

3. Abra o arquivo `propensity_score_matching.ipynb` e execute as células em ordem (`Kernel > Restart & Run All`).

As visualizações são geradas inline no próprio notebook.

---

## 📐 Metodologia — Propensity Score Matching (PSM)

### Contexto

Em estudos observacionais, os grupos de tratamento e controle raramente são equivalentes. Alunos de escolas católicas tendem a ter perfil socioeconômico mais favorável do que alunos de escolas não católicas, o que introduz **viés de seleção** em qualquer comparação direta de médias.

### O que é o PSM?

O **Propensity Score Matching** é uma técnica de inferência causal que busca simular as condições de um experimento controlado usando dados observacionais. A ideia central é:

> *Comparar indivíduos que têm a mesma probabilidade de receber o tratamento, mas que de fato receberam tratamentos diferentes.*

### Etapas da Análise

```
1. Análise exploratória
      └─ Comparação de médias e testes t entre grupos (antes do matching)

2. Estimativa do Propensity Score
      └─ Regressão logística: P(católica | raça, renda, idade mãe, escolaridade mãe, mobilidade)

3. Matching
      └─ Nearest Neighbor com caliper = 0.25 × DP(propensity score)
      └─ Sem reposição — cada controle é usado no máximo uma vez

4. Verificação do balanceamento
      └─ Testes t das covariáveis após o matching
      └─ Love Plot (Standardized Mean Difference)

5. Estimativa do efeito do tratamento
      └─ Teste t
      └─ Regressão linear simples (nota ~ catholic)
      └─ Regressão linear completa (nota ~ catholic + covariáveis)
```

### Intuição do Caliper

O caliper limita a distância máxima permitida entre os propensity scores de um par. Usar `0.25 × DP` é a regra de bolso mais difundida na literatura (Austin, 2011), e evita matches de baixa qualidade que poderiam introduzir novo viés.

---

## 📋 Descrição das Variáveis

A base de dados utilizada é a **ECLS** (*Early Childhood Longitudinal Study*), um estudo longitudinal americano de larga escala sobre desenvolvimento infantil.

| Variável | Tipo | Descrição |
|---|---|---|
| `catholic` | Binária | **Variável de tratamento.** `1` = escola católica, `0` = escola não católica |
| `c5r2mtsc_std` | Contínua | **Variável de resultado.** Nota padronizada (z-score) no teste de matemática |
| `race_white` | Binária | `1` = aluno branco, `0` = outra raça |
| `p5hmage` | Contínua | Idade da mãe (em anos) |
| `w3income_1k` | Contínua | Renda familiar anual (em milhares de dólares) |
| `p5numpla` | Inteira | Número de lugares onde o aluno viveu por pelo menos 4 meses |
| `w3momed_hsb` | Binária | `1` = mãe com ensino médio completo ou menos, `0` = nível superior ou acima |

### Variáveis auxiliares (presentes no CSV, não usadas na análise)

| Variável | Descrição |
|---|---|
| `childid` | Identificador único do aluno |
| `race` | Raça em formato texto |
| `race_black`, `race_hispanic`, `race_asian` | Dummies de raça |
| `w3daded`, `w3momed` | Escolaridade do pai/mãe em formato texto |
| `w3inccat` | Faixa de renda categórica |
| `w3income` | Renda familiar bruta |
| `c5r2mtsc` | Nota bruta de matemática (antes da padronização) |

---

## 📊 Principais Resultados

| | Antes do Matching | Após o Matching |
|---|---|---|
| Média — Não Católica | ~0.163 | ~0.27 |
| Média — Católica | ~0.220 | ~0.09 |
| Diferença (católica − não católica) | +0.057 | **−0.18** |
| p-valor | ~0.09 *(não significativo)* | **< 0.0001** |

> Antes do matching, a diferença aparente favorecia escolas católicas — mas ela era explicada pelo perfil socioeconômico mais privilegiado dos seus alunos. Após o pareamento, o efeito se inverte: alunos de escolas **não católicas** com perfil similar apresentam desempenho significativamente **melhor** em matemática.

---

## 📚 Referências

- Rosenbaum, P. R., & Rubin, D. B. (1983). The central role of the propensity score in observational studies for causal effects. *Biometrika*, 70(1), 41–55.
- Austin, P. C. (2011). Optimal caliper widths for propensity-score matching. *Pharmaceutical Statistics*, 10(2), 150–161.
- [R Tutorial 8: Propensity Score Matching](https://simonejdemyr.com/r-tutorials/statistics/tutorial8.html)
- [HW11a — PSM and LASSO (RPubs)](https://rpubs.com/metricsdawg/1037525)
- [Avaliação de Políticas Públicas — Canal da Quaest](https://www.youtube.com/watch?v=uMJeojTOcxc)
