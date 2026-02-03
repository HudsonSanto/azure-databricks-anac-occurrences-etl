# ANAC Aviation Safety Pipeline  
**Arquitetura Medallion com PySpark e Azure Databricks**

Processamento completo e moderno dos dados públicos de **ocorrências aeronáuticas** divulgados pela ANAC (Agência Nacional de Aviação Civil), seguindo o padrão **Medallion** (Bronze → Silver → Gold).

<p align="center">
  <img src="https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white" alt="Python" />
  <img src="https://img.shields.io/badge/Apache_Spark-E25A1C?style=for-the-badge&logo=apache-spark&logoColor=white" alt="Spark" />
  <img src="https://img.shields.io/badge/Databricks-FF3621?style=for-the-badge&logo=databricks&logoColor=white" alt="Databricks" />
  <img src="https://img.shields.io/badge/Azure-0078D4?style=for-the-badge&logo=microsoft-azure&logoColor=white" alt="Azure" />
</p>

## Sobre o Projeto

Este repositório implementa um pipeline de dados end-to-end para tratar os arquivos JSON de ocorrências aeronáuticas da ANAC, utilizando:

- **Azure Data Lake Storage Gen2** como camada de armazenamento
- **Databricks** como ambiente de execução
- **PySpark** para todo o processamento distribuído
- Arquitetura **Medallion** (padrão da indústria para Data Lakes modernos)

Objetivo: transformar dados brutos em uma camada analítica limpa, confiável e otimizada (Gold), pronta para relatórios, dashboards e análises avançadas.

## Arquitetura Medallion

JSON bruto (ANAC)
↓
Bronze → armazenamento cru
↓
Silver → limpeza, padronização, tratamento de nulos, conversão de tipos
↓
Gold   → modelo analítico, colunas renomeadas, métrica Total_Lesoes, particionamento por Estado, coluna de auditoria
↓
Consumo → Databricks SQL, Power BI, Synapse, etc.



## Estrutura do Repositório

```text
ANAC-Aviation-Safety-Pipeline/
├── notebooks/
│   ├── 2. Acesso Data Lake Storage Gen2.ipynb           # Montagem ADLS Gen2 com Service Principal
│   ├── 4. Anac - Camada Silver (Refaturado).ipynb       # Limpeza e padronização
│   ├── 5. Anac - Camada Gold (Refaturado).ipynb         # Transformações analíticas + particionamento
│   └── (versões anteriores para histórico)
├── README.md
└── .gitignore
```



## Principais Etapas do Pipeline

| Etapa              | Notebook principal                            | Principais operações realizadas                                                                                   |
|--------------------|-----------------------------------------------|--------------------------------------------------------------------------------------------------------------------|
| 1. Acesso ao ADLS  | 2. Acesso Data Lake Storage Gen2.ipynb        | Montagem do Azure Data Lake Gen2 via OAuth 2.0 (Service Principal) no DBFS                                       |
| 2. Camada Silver   | 4. Anac - Camada Silver (Refaturado).ipynb    | • Leitura JSON<br>• Tratamento sistemático de nulos (strings → "Sem Registro", inteiros → 0)<br>• Cast de tipos<br>• Salvamento em Parquet |
| 3. Camada Gold     | 5. Anac - Camada Gold (Refaturado).ipynb      | • Seleção de colunas de negócio<br>• Criação de `Total_Lesoes` (soma de todas lesões)<br>• Renomeação amigável<br>• Filtro de estados inválidos<br>• Coluna `Atualizacao` (horário BR)<br>• Particionamento por `Estado` |


## Como Executar

1)Configure um Databricks Workspace com acesso ao Azure Data Lake Storage Gen2

2)Crie um Service Principal e adicione as credenciais como Secrets no Databricks

3)Monte o storage seguindo o notebook 2. Acesso Data Lake Storage Gen2.ipynb

4) Execute os notebooks na ordem:

4. Camada Silver

5. Camada Gold


5) Os dados finais ficarão disponíveis em:

```text
text/mnt/Anac/Gold/anac_gold_particionado/
    ├── Estado=SP/
    ├── Estado=RJ/
    └── ...
```

## Resultados da Camada Gold

```text
Tabela particionada por Estado (otimiza consultas regionais)
Coluna calculada: Total_Lesoes
Coluna de governança: Atualizacao (yyyy-MM-dd HH:mm:ss – horário de Brasília)
Nomes de colunas intuitivos para analistas de negócio
```
