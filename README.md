# CIN0024 — Análise de Churn

Projeto da disciplina Sistemas Inteligentes (CIn-UFPE) sobre o cancelamento de contratos (*churn*) de clientes de uma operadora de telecomunicações.

Feito por: Cleber Lucas, Juliana Silva, Lucas Emanuel, Mauricio Andrey e Victória Pessoa.

- **Miniprojeto 1 — Análise e Preparação dos Dados:** análise exploratória, limpeza e preparação da base.
- **Miniprojeto 2 — Otimização Preditiva:** modelagem com Árvore de Decisão e XGBoost, com foco no recall da classe churn.

## Conteúdo

| Notebook | Conteúdo |
|---|---|
| [EDA.ipynb](notebooks/EDA.ipynb) | Análise exploratória, auditoria da qualidade e investigação das relações com churn. |
| [preprocessing.ipynb](notebooks/preprocessing.ipynb) | Limpeza, padronização, criação de atributos, divisão treino/teste, imputação, diagnóstico de normalidade e codificação. |
| [modeling.ipynb](notebooks/modeling.ipynb) | Métricas e baselines, Árvore de Decisão (pré × pós-poda, regras, importância), XGBoost (GridSearch × Optuna), overfitting, hipóteses da equipe e avaliação final no teste. |

A análise utiliza a data-base **21/09/2026**, horizonte de churn até **20/11/2026** e histórico de chamados dos **90 dias anteriores, incluindo a data-base**. A grafia das categorias é padronizada em maiúsculas e sem acentos.

## Como executar

Com Python 3 e um ambiente virtual ativo, instale as dependências e abra o Jupyter:

```bash
python -m pip install -r requirements.txt
python -m jupyter notebook
```

O arquivo original `data/base_churn_5000_VF2_aluno.csv` já está no repositório. Execute os notebooks nesta ordem, com as células em sequência: `EDA.ipynb` → `preprocessing.ipynb` → `modeling.ipynb`. No VS Code, selecione o ambiente virtual como kernel. O `modeling.ipynb` leva cerca de 10 minutos para rodar por completo (buscas de hiperparâmetros), e todos os resultados são reproduzíveis com a semente 42.

## Dados preparados (Miniprojeto 1)

Após remover 20 duplicatas e 10 registros com cancelamento anterior à data-base, restam **4.990 clientes**: **3.493 no treino** e **1.497 no teste**, em uma divisão estratificada de 70%/30%. A taxa de churn é de **35,5%** nos dois conjuntos.

Os oito atributos de entrada continuam sendo oito após a codificação ordinal de contrato e pagamento. Como nenhuma variável numérica apresentou distribuição normal no treino, nenhuma recebe `StandardScaler` e todas permanecem na escala original.

O pré-processamento preserva o CSV original e exporta para `data/processed/`:

| Arquivo | Conteúdo |
|---|---|
| **`base_tratada.csv`** | **Base final transformada (entregável 3):** treino e teste preparados, com `id_cliente`, as 8 features, `churn` e a coluna `conjunto`. |
| `base_analitica.csv` | Base limpa com datas e atributos derivados; ainda contém ausências. |
| `*_antes_imputacao.csv` | Treino e teste com as ausências preservadas; usados na modelagem. |
| `*_imputado.csv` | Dados com ausências preenchidas. |
| `*_preparado.csv` | Dados imputados e codificados (ordinal), prontos para os modelos. |
| `preprocessador_template.joblib` | Pipeline de pré-processamento sem ajuste. |

Na modelagem, o template é encadeado a cada modelo e reajustado dentro de cada rodada da validação cruzada, a partir dos arquivos `*_antes_imputacao.csv`. O notebook verifica que esse pipeline reproduz exatamente a base tratada. Mais detalhes em [data/processed/README.md](data/processed/README.md).

## Modelagem (Miniprojeto 2)

**Protocolo:** validação cruzada estratificada com 5 partes no treino, busca de hiperparâmetros pela PR-AUC e limiar de decisão escolhido para garantir **recall de 80%** na classe churn. O teste foi usado uma única vez, na avaliação final.

**Resultados no teste** (1.497 clientes):

| Modelo | Recall | Precisão | F1 | ROC-AUC | PR-AUC |
|---|---|---|---|---|---|
| Regressão logística (referência) | 0,780 | 0,555 | 0,648 | 0,801 | 0,694 |
| XGBoost (GridSearch) | 0,774 | 0,546 | 0,641 | 0,794 | 0,682 |
| Árvore de Decisão (pré-poda) | 0,737 | 0,509 | 0,602 | 0,752 | 0,622 |

**Principais conclusões:**

- O XGBoost supera claramente a Árvore de Decisão e empata com a regressão logística, o que indica relações em boa parte aditivas entre os atributos e o churn.
- A pré-poda (`min_samples_leaf=50`, `max_depth=8`) venceu a pós-poda por `ccp_alpha` e reduziu o gap de recall da árvore de 0,51 para 0,03.
- O Optuna alcançou o mesmo resultado do GridSearch com 6× menos ajustes de modelo.
- Regra de maior risco: **contrato mensal, até 28 meses de casa e 3 ou mais chamados em 90 dias → 74,5% de churn** (média da base: 35,5%).
- Fatores de risco: tipo de contrato, tempo de cliente, chamados recentes e atrasos. A forma de pagamento não contribui.
- Hipóteses testadas sem ganho relevante: one-hot × ordinal, ausentes nativos, atributos derivados e XGBoost monotônico (mesmo desempenho, com coerência de negócio).
