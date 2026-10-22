# NousSight, versão sequencial: Sprint 4 de Data Science and Statistical Computing

Esta é a nossa entrega do Challenge TOTVS (FIAP) para a Sprint 4. A gente remontou o pipeline de agentes do NousSight do jeito que ele era antes de as análises passarem a rodar em paralelo, e avaliou essa versão como um modelo estatístico, usando a base TOTVS anonimizada.

**Grupo:** Áurea (RM 563837) · Bernardo Hanashiro (RM 565266) · Kauan (RM 566043) · Matheus Vasques (RM 563309) · Thomas Sievers (RM 563566)

## Arquivos

| Arquivo | O que é |
|---|---|
| `NousSight_sequencial_DS_Sprint4.ipynb` | Notebook executado, com todas as saídas |
| `amostra_rotulada.csv` | 100 trechos rotulados (60 de desenvolvimento, 40 de holdout): pré-rótulo do LLM, marcação de revisão humana e rótulo final |
| `sorteio_inicial.csv` | Sorteio aleatório de 100 trechos, com o rótulo de cada um (usado para estimar a proporção real de classes) |
| `dados/ANON_nome_transcricao.csv` | Base TOTVS anonimizada (1.147 reuniões) |
| `requirements.txt` | Versões dos pacotes do ambiente em que o notebook foi executado |

## Base técnica

1. Partimos do commit `e4d6257` (03/10/2026, 14h35). Ele é o último antes do `0a4fd8d` (03/10/2026, 16h24), que passou as análises para `ParallelAgent`.
2. A cadeia que remontamos roda em fila, um agente depois do outro: `transcritor` → `extrator_de_insights` → `analista_de_historico_e_risco` → `estruturador_crm`. Os prompts e o catálogo daquela versão estão dentro do notebook.
3. Ficaram de fora: avaliador de confiança e validação humana, engajamento e receptividade, índice de percepção, desempenho do vendedor, síntese e acessibilidade.

## Como reproduzir

1. Instale o Python 3.13 e os pacotes do `requirements.txt` com `pip install -r requirements.txt`.
2. Abra e rode o notebook **a partir desta pasta**, porque todos os caminhos são relativos a ela. A execução completa leva de 5 a 13 minutos, dependendo da máquina.
3. Se a base estiver guardada em outro lugar, coloque o caminho na variável `BASE_TOTVS_CSV`.
4. O extrator LLM (seção 2.4) usa as credenciais padrão do Google Cloud do ambiente e o projeto da variável `GOOGLE_CLOUD_PROJECT`. Sem elas, a célula continua sem ele. Nenhuma chave fica no notebook.

## Decisões de método

1. **Holdout:** é o conjunto que a gente separou para testar o modelo só no final. São as reuniões de **índice ímpar** na ordem do CSV (573). As de índice par (574) servem para explorar, treinar e calibrar.
2. **Exploração:** tópicos com LDA (k = 6, escolhido pela coerência UMass), em vez de TF-IDF por produto. O LDA agrupa em temas de conversa as palavras que costumam aparecer juntas.
3. **Rotulagem:** pré-rótulo feito com apoio de um assistente de IA, seguindo as definições do extrator antigo, e revisão feita pelo grupo em 15 trechos (11 de 15 iguais; as 4 correções ficaram no lugar do pré-rótulo).
4. **Anonimização extra:** prenomes que sobraram na base (grudados no rótulo de quem fala ou ditos na conversa) viram `[PESSOA]` no notebook e nos CSVs de amostra.

## Resultados principais

O F1 é uma nota de 0 a 1 que junta precisão (quanto do que o modelo marcou estava certo) e recall (quanto do que existia ele achou); o F1 macro é a média dessa nota entre as quatro classes. A AUC é a chance de um cliente detrator ter score de risco maior que um não detrator: 0,5 é igual ao acaso.

| Item | Resultado |
|---|---|
| Validação cruzada, 60 trechos (F1 macro, 5-fold × 20) | NB 0,317 ± 0,10 · SVM 0,332 ± 0,11 · regra de pistas 0,52 (inflada pelo sorteio por pistas) |
| Modelo final | TF-IDF de caracteres 3 a 5 + Naive Bayes, com prior corrigido para a proporção real (68% de trechos sem insight) |
| Holdout, distribuição real (40 trechos aleatórios) | precisão de insight 0,50 · recall 0,33 · F1 macro 0,39 (regra: 0,38 / 0,50 / 0,35) |
| Holdout balanceado (40 trechos) | F1 macro 0,24 (NB sem correção 0,32; SVM 0,27; regra 0,45) |
| Score de risco × NPS (158 reuniões do holdout, 24 detratores) | AUC 0,42 (IC 95% 0,30 a 0,54): sem associação |
| Extrator LLM antigo | O extrator LLM não foi executado nesta entrega; a comparação usa os modelos locais (ver 2.4). O código e o prompt estão no notebook para execução futura. |
