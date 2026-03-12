# Análise Long Neck NENO - Ambev Case

Este repositório reúne uma análise exploratória em notebook sobre oferta, demanda, estoque, capacidade e custos logísticos para SKUs Long Neck da Ambev, com foco na região NENO.

## Conteúdo do Projeto

- `notebooks/Analise_LongNeck_Ambev.ipynb`: notebook principal com visualizações e comparações de cenário.
- `notebooks/Data Contract.md`: dicionário de dados e regras de qualidade das abas utilizadas na análise.

## Objetivo

Avaliar o risco de ruptura e os trade-offs operacionais/logísticos diante de mudanças de demanda, especialmente no SKU Malzbier Brahma, comparando:

- Cenário divulgado
- Cenário com nova demanda (+30%)

## Etapas da Análise (Notebook)

O notebook está organizado em seções, incluindo:

1. Carregamento das abas de dados
2. Evolução histórica de vendas Long Neck (NENO)
3. Demanda mensal projetada (1o semestre/2026)
4. Demanda por região no cenário Brasil
5. Plano de produção PCP
6. Custos de transferência e produção
7. Impacto da nova demanda (Malzbier)
8. Suficiência em dias (DOI)
9. Transferências programadas
10. Utilização de capacidade produtiva
11. Comparações finais de estoque e suficiência por SKU
12. Comparação de modais logísticos

## Principais Insights Apontados

- Crescimento sustentado da demanda de Long Neck no NENO.
- Linhas produtivas operando próximas ao limite em semanas críticas.
- Queda acentuada de suficiência com aumento de demanda para Malzbier.
- Dependência de transferências para alguns SKUs/regiões.
- Trade-off entre custo e velocidade de atendimento (cabotagem vs rodoviário).

## Requisitos

- Python 3.10+
- Jupyter Notebook ou VS Code com extensão Jupyter
- Bibliotecas Python:
	- `pandas`
	- `matplotlib`
	- `seaborn`
	- `numpy`
	- `openpyxl` (leitura de Excel)

## Como Executar

1. Crie e ative um ambiente virtual (opcional, recomendado).
2. Instale as dependências:

```bash
pip install pandas matplotlib seaborn numpy openpyxl jupyter
```

3. Abra o notebook:

```bash
jupyter notebook notebooks/Analise_LongNeck_Ambev.ipynb
```

4. Ajuste o caminho do arquivo Excel na célula de carregamento (`file_path`), pois o caminho atual está local e absoluto.

## Sobre os Dados

O arquivo Excel de origem (com as abas do case) nao está versionado neste repositório. Para reproduzir a análise:

- garanta acesso ao arquivo `.xlsx` utilizado no case;
- atualize o `file_path` no notebook para o caminho do seu ambiente.

Para entendimento de campos, tipos e regras de consistência, consulte `notebooks/Data Contract.md`.

## Estrutura

```text
.
|-- README.md
`-- notebooks/
		|-- Analise_LongNeck_Ambev.ipynb
		`-- Data Contract.md
```

## Observação

Este projeto prioriza análise exploratória e comunicação visual para suporte a decisões de planejamento (S&OP, supply e logística).