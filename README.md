# Antecipação de Alertas Críticos em Frotas de Mineração

Pipeline de ponta a ponta (ETL + Machine Learning) para prever alertas críticos
("don't go") em equipamentos de mineração, com até 4 horas de antecedência —
construído em Databricks com arquitetura Medalhão (Bronze / Silver / Gold).

**[📄 Relatório completo (PDF)](Analise-de-Dados-Avancado-Vale-Barbara-Tavares1 - Copiar.pdf)**

---

## O problema

Frotas de mineração geram telemetria contínua de cada equipamento. Um alerta
"don't go" indica que a máquina não deve operar — com impacto direto em
segurança, disponibilidade de frota e produção. A pergunta que guiou este
projeto: **é possível estimar o risco de um equipamento gerar esse alerta nas
próximas horas, com utilidade prática acima de uma regra simples?**

## O achado mais importante

Ao investigar a variável alvo, descobri que **70,5% dos registros brutos de
alerta eram, na verdade, o mesmo alarme "chacoalhando"** — um sensor de
temperatura de freio oscilando entre ativo/inativo dezenas de vezes por dia
(chattering), não risco novo a cada disparo. Sem tratar isso, o modelo teria
aprendido a prever ruído de instrumentação, e o equipamento apontado como
"mais crítico" da frota estava isolado no topo do ranking só por esse motivo.

Apliquei uma técnica de debounce para consolidar esses disparos em episódios
reais de risco antes de treinar qualquer modelo — reduzindo 19.962 eventos
brutos para 5.890 episódios genuínos.

## Resultados

| Modelo | Precision | Recall | F1 |
|---|---|---|---|
| Baseline (heurística de frequência) | 0,158 | 0,554 | 0,245 |
| Random Forest (limiar padrão) | 0,227 | 0,494 | 0,311 |
| **Random Forest (limiar ajustado)** | 0,149 | **0,700** | 0,245 |
| Isolation Forest (sinal precursor) | — | 0,079 | — |

O ponto de operação foi escolhido priorizando Recall — perder um alerta real
custa uma parada não planejada, um custo maior que investigar um alarme falso.

## Outros achados de destaque

- **Desigualdade entre frotas**: a frota com menor volume histórico de dados
  teve 100% de falso negativo na validação, contra 20% na frota com mais
  exemplos — evidência de que desbalanceamento de dado pesou tanto quanto a
  escolha do algoritmo.
- **Outlier de ingestão identificado e tratado**: um equipamento gerou 1,3
  milhão de eventos de telemetria em um único dia — investigado, isolado via
  winsorização e documentado, sem contaminar o rótulo.
- **Hipótese testada e descartada com transparência**: um cruzamento entre a
  telemetria e o catálogo de regras de negócio foi testado e rejeitado (as
  duas fontes usam escalas de criticidade diferentes) — reportado como
  decisão metodológica, não escondido.

## Stack técnico

- **Databricks** — arquitetura Medalhão (Bronze / Silver / Gold)
- **PySpark** — ETL e engenharia de features em escala (37M+ registros de telemetria)
- **scikit-learn** — Random Forest, Isolation Forest
- **SHAP** — interpretabilidade do modelo

## Estrutura do repositório

```
notebooks/     → notebooks Silver, EDA e Gold (com outputs)
relatorio/     → relatório final em PDF, com as 6 seções do desafio
imagens/       → figuras usadas no relatório e no material de divulgação
```

---

*Projeto desenvolvido para o Desafio de Análise Avançada de Dados —
Programa Desenvolver, Edição 2026.*
