# CIN0024 — Análise de Churn

**Miniprojeto 1 — Análise e Preparação dos Dados**, da disciplina Sistemas Inteligentes.

O projeto explora dados de clientes de uma operadora de telecomunicações e prepara uma base para estudar o cancelamento de contratos (*churn*). Esta é a primeira de três entregas da disciplina; a modelagem será desenvolvida posteriormente.

## Conteúdo

- [EDA.ipynb](notebooks/EDA.ipynb): análise exploratória, auditoria da qualidade e investigação das relações com churn.
- [preprocessing.ipynb](notebooks/preprocessing.ipynb): limpeza, padronização, criação de atributos, divisão treino/teste, imputação, codificação e escalonamento.

A análise utiliza a data-base **21/09/2026**, horizonte de churn até **20/11/2026** e histórico de chamados dos **90 dias anteriores, incluindo a data-base**. A grafia das categorias é padronizada em maiúsculas e sem acentos.

## Como executar

Com Python 3 e um ambiente virtual ativo, instale as dependências e abra o Jupyter:

```bash
python -m pip install -r requirements.txt
python -m jupyter notebook
```

Coloque o arquivo `base_churn_5000_VF2_aluno.csv`, fornecido pela disciplina, em `data/`. Essa pasta é ignorada pelo Git.

Execute primeiro `notebooks/EDA.ipynb` e depois `notebooks/preprocessing.ipynb`, com as células em ordem. No VS Code, selecione o ambiente virtual como kernel.

## Resultados e arquivos gerados

Após remover 20 duplicatas e 10 registros com cancelamento anterior à data-base, restam **4.990 clientes**: **3.493 no treino** e **1.497 no teste**, em uma divisão estratificada de 70%/30%. Os oito atributos de entrada resultam em 14 após a codificação.

O pré-processamento preserva o CSV original e exporta para `data/processed/`:

| Arquivo | Conteúdo |
|---|---|
| `base_analitica.csv` | Base limpa com datas e atributos derivados; ainda contém ausências. |
| `*_antes_imputacao.csv` | Treino e teste com as ausências preservadas. |
| `*_imputado.csv` | Dados com ausências preenchidas. |
| `*_preparado.csv` | Dados imputados, codificados e escalonados. |
| `preprocessador_template.joblib` | Pipeline sem ajuste para os futuros modelos. |

A imputação e o escalonamento são ajustados somente no treino. Para a futura validação cruzada, utilize `treino_antes_imputacao.csv` e o template dentro do pipeline com o modelo, reaprendendo os tratamentos em cada rodada. Os IDs e o alvo `churn` ficam fora dos preditores, e o teste permanece reservado para a avaliação final.

Mais detalhes sobre os arquivos estão em [data/processed/README.md](data/processed/README.md). O K-fold e o treinamento serão implementados no arquivo de modelagem.
