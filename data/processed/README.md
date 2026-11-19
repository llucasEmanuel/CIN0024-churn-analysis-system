# Dados preparados para previsao de churn

- base_tratada.csv: ENTREGAVEL 3. Base final transformada (treino + teste), com
  id_cliente, as 8 features preparadas, churn e a coluna conjunto (treino/teste).
  E a uniao de treino_preparado.csv e teste_preparado.csv.
- base_analitica.csv: base limpa, datas e atributos derivados; ainda contem ausencias.
- *_antes_imputacao.csv: oito features com ausencias, id_cliente e alvo churn.
- *_imputado.csv: valores preenchidos com parametros aprendidos no treino.
- *_preparado.csv: 8 features prontas para os modelos (codificacao ordinal;
  StandardScaler apenas em atributos com distribuicao normal), id_cliente e churn.
  Codigos: contrato MENSAL=0, ANUAL=1, BIENAL=2; pagamento BOLETO=0, CARTAO=1,
  PIX=2; AUSENTE ou categoria desconhecida = -1.
- preprocessador_template.joblib: pipeline sem ajuste para os experimentos.

## Validacao cruzada

Use treino_antes_imputacao.csv, separando id_cliente e churn dos preditores.
Monte um Pipeline com clone(preprocessador_template) e o modelo, e utilize
StratifiedKFold(n_splits=5, shuffle=True, random_state=42). O pipeline deve
ser ajustado dentro de cada rodada, incluindo imputacao e codificacao.
Ao ler o CSV, informe dtype={"id_cliente": "string"}.
As categorias AUSENTE usadas pelo pipeline representam ausencia registrada.

## Avaliacao final

Apos escolher os parametros por validacao cruzada, ajuste o pipeline completo
no treino antes da imputacao e avalie no teste antes da imputacao.
Os arquivos preparados sao alternativas para esse ajuste final quando forem
utilizados exatamente os mesmos tratamentos e as mesmas features.
O teste fica reservado para a avaliacao final.

O CSV original foi preservado. As datas da base analitica estao em ISO.
