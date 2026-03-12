# Data Contract — Cenário Atual BR

> **Data Contract** é um acordo formal que define a estrutura, significado e regras de qualidade de um conjunto de dados, funcionando como documentação viva que garante que todos os consumidores do dado — analistas, engenheiros, times de logística — entendam exatamente o que cada campo representa e como interpretá-lo.

**Contexto:** Esta aba apresenta o cenário consolidado mensal de oferta e demanda da Ambev para o SKU Long Neck, segmentado por região geográfica comercial (MG, SP, NENO, CO, RJ, SUL e Export). O objetivo é permitir que o time de planejamento de supply chain avalie, para cada mês, se o volume produzido e transferido entre fábricas é suficiente para atender à demanda de cada região sem gerar ruptura. Janeiro possui colunas adicionais de produção (realizado, planejado e programado) por refletir dados parcialmente realizados; Fevereiro traz apenas o planejamento WSNP, pois ainda está no horizonte de projeção. A métrica-chave desta aba é a suficiência em dias: ela traduz o estoque em "quantos dias de venda ele cobre", tornando comparáveis regiões com volumes absolutos muito diferentes.

| Nome da Coluna | Tipo | Descrição |
|---|---|---|
| mes | string | Mês de referência do bloco de dados (ex: "Janeiro", "Fevereiro") |
| dias_uteis | int | Quantidade de dias úteis do mês — usada como denominador no cálculo de demanda linear diária |
| sku | string | Família do SKU analisado (ex: "LONG NECK") |
| geo_reg | string | Região geográfica comercial de destino (MG, SP, NENO, CO, RJ, SUL, Export, TOTAL) |
| demanda | int | Volume previsto de venda na região no mês, em HL (hectolitros) |
| producao_realizada | int | Volume de produção já efetivamente concluído no mês, em HL. Presente apenas em Janeiro — quando o mês está parcialmente transcorrido e já há dado real |
| producao_weekly_supply_network_planning (WSNP) | int | Volume de produção planejado pelo WSNP (Weekly Supply Network Planning — processo semanal que define quanto cada fábrica deve produzir com base na demanda e nos estoques da rede), em HL |
| producao_realizada_projetada | int | Projeção do volume de produção total do mês com base no ritmo já realizado, em HL. Presente apenas em Janeiro |
| producao_1w (1W) | int | Volume programado para produção na primeira semana do mês, em HL. Presente apenas em Janeiro |
| estoque_inicial_mes (EI Mês) | int | Volume de estoque disponível no início do mês na região, em HL. Presente apenas em Janeiro — o estoque inicial de Fevereiro é o estoque final de Janeiro |
| suficiencia_inicial_dias (Suf. ini) | int | Suficiência de estoque no início do mês, expressa em dias de cobertura de demanda. Fórmula: `estoque_inicial_mes / (demanda / dias_uteis)`. Indica quantos dias de venda o estoque cobre sem nenhum reabastecimento |
| transferencia_malha | int | Volume líquido de transferência entre unidades na malha logística, em HL. Positivo = a região recebe produto de outra fábrica; negativo = a região envia produto para outra. "Malha" refere-se à rede integrada de fábricas e centros de distribuição da Ambev |
| estoque_final_mes (EFM) | int | Estoque projetado ao final do mês, em HL. Fórmula: `estoque_inicial_mes + producao + transferencia_malha − demanda` |
| suficiencia_final_dias (Suf. f) | int | Suficiência de estoque ao final do mês, em dias de cobertura. Fórmula: `estoque_final_mes / (demanda / dias_uteis)`. É a métrica principal de decisão: se cair abaixo do limiar operacional, o time aciona transferências adicionais |

### Regras de Qualidade

- A soma de `transferencia_malha` de todas as regiões deve tender a zero — o que uma região recebe, outra necessariamente envia. Desvio significativo indica erro de modelagem ou transferência não registrada.
- A linha TOTAL deve fechar como soma aritmética das linhas regionais para `demanda` (mes anterior), `transferencia_malha` e `estoque_final_mes`. Divergência indica inconsistência no balanceamento do plano.
- `suficiencia_final_dias` deve ser ≥ 12 ao fechamento de cada mês — este é o piso operacional de cobertura da Ambev. Valores abaixo de 12 dias indicam que o estoque não é suficiente para absorver variações de demanda ou atrasos na malha, configurando risco real de ruptura.

---

# Data Contract — Custos de Transferência

**Contexto:** Esta aba reúne os custos unitários de três naturezas distintas: (1) custo logístico de transferir um produto acabado de uma fábrica para outra via malha (rodoviário ou cabotagem), (2) custo de produção terceirizada via MACO (Make and Contract Operations — quando a Ambev contrata uma cervejaria parceira para produzir um SKU que ela não consegue atender internamente) e (3) custo de produção própria nas fábricas do grupo. Esses dados são a base para o solver de otimização decidir se é mais barato produzir localmente, transferir de outra planta ou acionar um MACO. O time de S&OP (Sales & Operations Planning) usa esta tabela para arbitrar decisões de sourcing: quando a suficiência de uma região está baixa, compara-se o custo de transferir versus produzir via MACO versus aumentar produção própria.

| Nome da Coluna | Tipo | Descrição |
|---|---|---|
| tipo_custo | string | Classificação da natureza do custo. Valores: "transferencia" (custo de mover produto entre fábricas), "maco" (custo de produção terceirizada via MACO — Make and Contract Operations), "producao" (custo de produção própria interna) |
| sku | string | Descrição completa do SKU ao qual o custo se aplica (ex: "COLORADO LAGER LN 355ML CX C/12") |
| origem | string | Código e nome da fábrica/planta de origem. Aplicável apenas quando `tipo_custo` = "transferencia" (ex: "BR16 - F. JACAREI - PLANT - SP"). Nulo para os demais tipos |
| destino | string | Código e nome da fábrica/planta de destino. Aplicável apenas quando `tipo_custo` = "transferencia" (ex: "BR04 - F. CAMACARI - PLANT - BA"). Nulo para os demais tipos |
| custo_reais_por_hl | int | Custo unitário em R$/HL. Para transferências, inclui frete e custos logísticos da rota origem→destino. Para MACO, inclui o valor pago ao terceiro. Para produção, inclui custo variável de fabricação |
---

# Data Contract — Produção PCP

**Contexto:** Esta aba detalha o Plano de Controle de Produção (PCP) semanal por linha de envase e fábrica para o mês de fevereiro de 2026. Cada bloco representa uma fábrica e sua linha de envase, listando quais SKUs serão produzidos em cada semana e em qual volume. O título "Crítica PCPs" indica que este é o plano revisado/criticado — ou seja, já passou por validação de capacidade e viabilidade operacional. O time de produção e o time de planejamento usam esta aba para confirmar que a alocação semanal de cada linha respeita a capacidade instalada e atende às necessidades de abastecimento da malha.

| Nome da Coluna | Tipo | Descrição |
|---|---|---|
| localizacao | string | Código e nome da fábrica (ex: "BR03 - F. AQUIRAZ - PLANT - CE"). Identifica a unidade fabril onde a produção será realizada |
| linha | string | Identificador da linha de envase na fábrica (ex: "L541"). Cada fábrica pode ter múltiplas linhas, cada uma com capacidade e tipos de embalagem específicos |
| nominal_garrafas_hora | int | Velocidade nominal da linha de envase, em garrafas por hora. Representa a capacidade teórica máxima de throughput da linha |
| capacidade_semanal_hl | int | Capacidade máxima da linha por semana, em HL. É o teto operacional: nenhum SKU ou combinação de SKUs pode ultrapassar esse volume numa mesma semana |
| item | string | Descrição completa do SKU programado para produção (ex: "MALZBIER BRAHMA LN355ML SIX PAC BAND C/4") |
| tipo_container | string | Tipo de embalagem produzida na linha (ex: "LONG NECK 355ML", "LONG NECK 330ML", "LONG NECK 269ML"). Determina compatibilidade da linha com o SKU |
| semana | date | Data de início da semana de produção (2026-02-02, 2026-02-09, 2026-02-16, 2026-02-23) |
| volume_programado | int | Volume de produção programado para o SKU naquela semana e linha, em HL. Zero indica que a linha não produzirá aquele SKU na semana |

### Regras de Qualidade

- A soma de `volume_programado` de todos os SKUs em uma mesma `linha` + `semana` não pode ultrapassar `capacidade_semanal_hl`. Violação indica sobre-alocação da linha — fisicamente impossível de executar.
- A linha de subtotal por fábrica (última linha de cada bloco) deve igualar a soma dos volumes individuais dos SKUs na mesma semana. Essa linha funciona como check de integridade.
- Se um SKU aparece com `volume_programado` = 0 em todas as 4 semanas de uma fábrica, ele pode estar ali apenas como placeholder de capacidade reservada; isso deve ser validado contra o plano de demanda.
- 
---

# Data Contract — Transferências Programadas

**Contexto:** Esta aba contém a programação semanal de transferências físicas de produto entre regionais da Ambev, com detalhamento de origem, destino, modal logístico e volumes. No contexto observado, toda a movimentação é da Regional Sudeste (REG SE) para a Regional Nordeste (REG NE) via cabotagem (transporte marítimo costeiro — mais barato que rodoviário para longas distâncias, porém com lead time maior). O time de logística utiliza esta aba para confirmar embarques, reservar capacidade nos navios e alinhar janelas de recebimento nas fábricas de destino.

| Nome da Coluna | Tipo | Descrição |
|---|---|---|
| regional_origem | string | Regional de origem que envia o produto (ex: "REG SE" — Regional Sudeste) |
| geo_destino | string | Regional geográfica de destino que receberá o produto (ex: "REG NE" — Regional Nordeste) |
| descricao_destino | string | Nome da unidade ou centro de distribuição de destino (ex: "F. CAMACARI", "F. FONTE MATA", "CDR Bahia") |
| modal | string | Modal logístico utilizado para a transferência. "Cabotagem" = transporte marítimo entre portos brasileiros, tipicamente usado para grandes volumes em rotas longas por ser mais econômico que rodoviário |
| codigo_produto | int | Código numérico identificador do SKU no sistema |
| descricao_produto | string | Descrição completa do SKU transferido (ex: "GOOSE ISLAND MIDWAY NAC LN 355ML CX C/12") |
| semana | date | Data de início da semana de transferência (2026-09-02, 2026-09-09, 2026-09-16, 2026-09-23) |
| volume_programado | int | Volume programado para embarque naquela semana, em HL. Nesta aba os valores são sempre positivos — a direção do fluxo é dada pela combinação origem→destino |

### Regras de Qualidade

- Volumes de cabotagem devem respeitar a capacidade dos navios e as janelas portuárias; variação abrupta entre semanas (ex: 16.707 HL na semana 1 vs 12.600 HL nas seguintes) pode indicar restrição de capacidade no modal.

---

# Data Contract — Cenário Divulgado

**Contexto:** Esta aba contém o cenário oficial de planejamento semanal divulgado para fevereiro de 2026 — é o plano que o time de S&OP aprovou e comunicou às áreas. Estrutura-se em dois grandes recortes geográficos: (1) sub-regiões do Nordeste/Norte (NENO) — Mapapi, NE Norte, NE Sul, NO Araguaia e NO Centro — onde os SKUs são analisados individualmente por sub-região, e (2) a região São Paulo (SP), onde múltiplos SKUs concorrem pelo mesmo estoque regional. Para cada combinação de SKU × sub-região × semana, o plano detalha demanda, produção, movimentações de estoque (transferências internas entre sub-regiões, transferências externas via cabotagem e rodoviário, volume em trânsito) e a suficiência resultante. Este é o cenário-base contra o qual variações de demanda são comparadas.

| Nome da Coluna | Tipo | Descrição |
|---|---|---|
| semana | string | Identificador da semana de planejamento (ex: "W0 - 02/02/2026"). W0 é a semana corrente, W1-W3 são as projeções futuras |
| codigo_sku | int | Código numérico do SKU (ex: 70934 = Patagonia Amber Lager, 65758 = Goose Island Midway, 70792 = Malzbier Brahma, 83179 = Colorado Lager) |
| descricao_sku | string | Descrição completa do SKU (ex: "PATAGONIA AMBER LAGER LN355ML CX12") |
| geo_reg | string | Sub-região geográfica de análise. No recorte NENO: Mapapi, NE Norte, NE Sul, NO Araguaia, NO Centro. No recorte SP: "SP". A linha "TOTAL" consolida o bloco |
| demanda | int | Volume previsto de demanda na semana para aquela sub-região, em HL |
| producao_weekly_supply_network_planning (WSNP) | int | Volume de produção planejado pelo WSNP na semana, em HL. Nas semanas W0 e W1, o cabeçalho original é "WSNP"; nas semanas W2 e W3, aparece como "Malha" (referência à malha de abastecimento) — ambos representam o volume planejado de produção para a região |
| estoque_inicial_semana (EI Semana) | int | Estoque disponível no início da semana, em HL. Presente explicitamente apenas em W0; nas semanas seguintes, é igual ao `estoque_final_semana` da semana anterior |
| suficiencia_inicial_dias (Suf. ini) | int | Suficiência no início da semana, em dias de cobertura. Fórmula: `estoque_inicial_semana / (demanda / 7)`. Presente apenas em W0 |
| transferencia_interna | int | Volume de transferência entre sub-regiões dentro da mesma macro-região, em HL. Positivo = recebe; negativo = envia. Representa redistribuição de estoque dentro do NENO ou dentro de SP |
| transferencia_externa_cabotagem | int | Volume de transferência via cabotagem (transporte marítimo costeiro entre portos), em HL. Positivo = recebe; negativo = envia. Tipicamente vem de fábricas do Sudeste para o Nordeste |
| transferencia_externa_rodoviario | int | Volume de transferência via modal rodoviário, em HL. Positivo = recebe; negativo = envia. Usado para rotas mais curtas ou urgentes onde cabotagem não é viável pelo lead time |
| transito | int | Volume em trânsito — produto já expedido mas ainda não recebido no destino, em HL. Reflete o lead time logístico: a mercadoria está no caminhão ou navio mas não está disponível para venda |
| estoque_final_semana (EF Semana) | int | Estoque projetado ao final da semana, em HL. Fórmula: `estoque_inicial_semana + producao_wsnp + transferencia_interna + transferencia_externa_cabotagem + transferencia_externa_rodoviario + transito − demanda` |
| suficiencia_final_dias (Suf. f) | int | Suficiência ao final da semana, em dias de cobertura de demanda. Fórmula: `estoque_final_semana / (demanda / 7)`. Valores abaixo de 5 dias indicam risco de ruptura; valores negativos confirmam ruptura |

### Regras de Qualidade

- A soma de `transferencia_interna` de todas as sub-regiões dentro de um bloco de SKU deve tender a zero — trata-se de redistribuição interna, o que sai de uma sub-região entra em outra. Desvio significativo indica volume "criado" ou "destruído" no modelo.
- `estoque_inicial_semana` de W(N+1) deve ser igual a `estoque_final_semana` de W(N) para a mesma combinação de SKU × sub-região.
- `estoque_final_semana` negativo sinaliza ruptura confirmada de estoque — a região ficará sem produto para atender a demanda, exigindo intervenção operacional imediata (antecipação de transferência, alocação emergencial ou redução de meta de vendas).
- Sub-regiões como NO Araguaia com `producao_wsnp` = 0 em todas as semanas e `transferencia_interna` ≈ `demanda` são regiões 100% dependentes de abastecimento externo — qualquer mudança na malha gera ruptura direta.
- `suficiencia_final_dias` < 12 é o limiar operacional típico que aciona o alerta de ruptura iminente no processo de S&OP.

---

# Data Contract — Cenário com Nova Demanda

**Contexto:** Esta aba é uma cópia estrutural do "Cenário Divulgado", porém com valores de demanda revisados para cima em determinados SKUs e sub-regiões — especialmente a Malzbier Brahma (70792), que apresenta aumento significativo de demanda. O objetivo é funcionar como cenário de stress test: dado que a produção e as transferências permanecem iguais ao cenário original, o time de planejamento precisa avaliar o impacto da nova demanda sobre estoques e suficiência. Campos de produção e transferência são idênticos ao Cenário Divulgado; apenas `demanda`, `estoque_final_semana` e `suficiencia_final_dias` mudam, permitindo comparação direta entre os dois cenários para apoiar decisões de contingência.

| Nome da Coluna | Tipo | Descrição |
|---|---|---|
| semana | string | Identificador da semana de planejamento (ex: "W0 - 02/02/2026"). W0 é a semana corrente, W1-W3 são as projeções futuras |
| codigo_sku | int | Código numérico do SKU (ex: 70934 = Patagonia Amber Lager, 65758 = Goose Island Midway, 70792 = Malzbier Brahma, 83179 = Colorado Lager) |
| descricao_sku | string | Descrição completa do SKU (ex: "MALZBIER BRAHMA LN355ML SIX PAC BAND C/4") |
| geo_reg | string | Sub-região geográfica de análise. No recorte NENO: Mapapi, NE Norte, NE Sul, NO Araguaia, NO Centro. No recorte SP: "SP". A linha "TOTAL" consolida o bloco |
| demanda | int | Volume revisado de demanda na semana para aquela sub-região, em HL. Este é o campo que difere do Cenário Divulgado — os demais inputs permanecem iguais |
| producao_weekly_supply_network_planning (WSNP) | int | Volume de produção planejado pelo WSNP na semana, em HL. Idêntico ao Cenário Divulgado — a produção não foi replanejada para absorver a nova demanda |
| estoque_inicial_semana (EI Semana) | int | Estoque disponível no início da semana, em HL. Presente explicitamente apenas em W0; nas semanas seguintes, é igual ao `estoque_final_semana` da semana anterior |
| suficiencia_inicial_dias (Suf. ini) | int | Suficiência no início da semana, em dias de cobertura. Fórmula: `estoque_inicial_semana / (demanda / 7)`. Presente apenas em W0. Será menor que no Cenário Divulgado quando a demanda é maior, mesmo com estoque idêntico |
| transferencia_interna | int | Volume de transferência entre sub-regiões dentro da mesma macro-região, em HL. Positivo = recebe; negativo = envia. Idêntico ao Cenário Divulgado |
| transferencia_externa_cabotagem | int | Volume de transferência via cabotagem, em HL. Positivo = recebe; negativo = envia. Idêntico ao Cenário Divulgado |
| transferencia_externa_rodoviario | int | Volume de transferência via modal rodoviário, em HL. Positivo = recebe; negativo = envia. Idêntico ao Cenário Divulgado |
| transito | int | Volume em trânsito — produto já expedido mas ainda não recebido, em HL. Idêntico ao Cenário Divulgado |
| estoque_final_semana (EF Semana) | int | Estoque projetado ao final da semana, em HL. Fórmula: `estoque_inicial_semana + producao_wsnp + transferencia_interna + transferencia_externa_cabotagem + transferencia_externa_rodoviario + transito − demanda`. Será menor que no Cenário Divulgado proporcionalmente ao aumento de demanda |
| suficiencia_final_dias (Suf. f) | int | Suficiência ao final da semana, em dias de cobertura de demanda. Fórmula: `estoque_final_semana / (demanda / 7)`. Sofre duplo impacto: numerador menor (menos estoque) e denominador maior (mais demanda por dia) |

### Regras de Qualidade

- A diferença de `demanda` em relação ao Cenário Divulgado deve ser rastreável e justificada: no caso observado, Malzbier Brahma (70792) apresenta aumento de ~30% em várias sub-regiões enquanto os demais SKUs permanecem iguais.
- `estoque_final_semana` negativo que não existia no Cenário Divulgado mas aparece neste cenário indica novas rupturas criadas exclusivamente pelo aumento de demanda. Isso indica uma ruptura na malha de distribuição
