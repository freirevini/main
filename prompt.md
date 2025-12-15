# 🔄 KNIME TO PYTHON CONVERTER - ARQUITETURA COMPLETA

## Documentação Técnica do Sistema de Conversão Híbrido (Python + IA)

**Versão:** 2.0  
**Data:** 2025-12-15  
**Autor:** Engenharia de Dados e Riscos - Setor Bancário  
**Contexto:** Migração de workflows KNIME para Python com foco em auditoria e compliance

---

## 📋 ÍNDICE

1. [Visão Geral](#1-visão-geral)
2. [Arquitetura do Sistema](#2-arquitetura-do-sistema)
3. [Estrutura de Arquivos do Projeto](#3-estrutura-de-arquivos-do-projeto)
4. [Schemas dos Arquivos JSON](#4-schemas-dos-arquivos-json)
5. [Fases de Implementação](#5-fases-de-implementação)
6. [Detalhamento das Etapas](#6-detalhamento-das-etapas)
7. [Ciclo de Aprendizado](#7-ciclo-de-aprendizado)
8. [Métricas e Critérios de Sucesso](#8-métricas-e-critérios-de-sucesso)
9. [Cronograma de Implementação](#9-cronograma-de-implementação)
10. [Glossário](#10-glossário)

---

## 1. VISÃO GERAL

### 1.1 Objetivo

Desenvolver um sistema **híbrido** de conversão de workflows KNIME para código Python executável, combinando:

- **Python Puro:** Para extração de estrutura, parsing de XMLs e construção do grafo de execução
- **Agentes de IA (Google Vertex AI):** Para interpretação de lógica de negócios complexa (Rule Engine, Java Snippet, SQL dinâmico)

### 1.2 Problema que Resolve

| Problema Atual | Solução Proposta |
|----------------|------------------|
| Código legado baseado em Regex (frágil) | Parser XML robusto com namespace |
| Dependência de software proprietário (KNIME) | Código Python autônomo e auditável |
| Conversão manual demorada | Automação com mapeamento determinístico |
| Sem rastreabilidade | Docstrings e comentários automáticos |
| Erros não detectados | Validação cruzada com spec.xml |

### 1.3 Contexto Bancário

Este conversor atende requisitos de:
- ✅ **Compliance regulatório** (BCB, BACEN)
- ✅ **Auditoria interna/externa** (rastreabilidade completa)
- ✅ **Governança de dados** (documentação automática)
- ✅ **Eficiência operacional** (redução de intervenção manual)

---

## 2. ARQUITETURA DO SISTEMA

### 2.1 Diagrama de Fluxo Principal

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         ENTRADA: arquivo .knwf                              │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│  FASE 1: EXTRAÇÃO (Python Puro)                                             │
│  ├── 1.1 Descompactar ZIP                                                   │
│  ├── 1.2 Parsear workflow.knime (nodes + conexões)                          │
│  ├── 1.3 Parsear settings.xml (configurações de cada node)                  │
│  ├── 1.4 Parsear spec.xml (schema de saída de cada node)                    │
│  └── 1.5 Expandir metanodes recursivamente                                  │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│  FASE 2: REPRESENTAÇÃO INTERMEDIÁRIA (RI)                                   │
│  ├── 2.1 Construir grafo direcionado completo                               │
│  ├── 2.2 Calcular ordem topológica                                          │
│  ├── 2.3 Anotar cada node com: inputs, outputs, schema                      │
│  ├── 2.4 Identificar estruturas especiais (loops, branches)                 │
│  └── 2.5 Exportar workflow_ir.json                                          │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│  FASE 3: CLASSIFICAÇÃO E ROTEAMENTO                                         │
│  ├── 3.1 Carregar node_mapping.json (mapeamentos oficiais)                  │
│  ├── 3.2 Classificar cada node: MAPPED | CANDIDATE | UNKNOWN                │
│  └── 3.3 Gerar relatório de cobertura                                       │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                    ┌─────────────────┴─────────────────┐
                    ▼                                   ▼
┌───────────────────────────────────┐     ┌───────────────────────────────────────┐
│  FASE 4A: TRADUÇÃO                │     │  FASE 4B: INTERPRETAÇÃO IA            │
│  DETERMINÍSTICA                   │     │  (apenas para UNKNOWN)                │
│  ├── Aplicar templates            │     │  ├── 4B.1 Montar prompt estruturado   │
│  └── 100% precisão                │     │  ├── 4B.2 Chamar Vertex AI            │
└───────────────────────────────────┘     │  ├── 4B.3 Validar sintaxe             │
                    │                     │  └── 4B.4 Marcar como CANDIDATE        │
                    │                     └───────────────────────────────────────┘
                    │                                   │
                    └─────────────────┬─────────────────┘
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│  FASE 5: GERAÇÃO DE CÓDIGO PYTHON                                           │
│  ├── 5.1 Gerar imports                                                      │
│  ├── 5.2 Gerar funções por node (ou inline)                                 │
│  ├── 5.3 Gerar main() com execução na ordem topológica                      │
│  └── 5.4 Gerar docstrings e comentários de rastreabilidade                  │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│  FASE 6: VALIDAÇÃO CRUZADA                                                  │
│  ├── 6.1 Comparar schema gerado vs spec.xml                                 │
│  ├── 6.2 Gerar relatório de divergências                                    │
│  └── 6.3 Classificar resultado: PASS | FAIL                                 │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                    ┌─────────────────┴─────────────────┐
                    ▼                                   ▼
            ┌───────────────┐                   ┌───────────────┐
            │     PASS      │                   │     FAIL      │
            └───────────────┘                   └───────────────┘
                    │                                   │
                    ▼                                   ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│  FASE 7: CICLO DE APRENDIZADO                                               │
│  ├── 7.1 Se PASS: promover CANDIDATE → OFFICIAL                             │
│  ├── 7.2 Se FAIL: mover para REJECTED com log de erro                       │
│  ├── 7.3 Atualizar node_mapping.json                                        │
│  └── 7.4 Gerar métricas de evolução                                         │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                    SAÍDA: código Python + relatórios                        │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 2.2 Princípios de Design

| Princípio | Descrição | Benefício |
|-----------|-----------|-----------|
| **Separação de Responsabilidades** | Extração (Python) vs Interpretação (IA) | Manutenção simplificada |
| **Determinismo Primeiro** | Templates para nodes conhecidos | Economia de tokens, 100% precisão |
| **Validação Contínua** | Comparação com spec.xml | Detecção precoce de erros |
| **Aprendizado Incremental** | Candidatos → Oficiais | Sistema evolui sozinho |
| **Rastreabilidade Total** | Origem KNIME em cada linha | Auditoria facilitada |

---

## 3. ESTRUTURA DE ARQUIVOS DO PROJETO

### 3.1 Árvore de Diretórios

```
knime_converter/
│
├── config/                          # Arquivos de configuração
│   ├── node_mapping.json            # Mapeamentos OFICIAIS (determinísticos)
│   ├── candidates.json              # Mapeamentos em validação
│   ├── rejected.json                # Mapeamentos que falharam (para análise)
│   └── knime_types.json             # Mapeamento de tipos KNIME → Python
│
├── output/                          # Saídas geradas
│   ├── workflow_ir.json             # Representação Intermediária
│   ├── generated_code.py            # Código Python gerado
│   ├── validation_report.json       # Relatório de validação
│   └── coverage_report.json         # Relatório de cobertura de nodes
│
├── src/                             # Código-fonte
│   │
│   ├── extractors/                  # FASE 1: Extração
│   │   ├── __init__.py
│   │   ├── zip_extractor.py         # Descompacta .knwf
│   │   ├── workflow_parser.py       # Parseia workflow.knime
│   │   ├── settings_parser.py       # Parseia settings.xml
│   │   └── spec_parser.py           # Parseia spec.xml
│   │
│   ├── ir/                          # FASE 2: Representação Intermediária
│   │   ├── __init__.py
│   │   ├── graph_builder.py         # Constrói grafo NetworkX
│   │   ├── topological_sort.py      # Ordena nodes
│   │   └── ir_exporter.py           # Exporta workflow_ir.json
│   │
│   ├── mapping/                     # FASE 3 + 4A: Classificação + Tradução
│   │   ├── __init__.py
│   │   ├── classifier.py            # Classifica nodes
│   │   └── deterministic_translator.py  # Aplica templates
│   │
│   ├── ai/                          # FASE 4B: Interpretação IA
│   │   ├── __init__.py
│   │   ├── prompt_builder.py        # Monta prompts estruturados
│   │   ├── vertex_client.py         # Cliente Google Vertex AI
│   │   └── response_validator.py    # Valida respostas da IA
│   │
│   ├── codegen/                     # FASE 5: Geração de Código
│   │   ├── __init__.py
│   │   ├── python_generator.py      # Gera código Python
│   │   └── docstring_generator.py   # Gera documentação
│   │
│   ├── validation/                  # FASE 6: Validação
│   │   ├── __init__.py
│   │   ├── schema_comparator.py     # Compara schemas
│   │   └── report_generator.py      # Gera relatórios
│   │
│   └── learning/                    # FASE 7: Aprendizado
│       ├── __init__.py
│       ├── promoter.py              # Promove candidatos
│       └── metrics_tracker.py       # Rastreia métricas
│
├── tests/                           # Testes unitários
│   ├── test_extractors.py
│   ├── test_ir.py
│   ├── test_mapping.py
│   └── test_validation.py
│
├── main.py                          # Orquestrador principal
├── cli.py                           # Interface de linha de comando
├── requirements.txt                 # Dependências Python
└── README.md                        # Documentação inicial
```

### 3.2 Descrição dos Módulos

#### 3.2.1 Módulo `extractors/`

| Arquivo | Responsabilidade | Input | Output |
|---------|------------------|-------|--------|
| `zip_extractor.py` | Descompactar .knwf | arquivo .knwf | pasta temporária |
| `workflow_parser.py` | Extrair nodes e conexões | workflow.knime | dict{nodes, edges} |
| `settings_parser.py` | Extrair configurações | settings.xml | dict{factory, model} |
| `spec_parser.py` | Extrair schemas | spec.xml | dict{columns, types} |

#### 3.2.2 Módulo `ir/`

| Arquivo | Responsabilidade | Input | Output |
|---------|------------------|-------|--------|
| `graph_builder.py` | Construir grafo direcionado | nodes + edges | DiGraph |
| `topological_sort.py` | Calcular ordem de execução | DiGraph | lista ordenada |
| `ir_exporter.py` | Serializar para JSON | grafo anotado | workflow_ir.json |

#### 3.2.3 Módulo `mapping/`

| Arquivo | Responsabilidade | Input | Output |
|---------|------------------|-------|--------|
| `classifier.py` | Classificar nodes | IR + mappings | classificação |
| `deterministic_translator.py` | Aplicar templates | template + params | código Python |

#### 3.2.4 Módulo `ai/`

| Arquivo | Responsabilidade | Input | Output |
|---------|------------------|-------|--------|
| `prompt_builder.py` | Construir prompt | node + context | string prompt |
| `vertex_client.py` | Chamar API Vertex AI | prompt | resposta |
| `response_validator.py` | Validar código gerado | código | bool válido |

#### 3.2.5 Módulo `codegen/`

| Arquivo | Responsabilidade | Input | Output |
|---------|------------------|-------|--------|
| `python_generator.py` | Gerar código completo | IR + traduções | arquivo .py |
| `docstring_generator.py` | Gerar documentação | metadados | docstrings |

#### 3.2.6 Módulo `validation/`

| Arquivo | Responsabilidade | Input | Output |
|---------|------------------|-------|--------|
| `schema_comparator.py` | Comparar schemas | gerado vs esperado | divergências |
| `report_generator.py` | Gerar relatórios | dados validação | JSON report |

#### 3.2.7 Módulo `learning/`

| Arquivo | Responsabilidade | Input | Output |
|---------|------------------|-------|--------|
| `promoter.py` | Promover candidatos | candidatos validados | node_mapping atualizado |
| `metrics_tracker.py` | Rastrear evolução | histórico | métricas |

---

## 4. SCHEMAS DOS ARQUIVOS JSON

### 4.1 workflow_ir.json (Representação Intermediária)

Este arquivo é o **coração do sistema** - contém toda a informação extraída do workflow KNIME em formato estruturado.

```json
{
  "metadata": {
    "source_file": "workflow.knwf",
    "knime_version": "4.5.2",
    "extracted_at": "2025-12-15T10:30:00Z",
    "author": "andre.carlucci",
    "last_editor": "vinicius.silva",
    "description": "Workflow de validação CET",
    "credentials_used": ["PTASYBCTR"]
  },
  
  "graph": {
    "nodes": {
      "1": {
        "id": "1",
        "name": "CSV Reader",
        "factory": "org.knime.base.node.io.csvreader.CSVReaderNodeFactory",
        "display_name": "CSV Reader (#1)",
        "annotation": "Lê arquivo de contratos",
        "is_metanode": false,
        "is_loop_start": false,
        "is_loop_end": false,
        "position": {"x": 100, "y": 200},
        "state": "EXECUTED",
        
        "settings": {
          "file_path": "/data/input.csv",
          "delimiter": ",",
          "has_header": true,
          "encoding": "UTF-8"
        },
        
        "input_ports": [],
        
        "output_ports": [
          {
            "index": 0,
            "type": "data",
            "schema": {
              "columns": [
                {
                  "name": "NuContrato",
                  "knime_type": "org.knime.core.data.def.DoubleCell",
                  "python_type": "Float64",
                  "domain": {"min": 1.2014E13, "max": 1.2367E13}
                },
                {
                  "name": "NmProduto",
                  "knime_type": "org.knime.core.data.def.StringCell",
                  "python_type": "str",
                  "domain": {"possible_values": ["Veículos"]}
                },
                {
                  "name": "DtLiberacao",
                  "knime_type": "org.knime.core.data.time.localdate.LocalDateCell",
                  "python_type": "datetime64[ns]",
                  "domain": {}
                }
              ],
              "row_count": 48,
              "column_count": 27
            }
          }
        ],
        
        "classification": "MAPPED",
        "python_code": "df_node_1 = pd.read_csv('/data/input.csv', sep=',')"
      },
      
      "2": {
        "id": "2",
        "name": "Math Formula",
        "factory": "org.knime.ext.jep.JEPNodeFactory",
        "display_name": "Math Formula (#2)",
        "annotation": "Calculo preliminar da CET",
        "is_metanode": false,
        
        "settings": {
          "expression": "$VrPrestacaoContrato$ / pow((1 + ($TxCetAnualContrato$/100)),($dj-d0$/365))",
          "output_column": "Resultado",
          "append_column": false
        },
        
        "input_ports": [
          {
            "index": 0,
            "type": "data",
            "source_node": "1",
            "source_port": 0
          }
        ],
        
        "output_ports": [
          {
            "index": 0,
            "type": "data",
            "schema": {
              "columns": ["...herda de entrada + Resultado"],
              "row_count": 48
            }
          }
        ],
        
        "classification": "CANDIDATE",
        "python_code": "df_node_2['Resultado'] = df_node_1['VrPrestacaoContrato'] / np.power((1 + (df_node_1['TxCetAnualContrato']/100)), (df_node_1['dj-d0']/365))"
      }
    },
    
    "edges": [
      {
        "id": "e1",
        "source_node": "1",
        "source_port": 0,
        "target_node": "2",
        "target_port": 0,
        "type": "data"
      },
      {
        "id": "e2",
        "source_node": "2",
        "source_port": 0,
        "target_node": "3",
        "target_port": 0,
        "type": "data"
      }
    ],
    
    "execution_order": ["1", "2", "3", "4", "5"],
    
    "loops": [
      {
        "id": "loop_1",
        "start_node": "5",
        "end_node": "8",
        "internal_nodes": ["6", "7"],
        "loop_type": "GroupLoop",
        "loop_variables": [
          {"name": "currentIteration", "type": "INTEGER"},
          {"name": "NuContrato", "type": "DOUBLE"},
          {"name": "groupIdentifier", "type": "STRING"}
        ]
      }
    ],
    
    "metanodes": {
      "10": {
        "name": "CRIA DATA DE REFERÊNCIA",
        "folder": "CRIA DATA DE REFERÊNCIA (#10)",
        "internal_workflow": {
          "metadata": {},
          "graph": {
            "nodes": {},
            "edges": [],
            "execution_order": []
          }
        },
        "exposed_variables": [
          {"name": "DtInicialMesAnt", "type": "STRING"},
          {"name": "DtFinalMesAnt", "type": "STRING"},
          {"name": "DtReferencia", "type": "STRING"}
        ]
      }
    },
    
    "flow_variables": [
      {"name": "knime.workspace", "type": "STRING", "scope": "global"}
    ]
  },
  
  "statistics": {
    "total_nodes": 25,
    "total_edges": 24,
    "total_metanodes": 3,
    "total_loops": 1,
    "nodes_by_classification": {
      "MAPPED": 18,
      "CANDIDATE": 5,
      "UNKNOWN": 2
    }
  }
}
```

### 4.2 node_mapping.json (Mapeamentos Oficiais)

Este arquivo contém os **templates determinísticos** para nodes conhecidos. É o principal mecanismo de economia de tokens de IA.

```json
{
  "version": "1.0.0",
  "last_updated": "2025-12-15",
  
  "statistics": {
    "total_mappings": 45,
    "by_category": {
      "io": 8,
      "manipulation": 15,
      "transformation": 12,
      "flow_control": 10
    },
    "by_status": {
      "OFFICIAL": 40,
      "CANDIDATE": 5
    }
  },
  
  "type_mappings": {
    "org.knime.core.data.def.DoubleCell": "Float64",
    "org.knime.core.data.def.IntCell": "Int64",
    "org.knime.core.data.def.LongCell": "Int64",
    "org.knime.core.data.def.StringCell": "str",
    "org.knime.core.data.def.BooleanCell": "bool",
    "org.knime.core.data.time.localdate.LocalDateCell": "datetime64[ns]",
    "org.knime.core.data.date.DateAndTimeCell": "datetime64[ns]"
  },
  
  "mappings": [
    {
      "factory": "org.knime.base.node.io.csvreader.CSVReaderNodeFactory",
      "name": "CSV Reader",
      "category": "io",
      "status": "OFFICIAL",
      "confidence": 1.0,
      "complexity": "simple",
      
      "settings_extraction": {
        "file_path": {
          "path": "model/url",
          "type": "string",
          "required": true
        },
        "delimiter": {
          "path": "model/colDelimiter",
          "type": "string",
          "default": ","
        },
        "has_header": {
          "path": "model/hasColHeader",
          "type": "boolean",
          "default": true
        },
        "encoding": {
          "path": "model/charSet",
          "type": "string",
          "default": "UTF-8"
        }
      },
      
      "python_template": "df_{output_var} = pd.read_csv('{file_path}', sep='{delimiter}', header={header_param}, encoding='{encoding}')",
      
      "template_variables": {
        "header_param": {
          "condition": "has_header",
          "true_value": "0",
          "false_value": "None"
        }
      },
      
      "imports": ["pandas as pd"],
      
      "output_behavior": {
        "type": "creates_dataframe",
        "schema_source": "runtime"
      },
      
      "validation_count": 127,
      "last_validated": "2025-12-14",
      "example_workflows": ["workflow_contratos.knwf", "workflow_propostas.knwf"]
    },
    
    {
      "factory": "org.knime.base.node.preproc.filter.column.DataColumnSpecFilterNodeFactory",
      "name": "Column Filter",
      "category": "manipulation",
      "status": "OFFICIAL",
      "confidence": 1.0,
      "complexity": "simple",
      
      "settings_extraction": {
        "included_columns": {
          "path": "model/column-filter/included_names",
          "type": "array",
          "required": true
        },
        "filter_mode": {
          "path": "model/column-filter/filter-type",
          "type": "string",
          "default": "STANDARD"
        }
      },
      
      "python_template": "df_{output_var} = df_{input_var}[{included_columns}]",
      
      "imports": [],
      
      "output_behavior": {
        "type": "passthrough_filtered",
        "schema_transformation": "keep_only_included"
      },
      
      "validation_count": 89,
      "last_validated": "2025-12-14"
    },
    
    {
      "factory": "org.knime.base.node.preproc.joiner.Joiner2NodeFactory",
      "name": "Joiner",
      "category": "manipulation",
      "status": "OFFICIAL",
      "confidence": 1.0,
      "complexity": "medium",
      
      "settings_extraction": {
        "left_key": {
          "path": "model/leftTableJoinPredicate/0/leftColumn",
          "type": "string",
          "required": true
        },
        "right_key": {
          "path": "model/rightTableJoinPredicate/0/rightColumn",
          "type": "string",
          "required": true
        },
        "join_mode": {
          "path": "model/joinMode",
          "type": "string",
          "required": true
        },
        "duplicate_handling": {
          "path": "model/duplicateHandling",
          "type": "string",
          "default": "AppendSuffix"
        }
      },
      
      "python_template": "df_{output_var} = df_{input_var_0}.merge(df_{input_var_1}, left_on='{left_key}', right_on='{right_key}', how='{join_type}', suffixes=('', '_right'))",
      
      "value_mappings": {
        "join_type": {
          "InnerJoin": "inner",
          "LeftOuterJoin": "left",
          "RightOuterJoin": "right",
          "FullOuterJoin": "outer"
        }
      },
      
      "imports": [],
      
      "output_behavior": {
        "type": "merge",
        "schema_transformation": "combine_columns"
      },
      
      "validation_count": 56,
      "last_validated": "2025-12-13"
    },
    
    {
      "factory": "org.knime.base.node.rules.engine.RuleEngineNodeFactory",
      "name": "Rule Engine",
      "category": "logic",
      "status": "OFFICIAL",
      "confidence": 0.85,
      "complexity": "high",
      "requires_ai_for_rules": true,
      
      "settings_extraction": {
        "rules": {
          "path": "model/rules",
          "type": "rule_array",
          "required": true
        },
        "output_column": {
          "path": "model/new-column-name",
          "type": "string",
          "required": true
        },
        "append_column": {
          "path": "model/append-column",
          "type": "boolean",
          "default": true
        }
      },
      
      "python_template": "# Rule Engine - Requires AI interpretation\n# Original rules:\n{rules_comment}\n{ai_generated_code}",
      
      "ai_prompt_template": "Converta as seguintes regras KNIME Rule Engine para pandas usando np.select:\n\nRegras KNIME:\n{rules}\n\nColunas disponíveis: {columns}\n\nGere código Python que:\n1. Use np.select para múltiplas condições\n2. Crie a coluna '{output_column}'\n3. Preserve a ordem das regras (primeira match vence)",
      
      "imports": ["numpy as np"],
      
      "validation_count": 34,
      "last_validated": "2025-12-12"
    },
    
    {
      "factory": "org.knime.ext.jep.JEPNodeFactory",
      "name": "Math Formula",
      "category": "transformation",
      "status": "OFFICIAL",
      "confidence": 0.90,
      "complexity": "medium",
      "requires_ai_for_complex": true,
      
      "settings_extraction": {
        "expression": {
          "path": "model/expression",
          "type": "string",
          "required": true
        },
        "output_column": {
          "path": "model/replaced_column",
          "type": "string",
          "required": true
        },
        "append_column": {
          "path": "model/append_column",
          "type": "boolean",
          "default": false
        }
      },
      
      "expression_mappings": {
        "pow": "np.power",
        "sqrt": "np.sqrt",
        "abs": "np.abs",
        "exp": "np.exp",
        "log": "np.log",
        "log10": "np.log10",
        "sin": "np.sin",
        "cos": "np.cos",
        "tan": "np.tan",
        "round": "np.round",
        "floor": "np.floor",
        "ceil": "np.ceil"
      },
      
      "column_reference_pattern": "\\$([^$]+)\\$",
      "column_replacement": "df['{column_name}']",
      
      "python_template": "df_{output_var}['{output_column}'] = {converted_expression}",
      
      "imports": ["numpy as np"],
      
      "validation_count": 45,
      "last_validated": "2025-12-14"
    },
    
    {
      "factory": "org.knime.base.node.preproc.filter.row.RowFilterNodeFactory",
      "name": "Row Filter",
      "category": "manipulation",
      "status": "OFFICIAL",
      "confidence": 1.0,
      "complexity": "simple",
      
      "settings_extraction": {
        "filter_column": {
          "path": "model/FilterCriterion/column/column_name",
          "type": "string"
        },
        "filter_type": {
          "path": "model/FilterCriterion/type",
          "type": "string"
        },
        "filter_value": {
          "path": "model/FilterCriterion/value",
          "type": "dynamic"
        }
      },
      
      "filter_type_templates": {
        "StringEquals": "df_{output_var} = df_{input_var}[df_{input_var}['{filter_column}'] == '{filter_value}']",
        "StringContains": "df_{output_var} = df_{input_var}[df_{input_var}['{filter_column}'].str.contains('{filter_value}', na=False)]",
        "NumberGreater": "df_{output_var} = df_{input_var}[df_{input_var}['{filter_column}'] > {filter_value}]",
        "NumberLess": "df_{output_var} = df_{input_var}[df_{input_var}['{filter_column}'] < {filter_value}]",
        "NumberBetween": "df_{output_var} = df_{input_var}[(df_{input_var}['{filter_column}'] >= {min_value}) & (df_{input_var}['{filter_column}'] <= {max_value})]",
        "MissingValue": "df_{output_var} = df_{input_var}[df_{input_var}['{filter_column}'].isna()]",
        "NotMissingValue": "df_{output_var} = df_{input_var}[df_{input_var}['{filter_column}'].notna()]"
      },
      
      "imports": [],
      
      "validation_count": 78,
      "last_validated": "2025-12-14"
    },
    
    {
      "factory": "org.knime.base.node.preproc.rename.RenameNodeFactory",
      "name": "Column Rename",
      "category": "manipulation",
      "status": "OFFICIAL",
      "confidence": 1.0,
      "complexity": "simple",
      
      "settings_extraction": {
        "rename_pairs": {
          "path": "model/columns",
          "type": "rename_map",
          "extraction_logic": "iterate config/*/new_name where old_name != new_name"
        }
      },
      
      "python_template": "df_{output_var} = df_{input_var}.rename(columns={rename_dict})",
      
      "imports": [],
      
      "validation_count": 67,
      "last_validated": "2025-12-14"
    },
    
    {
      "factory": "org.knime.base.node.preproc.sorter.SorterNodeFactory",
      "name": "Sorter",
      "category": "manipulation",
      "status": "OFFICIAL",
      "confidence": 1.0,
      "complexity": "simple",
      
      "settings_extraction": {
        "sort_columns": {
          "path": "model/sortOrder/*/column_name",
          "type": "array"
        },
        "ascending": {
          "path": "model/sortOrder/*/ascending",
          "type": "boolean_array"
        }
      },
      
      "python_template": "df_{output_var} = df_{input_var}.sort_values(by={sort_columns}, ascending={ascending_list})",
      
      "imports": [],
      
      "validation_count": 54,
      "last_validated": "2025-12-13"
    }
  ]
}
```

### 4.3 candidates.json (Candidatos em Validação)

Armazena mapeamentos gerados pela IA que ainda estão sendo validados.

```json
{
  "schema_version": "1.0",
  "last_updated": "2025-12-15T10:30:00Z",
  
  "candidates": [
    {
      "id": "cand_001",
      "factory": "org.knime.ext.jep.JEPNodeFactory",
      "name": "Math Formula",
      "pattern_hash": "a1b2c3d4e5f6",
      "status": "VALIDATING",
      
      "generated_by": {
        "model": "vertex-ai-gemini-1.5-pro",
        "timestamp": "2025-12-15T10:00:00Z",
        "prompt_version": "1.2"
      },
      
      "source_context": {
        "workflow": "workflow_cet_validation.knwf",
        "node_id": "15",
        "node_display_name": "Math Formula (#15)"
      },
      
      "settings_snapshot": {
        "expression": "$VrPrestacaoContrato$ / pow((1 + ($TxCetAnualContrato$/100)),($dj-d0$/365))",
        "output_column": "Resultado",
        "append_column": false
      },
      
      "input_schema": [
        {"name": "VrPrestacaoContrato", "type": "Float64"},
        {"name": "TxCetAnualContrato", "type": "Float64"},
        {"name": "dj-d0", "type": "Int64"}
      ],
      
      "generated_code": "df['Resultado'] = df['VrPrestacaoContrato'] / np.power((1 + (df['TxCetAnualContrato']/100)), (df['dj-d0']/365))",
      
      "imports_required": ["numpy as np"],
      
      "validation_results": [
        {
          "workflow": "workflow_cet_validation.knwf",
          "result": "PASS",
          "schema_match": true,
          "columns_expected": 27,
          "columns_generated": 27,
          "type_mismatches": [],
          "validated_at": "2025-12-15T10:05:00Z"
        },
        {
          "workflow": "workflow_cet_v2.knwf",
          "result": "PASS",
          "schema_match": true,
          "validated_at": "2025-12-15T11:00:00Z"
        }
      ],
      
      "promotion_criteria": {
        "min_validations": 3,
        "current_validations": 2,
        "required_pass_rate": 1.0,
        "current_pass_rate": 1.0
      },
      
      "generalization_pattern": {
        "is_generalizable": true,
        "pattern_description": "Math Formula com pow() → np.power()",
        "applicable_to": "Qualquer expressão JEP com pow()"
      }
    }
  ],
  
  "promotion_queue": [
    {
      "candidate_id": "cand_001",
      "ready_for_promotion": false,
      "reason": "Faltam 1 validação(ões)"
    }
  ]
}
```

### 4.4 rejected.json (Mapeamentos Rejeitados)

Mantém histórico de falhas para análise e melhoria do sistema.

```json
{
  "schema_version": "1.0",
  "last_updated": "2025-12-15T10:30:00Z",
  
  "rejected": [
    {
      "id": "rej_001",
      "original_candidate_id": "cand_000",
      "factory": "org.knime.base.node.rules.engine.RuleEngineNodeFactory",
      "name": "Rule Engine",
      "rejected_at": "2025-12-14T15:00:00Z",
      
      "failure_context": {
        "workflow": "workflow_classificacao.knwf",
        "node_id": "23",
        "node_display_name": "Rule Engine (#23)"
      },
      
      "generated_code": "# Código que falhou\ndf['Classificacao'] = np.where(df['Valor'] > 1000, 'Alto', 'Baixo')",
      
      "failure_details": {
        "type": "SCHEMA_MISMATCH",
        "expected_columns": ["Classificacao", "SubCategoria"],
        "generated_columns": ["Classificacao"],
        "missing_columns": ["SubCategoria"],
        "error_message": "Coluna SubCategoria não foi gerada"
      },
      
      "original_settings": {
        "rules": [
          "$Valor$ > 1000 => \"Alto\"",
          "$Valor$ <= 1000 AND $Categoria$ = \"A\" => \"Médio\"",
          "TRUE => \"Baixo\""
        ],
        "output_column": "Classificacao"
      },
      
      "analysis": {
        "root_cause": "IA não detectou segunda regra com output diferente",
        "suggested_fix": "Melhorar prompt para explicitar múltiplos outputs",
        "assigned_to": null,
        "fixed": false
      },
      
      "retry_count": 2,
      "max_retries": 3
    }
  ],
  
  "statistics": {
    "total_rejected": 12,
    "by_failure_type": {
      "SCHEMA_MISMATCH": 7,
      "SYNTAX_ERROR": 3,
      "RUNTIME_ERROR": 2
    },
    "by_node_type": {
      "Rule Engine": 5,
      "Java Snippet": 4,
      "String Manipulation": 3
    }
  }
}
```

### 4.5 validation_report.json (Relatório de Validação)

Gerado após cada conversão para verificar qualidade.

```json
{
  "report_id": "val_20251215_103000",
  "generated_at": "2025-12-15T10:30:00Z",
  
  "workflow_info": {
    "source_file": "workflow_cet_validation.knwf",
    "knime_version": "4.5.2",
    "total_nodes": 25
  },
  
  "overall_result": "PASS",
  
  "summary": {
    "nodes_validated": 25,
    "nodes_passed": 24,
    "nodes_failed": 1,
    "pass_rate": 0.96,
    "coverage": {
      "deterministic": 18,
      "ai_generated": 7
    }
  },
  
  "node_results": [
    {
      "node_id": "1",
      "node_name": "CSV Reader (#1)",
      "classification": "MAPPED",
      "result": "PASS",
      "validation_type": "schema_comparison",
      "details": {
        "expected_columns": 27,
        "actual_columns": 27,
        "column_matches": 27,
        "type_matches": 27
      }
    },
    {
      "node_id": "15",
      "node_name": "Math Formula (#15)",
      "classification": "CANDIDATE",
      "result": "PASS",
      "validation_type": "schema_comparison",
      "details": {
        "expected_columns": 27,
        "actual_columns": 27,
        "new_column_created": "Resultado",
        "new_column_type_expected": "Float64",
        "new_column_type_actual": "Float64"
      }
    },
    {
      "node_id": "23",
      "node_name": "Rule Engine (#23)",
      "classification": "CANDIDATE",
      "result": "FAIL",
      "validation_type": "schema_comparison",
      "details": {
        "expected_columns": 28,
        "actual_columns": 27,
        "missing_columns": ["SubCategoria"],
        "error_description": "Coluna SubCategoria não foi gerada pelo código"
      },
      "suggested_action": "Revisar prompt de Rule Engine para múltiplos outputs"
    }
  ],
  
  "divergences": [
    {
      "node_id": "23",
      "type": "MISSING_COLUMN",
      "severity": "ERROR",
      "column_name": "SubCategoria",
      "expected_type": "str",
      "description": "Coluna esperada não foi criada"
    }
  ],
  
  "recommendations": [
    {
      "priority": "HIGH",
      "type": "FIX_REQUIRED",
      "node_id": "23",
      "message": "Corrigir geração de Rule Engine para incluir todas as colunas de output"
    },
    {
      "priority": "LOW",
      "type": "OPTIMIZATION",
      "node_id": null,
      "message": "Considerar promover candidato 'Math Formula pow()' após mais 1 validação"
    }
  ]
}
```

### 4.6 coverage_report.json (Relatório de Cobertura)

Mostra quanto do workflow foi convertido por cada método.

```json
{
  "report_id": "cov_20251215_103000",
  "generated_at": "2025-12-15T10:30:00Z",
  
  "workflow_info": {
    "source_file": "workflow_cet_validation.knwf",
    "total_nodes": 25,
    "unique_factory_classes": 12
  },
  
  "coverage_summary": {
    "mapped_deterministic": {
      "count": 18,
      "percentage": 72.0,
      "confidence": 1.0
    },
    "mapped_candidate": {
      "count": 5,
      "percentage": 20.0,
      "confidence": 0.87
    },
    "unknown_ai_required": {
      "count": 2,
      "percentage": 8.0,
      "confidence": null
    }
  },
  
  "by_category": {
    "io": {
      "total": 4,
      "mapped": 4,
      "coverage": 1.0
    },
    "manipulation": {
      "total": 10,
      "mapped": 9,
      "coverage": 0.9
    },
    "transformation": {
      "total": 6,
      "mapped": 5,
      "coverage": 0.83
    },
    "logic": {
      "total": 3,
      "mapped": 2,
      "coverage": 0.67
    },
    "flow_control": {
      "total": 2,
      "mapped": 2,
      "coverage": 1.0
    }
  },
  
  "factory_breakdown": [
    {
      "factory": "org.knime.base.node.io.csvreader.CSVReaderNodeFactory",
      "name": "CSV Reader",
      "occurrences": 2,
      "status": "OFFICIAL",
      "coverage": 1.0
    },
    {
      "factory": "org.knime.base.node.preproc.filter.column.DataColumnSpecFilterNodeFactory",
      "name": "Column Filter",
      "occurrences": 5,
      "status": "OFFICIAL",
      "coverage": 1.0
    },
    {
      "factory": "org.knime.ext.jep.JEPNodeFactory",
      "name": "Math Formula",
      "occurrences": 3,
      "status": "CANDIDATE",
      "coverage": 0.90
    },
    {
      "factory": "org.knime.base.node.rules.engine.RuleEngineNodeFactory",
      "name": "Rule Engine",
      "occurrences": 2,
      "status": "OFFICIAL",
      "coverage": 0.85,
      "note": "Requires AI for rule interpretation"
    },
    {
      "factory": "com.custom.JavaSnippetNodeFactory",
      "name": "Java Snippet (Custom)",
      "occurrences": 1,
      "status": "UNKNOWN",
      "coverage": 0.0,
      "note": "Custom node - requires full AI interpretation"
    }
  ],
  
  "unmapped_nodes": [
    {
      "factory": "com.custom.JavaSnippetNodeFactory",
      "name": "Java Snippet (Custom)",
      "node_ids": ["22"],
      "recommendation": "Create new mapping or use AI"
    }
  ],
  
  "ai_token_estimate": {
    "nodes_requiring_ai": 2,
    "estimated_prompt_tokens": 1500,
    "estimated_completion_tokens": 800,
    "estimated_total_cost_usd": 0.03
  }
}
```

---

## 5. FASES DE IMPLEMENTAÇÃO

### 5.1 Visão Geral das Fases

| Fase | Nome | Objetivo | Tecnologia |
|------|------|----------|------------|
| 1 | Extração | Ler arquivos KNIME | Python + xml.etree |
| 2 | Representação Intermediária | Estruturar dados | Python + NetworkX |
| 3 | Classificação | Rotear para tradução | Python |
| 4A | Tradução Determinística | Aplicar templates | Python (string formatting) |
| 4B | Interpretação IA | Gerar código complexo | Google Vertex AI |
| 5 | Geração de Código | Criar arquivo .py | Python |
| 6 | Validação | Verificar corretude | Python |
| 7 | Aprendizado | Evoluir mapeamentos | Python |

### 5.2 Dependências entre Fases

```
Fase 1 ────► Fase 2 ────► Fase 3 ────┬──► Fase 4A ───┬──► Fase 5 ────► Fase 6 ────► Fase 7
                                     │               │
                                     └──► Fase 4B ───┘
```

---

## 6. DETALHAMENTO DAS ETAPAS

### 6.1 FASE 1: Extração

#### Etapa 1.1 - Descompactar ZIP

| Item | Descrição |
|------|-----------|
| **Objetivo** | Extrair conteúdo do arquivo .knwf |
| **Input** | Arquivo .knwf (é um ZIP renomeado) |
| **Output** | Pasta temporária com estrutura extraída |
| **Biblioteca** | `zipfile` (nativa Python) |
| **Critério de Sucesso** | Todos arquivos extraídos sem erro |
| **Tratamento de Erro** | Verificar se é ZIP válido antes de extrair |

#### Etapa 1.2 - Parsear workflow.knime

| Item | Descrição |
|------|-----------|
| **Objetivo** | Extrair lista de nodes e conexões |
| **Input** | Arquivo workflow.knime (XML) |
| **Output** | Dict com nodes[] e edges[] |
| **Biblioteca** | `xml.etree.ElementTree` |
| **Namespace** | `http://www.knime.org/2008/09/XMLConfig` |
| **Critério de Sucesso** | Nenhum node com ID nulo |
| **Elementos a Extrair** | `<config key="nodes">`, `<config key="connections">` |

#### Etapa 1.3 - Parsear settings.xml

| Item | Descrição |
|------|-----------|
| **Objetivo** | Extrair configurações de cada node |
| **Input** | Arquivo settings.xml de cada pasta de node |
| **Output** | Dict com factory, node-name, model, state |
| **Critério de Sucesso** | Factory extraído para 100% dos nodes |
| **Campos Críticos** | `factory`, `node-name`, `model/*`, `state` |

#### Etapa 1.4 - Parsear spec.xml

| Item | Descrição |
|------|-----------|
| **Objetivo** | Extrair schema de saída de cada porta |
| **Input** | Arquivo spec.xml em cada port_N/ |
| **Output** | Dict com columns[], types[], row_count |
| **Critério de Sucesso** | Schema extraído para nodes EXECUTED |
| **Campos Críticos** | `number_columns`, `column_spec_N/column_name`, `column_type/cell_class` |

#### Etapa 1.5 - Expandir Metanodes

| Item | Descrição |
|------|-----------|
| **Objetivo** | Processar metanodes recursivamente |
| **Input** | Pastas de metanodes (contêm workflow.knime interno) |
| **Output** | WorkflowIR aninhado para cada metanode |
| **Critério de Sucesso** | Todos metanodes expandidos |
| **Recursão** | Repetir etapas 1.2-1.5 para cada metanode |

---

### 6.2 FASE 2: Representação Intermediária

#### Etapa 2.1 - Construir Grafo

| Item | Descrição |
|------|-----------|
| **Objetivo** | Criar grafo direcionado de execução |
| **Input** | Nodes e edges da Fase 1 |
| **Output** | NetworkX DiGraph |
| **Biblioteca** | `networkx` |
| **Critério de Sucesso** | Grafo conectado (pode ter múltiplos componentes) |

#### Etapa 2.2 - Calcular Ordem Topológica

| Item | Descrição |
|------|-----------|
| **Objetivo** | Determinar ordem de execução |
| **Input** | DiGraph |
| **Output** | Lista ordenada de node_ids |
| **Algoritmo** | Kahn's Algorithm |
| **Critério de Sucesso** | Sem exceção de ciclo (loops tratados separadamente) |
| **Tratamento Especial** | Loops devem ser detectados antes |

#### Etapa 2.3 - Anotar Nodes

| Item | Descrição |
|------|-----------|
| **Objetivo** | Adicionar informações de entrada/saída a cada node |
| **Input** | Grafo + specs extraídos |
| **Output** | Grafo com atributos anotados |
| **Anotações** | input_ports[], output_ports[], schema, predecessors[], successors[] |
| **Critério de Sucesso** | Cada node tem input/output schema |

#### Etapa 2.4 - Identificar Estruturas Especiais

| Item | Descrição |
|------|-----------|
| **Objetivo** | Detectar loops e branches |
| **Input** | Grafo + settings com flow_stack |
| **Output** | Lista de loops com start/end/internals |
| **Identificadores** | `loopcontext`, `loopcontext_execute` em flow_stack |
| **Critério de Sucesso** | Loops têm start/end pareados |

#### Etapa 2.5 - Exportar IR

| Item | Descrição |
|------|-----------|
| **Objetivo** | Serializar grafo para JSON |
| **Input** | Grafo anotado |
| **Output** | workflow_ir.json |
| **Critério de Sucesso** | JSON válido, < 10MB |
| **Encoding** | UTF-8 |

---

### 6.3 FASE 3: Classificação e Roteamento

#### Etapa 3.1 - Carregar Mapeamentos

| Item | Descrição |
|------|-----------|
| **Objetivo** | Ler node_mapping.json |
| **Input** | Arquivo node_mapping.json |
| **Output** | Dict em memória |
| **Critério de Sucesso** | Arquivo carregado sem erro |
| **Cache** | Manter em memória durante execução |

#### Etapa 3.2 - Classificar Nodes

| Item | Descrição |
|------|-----------|
| **Objetivo** | Determinar método de tradução para cada node |
| **Input** | IR + mapeamentos |
| **Output** | Classificação: MAPPED \| CANDIDATE \| UNKNOWN |
| **Lógica** | factory em mappings.OFFICIAL → MAPPED; factory em candidates → CANDIDATE; else → UNKNOWN |
| **Critério de Sucesso** | 100% dos nodes classificados |

#### Etapa 3.3 - Gerar Relatório de Cobertura

| Item | Descrição |
|------|-----------|
| **Objetivo** | Reportar % de cobertura |
| **Input** | Classificações |
| **Output** | coverage_report.json |
| **Métricas** | % MAPPED, % CANDIDATE, % UNKNOWN, por categoria |
| **Critério de Sucesso** | Relatório gerado |

---

### 6.4 FASE 4A: Tradução Determinística

#### Etapa 4A.1 - Extrair Parâmetros

| Item | Descrição |
|------|-----------|
| **Objetivo** | Extrair valores do settings usando extraction_rules |
| **Input** | settings + rules do mapping |
| **Output** | Dict de parâmetros |
| **Exemplo** | path "model/url" → valor "/data/input.csv" |
| **Critério de Sucesso** | Todos params required extraídos |

#### Etapa 4A.2 - Aplicar Template

| Item | Descrição |
|------|-----------|
| **Objetivo** | Substituir placeholders no template |
| **Input** | template + params |
| **Output** | Código Python |
| **Método** | String formatting (.format() ou f-string) |
| **Critério de Sucesso** | Código sintaticamente válido |

#### Etapa 4A.3 - Resolver Variáveis

| Item | Descrição |
|------|-----------|
| **Objetivo** | Substituir referências de input/output |
| **Input** | Código + contexto (predecessors, node_id) |
| **Output** | Código com variáveis corretas |
| **Exemplo** | `df_{input_var}` → `df_node_1` |
| **Critério de Sucesso** | Nenhum placeholder restante |

---

### 6.5 FASE 4B: Interpretação IA

#### Etapa 4B.1 - Montar Prompt

| Item | Descrição |
|------|-----------|
| **Objetivo** | Construir prompt estruturado |
| **Input** | node + schema + settings |
| **Output** | String prompt |
| **Limite** | < 4000 tokens |
| **Componentes** | Contexto, regras/expressão, colunas disponíveis, formato esperado |

**Template de Prompt:**

```
CONTEXTO:
Você é um especialista em migração de workflows KNIME para Python/Pandas.

NODE KNIME:
- Tipo: {factory}
- Nome: {node_name}
- Descrição: {annotation}

CONFIGURAÇÕES:
{settings_json}

SCHEMA DE ENTRADA:
{input_columns_with_types}

TAREFA:
Gere código Python que replique exatamente o comportamento deste node KNIME.

REQUISITOS:
1. Input: DataFrame 'df_{input_var}' com as colunas listadas acima
2. Output: DataFrame 'df_{output_var}' com a transformação aplicada
3. Preservar tipos de dados
4. Não usar loops - preferir operações vetorizadas
5. Incluir comentário explicando a transformação

FORMATO DE RESPOSTA:
Retorne APENAS o código Python, sem explicações adicionais.
```

#### Etapa 4B.2 - Chamar Vertex AI

| Item | Descrição |
|------|-----------|
| **Objetivo** | Enviar prompt e receber código |
| **Input** | Prompt |
| **Output** | Resposta com código |
| **Modelo** | gemini-1.5-pro |
| **Timeout** | 30 segundos |
| **Retry** | 3 tentativas com backoff exponencial |
| **Critério de Sucesso** | Resposta em < 30s |

#### Etapa 4B.3 - Validar Sintaxe

| Item | Descrição |
|------|-----------|
| **Objetivo** | Verificar se código é válido |
| **Input** | Código gerado |
| **Output** | Bool válido + erros se houver |
| **Método** | `ast.parse()` |
| **Critério de Sucesso** | Parse sem exceção |

#### Etapa 4B.4 - Registrar Candidato

| Item | Descrição |
|------|-----------|
| **Objetivo** | Salvar código para validação futura |
| **Input** | Código válido + contexto |
| **Output** | Entrada em candidates.json |
| **Dados Salvos** | factory, settings_snapshot, generated_code, source_workflow |
| **Critério de Sucesso** | Candidato persistido |

---

### 6.6 FASE 5: Geração de Código

#### Etapa 5.1 - Coletar Imports

| Item | Descrição |
|------|-----------|
| **Objetivo** | Agregar todos imports necessários |
| **Input** | Todos os nodes traduzidos |
| **Output** | Lista de imports únicos |
| **Deduplicação** | Remover duplicatas |
| **Ordem** | stdlib → third-party → local |
| **Critério de Sucesso** | Sem duplicatas |

#### Etapa 5.2 - Gerar Funções

| Item | Descrição |
|------|-----------|
| **Objetivo** | Criar função ou bloco para cada node |
| **Input** | Código por node |
| **Output** | Funções Python |
| **Estratégia** | Inline para simples, função para complexos |
| **Critério de Sucesso** | Cada node = 1 bloco identificável |

#### Etapa 5.3 - Gerar Main

| Item | Descrição |
|------|-----------|
| **Objetivo** | Criar função principal com orquestração |
| **Input** | Ordem topológica + funções |
| **Output** | Função main() |
| **Responsabilidades** | Chamar funções na ordem, passar DataFrames |
| **Critério de Sucesso** | Respeita ordem topológica |

#### Etapa 5.4 - Adicionar Documentação

| Item | Descrição |
|------|-----------|
| **Objetivo** | Inserir docstrings e comentários |
| **Input** | Metadados KNIME |
| **Output** | Código documentado |
| **Conteúdo** | Origem KNIME, transformação, colunas |
| **Critério de Sucesso** | Rastreabilidade completa |

---

### 6.7 FASE 6: Validação Cruzada

#### Etapa 6.1 - Comparar Schemas

| Item | Descrição |
|------|-----------|
| **Objetivo** | Verificar se output bate com spec.xml |
| **Input** | Código + spec.xml de saída |
| **Output** | Lista de divergências |
| **Comparações** | Nomes de colunas, tipos, quantidade |
| **Critério de Sucesso** | Lista gerada (pode ser vazia) |

#### Etapa 6.2 - Gerar Relatório

| Item | Descrição |
|------|-----------|
| **Objetivo** | Documentar resultado da validação |
| **Input** | Divergências |
| **Output** | validation_report.json |
| **Conteúdo** | Por node: PASS/FAIL, detalhes |
| **Critério de Sucesso** | Relatório completo |

#### Etapa 6.3 - Classificar Resultado

| Item | Descrição |
|------|-----------|
| **Objetivo** | Determinar sucesso global |
| **Input** | Divergências |
| **Output** | PASS ou FAIL |
| **Critério PASS** | 0 divergências críticas |
| **Critério FAIL** | ≥1 divergência crítica |

---

### 6.8 FASE 7: Ciclo de Aprendizado

#### Etapa 7.1 - Promover Candidatos

| Item | Descrição |
|------|-----------|
| **Objetivo** | Mover CANDIDATE → OFFICIAL |
| **Input** | Candidatos com validations ≥ 3 e pass_rate = 1.0 |
| **Output** | node_mapping.json atualizado |
| **Critério** | min_validations atingido + 100% pass rate |
| **Ação** | Criar nova entrada em mappings[] |

#### Etapa 7.2 - Rejeitar Falhas

| Item | Descrição |
|------|-----------|
| **Objetivo** | Mover candidatos que falharam para rejected |
| **Input** | Candidatos com FAIL |
| **Output** | rejected.json atualizado |
| **Dados** | Código, erro, contexto |
| **Ação** | Preservar para análise |

#### Etapa 7.3 - Atualizar Mapeamentos

| Item | Descrição |
|------|-----------|
| **Objetivo** | Persistir mudanças |
| **Input** | Promoções e rejeições |
| **Output** | Arquivos JSON atualizados |
| **Versionamento** | Incrementar version, atualizar last_updated |
| **Critério de Sucesso** | Arquivos salvos sem erro |

#### Etapa 7.4 - Calcular Métricas

| Item | Descrição |
|------|-----------|
| **Objetivo** | Rastrear evolução do sistema |
| **Input** | Histórico de conversões |
| **Output** | Métricas agregadas |
| **Métricas** | Cobertura, taxa IA, promotions/rejections |
| **Critério de Sucesso** | Taxa de IA diminuindo |

---

## 7. CICLO DE APRENDIZADO

### 7.1 Fluxo de Estados

```
                    ┌─────────────────────────────────────────┐
                    │                                         │
                    ▼                                         │
┌─────────────┐     ┌─────────────┐     ┌─────────────┐      │
│  UNKNOWN    │ ──▶ │  CANDIDATE  │ ──▶ │  OFFICIAL   │      │
│  (IA gera)  │     │ (validando) │     │ (template)  │      │
└─────────────┘     └─────────────┘     └─────────────┘      │
                           │                   │              │
                           │ (se falhar)       │              │
                           ▼                   │              │
                    ┌─────────────┐            │              │
                    │  REJECTED   │            │              │
                    │  (com log)  │            │              │
                    └─────────────┘            │              │
                           │                   │              │
                           │ (análise manual)  │              │
                           └───────────────────┴──────────────┘
```

### 7.2 Critérios de Transição

| Transição | De | Para | Critério |
|-----------|-----|------|----------|
| Geração | UNKNOWN | CANDIDATE | IA gera código sintaticamente válido |
| Promoção | CANDIDATE | OFFICIAL | ≥3 validações com 100% pass rate |
| Rejeição | CANDIDATE | REJECTED | Falha em validação |
| Recuperação | REJECTED | CANDIDATE | Análise manual + correção |

### 7.3 Generalização de Padrões

Quando um candidato é promovido, o sistema tenta generalizar o padrão:

1. **Identificar variáveis**: Quais partes do código dependem de settings?
2. **Criar template**: Substituir valores específicos por placeholders
3. **Documentar extraction_rules**: Como extrair valores do settings.xml
4. **Validar template**: Testar com outros nodes similares

---

## 8. MÉTRICAS E CRITÉRIOS DE SUCESSO

### 8.1 Métricas Principais

| Métrica | Descrição | Meta | Fórmula |
|---------|-----------|------|---------|
| Cobertura Determinística | % de nodes com template | ≥ 80% | MAPPED / total |
| Precisão de Validação | % de schemas corretos | ≥ 95% | PASS / validados |
| Redução de Chamadas IA | Economia ao longo do tempo | -70% em 30 dias | chamadas_atual / chamadas_inicial |
| Tempo por Workflow | Duração total | < 60s | end_time - start_time |
| Taxa de Promoção | % de candidatos promovidos | ≥ 80% | promoted / candidates |

### 8.2 Dashboard de Acompanhamento

```json
{
  "dashboard": {
    "period": "2025-12-01 to 2025-12-15",
    "workflows_processed": 50,
    "nodes_processed": 1250,
    
    "coverage_evolution": [
      {"date": "2025-12-01", "deterministic": 0.45, "candidate": 0.20, "unknown": 0.35},
      {"date": "2025-12-08", "deterministic": 0.60, "candidate": 0.25, "unknown": 0.15},
      {"date": "2025-12-15", "deterministic": 0.78, "candidate": 0.15, "unknown": 0.07}
    ],
    
    "ai_calls_per_week": [
      {"week": 1, "calls": 450},
      {"week": 2, "calls": 280},
      {"week": 3, "calls": 120}
    ],
    
    "validation_success_rate": {
      "deterministic": 1.00,
      "ai_generated": 0.87
    },
    
    "top_unknown_nodes": [
      {"factory": "com.custom.JavaSnippet", "occurrences": 15},
      {"factory": "org.knime.python.nodes.script", "occurrences": 8}
    ]
  }
}
```

### 8.3 Alertas

| Alerta | Condição | Ação |
|--------|----------|------|
| Cobertura baixa | < 60% MAPPED | Revisar node_mapping.json |
| Muitas falhas | > 20% FAIL | Revisar prompts de IA |
| IA lenta | > 30s por node | Verificar quota/rede |
| Candidatos estagnados | > 7 dias sem promoção | Rodar mais workflows |

---

## 9. CRONOGRAMA DE IMPLEMENTAÇÃO

### 9.1 Visão Geral

| Sprint | Semanas | Fases | Entregável |
|--------|---------|-------|------------|
| 1 | 1-2 | Fase 1 | Extração funcionando |
| 2 | 3 | Fase 2 | workflow_ir.json gerado |
| 3 | 4 | Fase 3 + 4A | Tradução determinística |
| 4 | 5 | Fase 4B | Integração com IA |
| 5 | 6 | Fase 5 | Código Python gerado |
| 6 | 7 | Fases 6 + 7 | Sistema completo |

### 9.2 Detalhamento por Sprint

#### Sprint 1 (Semanas 1-2): Fundação

```
┌─────────────────────────────────────────────────────────────────┐
│  SPRINT 1: FUNDAÇÃO                                             │
│  ├── Etapa 1.1: Extrator ZIP                                    │
│  ├── Etapa 1.2: Parser workflow.knime                           │
│  ├── Etapa 1.3: Parser settings.xml                             │
│  └── Etapa 1.4: Parser spec.xml                                 │
│                                                                 │
│  ENTREGÁVEL: Extração completa funcionando                      │
│  VALIDAÇÃO: Processar 3 workflows de teste sem erro             │
└─────────────────────────────────────────────────────────────────┘
```

#### Sprint 2 (Semana 3): Representação Intermediária

```
┌─────────────────────────────────────────────────────────────────┐
│  SPRINT 2: REPRESENTAÇÃO INTERMEDIÁRIA                          │
│  ├── Etapa 2.1-2.2: Grafo + Ordenação                           │
│  ├── Etapa 2.3: Anotações de schema                             │
│  └── Etapa 2.5: Exportar workflow_ir.json                       │
│                                                                 │
│  ENTREGÁVEL: workflow_ir.json gerado corretamente               │
│  VALIDAÇÃO: IR contém todos nodes e edges do workflow           │
└─────────────────────────────────────────────────────────────────┘
```

#### Sprint 3 (Semana 4): Mapeamento Determinístico

```
┌─────────────────────────────────────────────────────────────────┐
│  SPRINT 3: MAPEAMENTO DETERMINÍSTICO                            │
│  ├── Criar node_mapping.json inicial (20 nodes comuns)          │
│  ├── Etapa 3.1-3.3: Classificador                               │
│  └── Etapa 4A: Tradução determinística                          │
│                                                                 │
│  ENTREGÁVEL: 70% dos nodes traduzidos sem IA                    │
│  VALIDAÇÃO: coverage_report mostra ≥70% MAPPED                  │
└─────────────────────────────────────────────────────────────────┘
```

#### Sprint 4 (Semana 5): Integração com IA

```
┌─────────────────────────────────────────────────────────────────┐
│  SPRINT 4: INTEGRAÇÃO COM IA                                    │
│  ├── Etapa 4B.1: Prompt builder                                 │
│  ├── Etapa 4B.2: Cliente Vertex AI                              │
│  └── Etapa 4B.3-4B.4: Validação + Candidatos                    │
│                                                                 │
│  ENTREGÁVEL: Nodes desconhecidos interpretados por IA           │
│  VALIDAÇÃO: IA responde em <30s com código válido               │
└─────────────────────────────────────────────────────────────────┘
```

#### Sprint 5 (Semana 6): Geração de Código

```
┌─────────────────────────────────────────────────────────────────┐
│  SPRINT 5: GERAÇÃO DE CÓDIGO                                    │
│  ├── Etapa 5.1-5.4: Gerador Python completo                     │
│  └── Etapa 1.5 + 2.4: Metanodes e Loops                         │
│                                                                 │
│  ENTREGÁVEL: Código Python executável gerado                    │
│  VALIDAÇÃO: Código executa sem SyntaxError                      │
└─────────────────────────────────────────────────────────────────┘
```

#### Sprint 6 (Semana 7): Validação e Aprendizado

```
┌─────────────────────────────────────────────────────────────────┐
│  SPRINT 6: VALIDAÇÃO E APRENDIZADO                              │
│  ├── Etapa 6.1-6.3: Validação cruzada                           │
│  └── Etapa 7.1-7.4: Ciclo de aprendizado                        │
│                                                                 │
│  ENTREGÁVEL: Sistema auto-evolutivo funcionando                 │
│  VALIDAÇÃO: Candidatos sendo promovidos automaticamente         │
└─────────────────────────────────────────────────────────────────┘
```

---

## 10. GLOSSÁRIO

| Termo | Definição |
|-------|-----------|
| **KNIME** | Konstanz Information Miner - plataforma de workflow de dados |
| **Node** | Unidade de processamento no KNIME (equivale a uma função) |
| **Metanode** | Grupo de nodes encapsulado (equivale a um sub-workflow) |
| **Factory** | Classe Java que identifica unicamente um tipo de node |
| **settings.xml** | Arquivo de configuração de cada node |
| **spec.xml** | Arquivo com schema de saída de cada porta |
| **workflow.knime** | Arquivo principal com estrutura do workflow |
| **IR** | Intermediate Representation - estrutura intermediária |
| **Template** | Código Python com placeholders para substituição |
| **Candidato** | Mapeamento gerado por IA em validação |
| **MAPPED** | Node com template determinístico oficial |
| **UNKNOWN** | Node sem mapeamento, requer IA |
| **Validação Cruzada** | Comparação do output gerado com spec.xml esperado |
| **Promoção** | Transição de CANDIDATE para OFFICIAL |
| **Vertex AI** | Plataforma de IA do Google Cloud |

---

## 📝 NOTAS FINAIS

### Princípios de Design

1. **Fail Fast**: Detectar erros o mais cedo possível no pipeline
2. **Idempotência**: Rodar múltiplas vezes produz mesmo resultado
3. **Rastreabilidade**: Toda linha de código gerado tem origem documentada
4. **Evolução Gradual**: Sistema melhora a cada workflow processado

### Riscos e Mitigações

| Risco | Mitigação |
|-------|-----------|
| IA gera código errado | Validação cruzada com spec.xml |
| Nodes customizados | Fallback para interpretação manual |
| Performance lenta | Cache de mapeamentos, processamento paralelo |
| Custo de IA alto | Priorizar tradução determinística |

### Próximos Passos

1. Implementar Etapa 1.1 (Extrator ZIP)
2. Criar node_mapping.json com 20 nodes mais comuns
3. Configurar ambiente Vertex AI
4. Definir workflows de teste para validação

---

**Documento gerado em:** 2025-12-15  
**Versão:** 2.0  
**Status:** Aprovado para implementação
