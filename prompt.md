# 🔍 AVALIAÇÃO DAS TRÊS SUGESTÕES

## Análise Individual

### 1. Representação Intermediária (RI) como Grafo Completo

| Aspecto | Avaliação |
|---------|-----------|
| **Faz sentido?** | ✅ **SIM - ESSENCIAL** |
| **Por quê?** | Lista ordenada perde informação crítica: múltiplas entradas, branches paralelos, contexto de vizinhança |
| **Benefício adicional** | IA pode entender padrões como "Filter → Join" e otimizar |
| **Risco de não fazer** | Código gerado pode ter variáveis de entrada erradas |

**Exemplo do problema com apenas ordem:**
```
Ordem: [Node1, Node2, Node3, Node4]
Mas na verdade: Node1 → Node2
               Node1 → Node3 → Node4
                       ↑
               Node2 ──┘
```
Sem o grafo, a IA não sabe que Node4 recebe de Node2 E Node3.

---

### 2. Mapeamento Determinístico (node_mapping.json)

| Aspecto | Avaliação |
|---------|-----------|
| **Faz sentido?** | ✅ **SIM - CRÍTICO** |
| **Por quê?** | ~70% dos nodes em workflows típicos são simples e determinísticos |
| **Economia** | Reduz chamadas à IA em 70%, economiza tokens e tempo |
| **Precisão** | Template validado = 100% correto, IA = ~85-95% |

**Análise do código legado:**
Olhando o `knime_to_python_converter_banking.txt`, já existe um padrão de mapeamento, mas está hardcoded. Externalizar para JSON permite:
- Atualizar sem mexer no código
- Versionar separadamente
- Contribuições de outros desenvolvedores

---

### 3. Ciclo de Aprendizado (Candidatos → Validação → Oficial)

| Aspecto | Avaliação |
|---------|-----------|
| **Faz sentido?** | ✅ **SIM - DIFERENCIAL COMPETITIVO** |
| **Por quê?** | Sistema evolui sozinho, reduzindo dependência de IA ao longo do tempo |
| **Implementação** | Precisa de estados claros e critérios de promoção |
| **Risco** | Sem validação rigorosa, pode propagar erros |

**Fluxo proposto:**
```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  UNKNOWN    │ ──▶ │  CANDIDATE  │ ──▶ │  OFFICIAL   │
│  (IA gera)  │     │ (validando) │     │ (template)  │
└─────────────┘     └─────────────┘     └─────────────┘
                           │
                           ▼ (se falhar)
                    ┌─────────────┐
                    │  REJECTED   │
                    │  (com log)  │
                    └─────────────┘
```

---

## ✅ CONCLUSÃO: TODAS AS TRÊS FAZEM SENTIDO

Integrar as três sugestões cria um sistema com:
- **Precisão** (grafo completo)
- **Eficiência** (mapeamento determinístico)
- **Evolução** (ciclo de aprendizado)

---

# 📋 PLANO REVISADO E CONSOLIDADO

## Arquitetura Final

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
│  FASE 2: REPRESENTAÇÃO INTERMEDIÁRIA (RI)                         [NOVO]    │
│  ├── 2.1 Construir grafo direcionado completo                               │
│  ├── 2.2 Calcular ordem topológica                                          │
│  ├── 2.3 Anotar cada node com: inputs, outputs, schema                      │
│  ├── 2.4 Identificar estruturas especiais (loops, branches)                 │
│  └── 2.5 Exportar workflow_ir.json                                          │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│  FASE 3: CLASSIFICAÇÃO E ROTEAMENTO                               [NOVO]    │
│  ├── 3.1 Carregar node_mapping.json (mapeamentos oficiais)                  │
│  ├── 3.2 Classificar cada node: MAPPED | CANDIDATE | UNKNOWN                │
│  └── 3.3 Gerar relatório de cobertura                                       │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                    ┌─────────────────┴─────────────────┐
                    ▼                                   ▼
┌───────────────────────────────┐     ┌───────────────────────────────────────┐
│  FASE 4A: TRADUÇÃO            │     │  FASE 4B: INTERPRETAÇÃO IA            │
│  DETERMINÍSTICA               │     │  (apenas para UNKNOWN)                │
│  ├── Aplicar templates        │     │  ├── 4B.1 Montar prompt estruturado   │
│  └── 100% precisão            │     │  ├── 4B.2 Chamar Vertex AI            │
└───────────────────────────────┘     │  ├── 4B.3 Validar sintaxe             │
                    │                 │  └── 4B.4 Marcar como CANDIDATE        │
                    │                 └───────────────────────────────────────┘
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
│  FASE 7: CICLO DE APRENDIZADO                                     [NOVO]    │
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

---

## 📁 ESTRUTURA DE ARQUIVOS DO PROJETO

```
knime_converter/
├── config/
│   ├── node_mapping.json          # Mapeamentos OFICIAIS (determinísticos)
│   ├── candidates.json            # Mapeamentos em validação
│   ├── rejected.json              # Mapeamentos que falharam (para análise)
│   └── knime_types.json           # Mapeamento de tipos KNIME → Python
│
├── output/
│   ├── workflow_ir.json           # Representação Intermediária do último workflow
│   ├── generated_code.py          # Código Python gerado
│   ├── validation_report.json     # Relatório de validação
│   └── coverage_report.json       # Relatório de cobertura de nodes
│
├── src/
│   ├── extractors/                # FASE 1
│   │   ├── zip_extractor.py
│   │   ├── workflow_parser.py
│   │   ├── settings_parser.py
│   │   └── spec_parser.py
│   │
│   ├── ir/                        # FASE 2
│   │   ├── graph_builder.py
│   │   ├── topological_sort.py
│   │   └── ir_exporter.py
│   │
│   ├── mapping/                   # FASE 3 + 4A
│   │   ├── classifier.py
│   │   └── deterministic_translator.py
│   │
│   ├── ai/                        # FASE 4B
│   │   ├── prompt_builder.py
│   │   ├── vertex_client.py
│   │   └── response_validator.py
│   │
│   ├── codegen/                   # FASE 5
│   │   ├── python_generator.py
│   │   └── docstring_generator.py
│   │
│   ├── validation/                # FASE 6
│   │   ├── schema_comparator.py
│   │   └── report_generator.py
│   │
│   └── learning/                  # FASE 7
│       ├── promoter.py
│       └── metrics_tracker.py
│
├── main.py                        # Orquestrador principal
└── cli.py                         # Interface de linha de comando
```

---

## 📐 SCHEMA DOS ARQUIVOS JSON

### 1. workflow_ir.json (Representação Intermediária)

```json
{
  "metadata": {
    "source_file": "workflow.knwf",
    "knime_version": "4.5.2",
    "extracted_at": "2025-12-15T10:30:00Z",
    "author": "andre.carlucci"
  },
  "graph": {
    "nodes": {
      "1": {
        "id": "1",
        "name": "CSV Reader",
        "factory": "org.knime.base.node.io.csvreader.CSVReaderNodeFactory",
        "display_name": "CSV Reader (#1)",
        "is_metanode": false,
        "is_loop_start": false,
        "is_loop_end": false,
        "position": {"x": 100, "y": 200},
        "state": "EXECUTED",
        "settings": {
          "file_path": "/data/input.csv",
          "delimiter": ",",
          "has_header": true
        },
        "input_ports": [],
        "output_ports": [
          {
            "index": 0,
            "type": "data",
            "schema": {
              "columns": [
                {"name": "NuContrato", "knime_type": "DoubleCell", "python_type": "Float64"},
                {"name": "NmProduto", "knime_type": "StringCell", "python_type": "str"}
              ],
              "row_count": 48
            }
          }
        ]
      },
      "2": {
        "id": "2",
        "name": "Column Filter",
        "factory": "org.knime.base.node.preproc.filter.column.DataColumnSpecFilterNodeFactory",
        "settings": {
          "included_columns": ["NuContrato", "NmProduto"],
          "excluded_columns": ["TempCol1"]
        },
        "input_ports": [
          {"index": 0, "type": "data", "source_node": "1", "source_port": 0}
        ],
        "output_ports": [
          {"index": 0, "type": "data", "schema": {"columns": [/*...*/]}}
        ]
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
      }
    ],
    "execution_order": ["1", "2", "3", "4"],
    "loops": [
      {
        "start_node": "5",
        "end_node": "8",
        "internal_nodes": ["6", "7"],
        "loop_variables": ["currentIteration", "NuContrato"]
      }
    ],
    "metanodes": {
      "10": {
        "name": "CRIA DATA DE REFERÊNCIA",
        "internal_workflow": {/* workflow_ir recursivo */}
      }
    }
  }
}
```

---

### 2. node_mapping.json (Mapeamentos Oficiais)

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
    }
  },
  "mappings": [
    {
      "factory": "org.knime.base.node.io.csvreader.CSVReaderNodeFactory",
      "name": "CSV Reader",
      "category": "io",
      "status": "OFFICIAL",
      "confidence": 1.0,
      "settings_extraction": {
        "file_path": "model/url",
        "delimiter": "model/colDelimiter",
        "has_header": "model/hasColHeader"
      },
      "python_template": "df_{output_var} = pd.read_csv('{file_path}', sep='{delimiter}', header={has_header_int})",
      "imports": ["pandas as pd"],
      "validation_count": 127,
      "last_validated": "2025-12-14"
    },
    {
      "factory": "org.knime.base.node.preproc.filter.column.DataColumnSpecFilterNodeFactory",
      "name": "Column Filter",
      "category": "manipulation",
      "status": "OFFICIAL",
      "confidence": 1.0,
      "settings_extraction": {
        "included_columns": "model/column-filter/included_names",
        "mode": "model/column-filter/filter-type"
      },
      "python_template": "df_{output_var} = df_{input_var}[{included_columns}]",
      "imports": [],
      "validation_count": 89
    },
    {
      "factory": "org.knime.base.node.preproc.joiner.Joiner2NodeFactory",
      "name": "Joiner",
      "category": "manipulation",
      "status": "OFFICIAL",
      "settings_extraction": {
        "left_key": "model/leftTableJoinPredicate/0/leftColumn",
        "right_key": "model/rightTableJoinPredicate/0/rightColumn",
        "join_type": "model/joinMode"
      },
      "python_template": "df_{output_var} = df_{input_var_0}.merge(df_{input_var_1}, left_on='{left_key}', right_on='{right_key}', how='{join_type}')",
      "join_type_mapping": {
        "InnerJoin": "inner",
        "LeftOuterJoin": "left",
        "RightOuterJoin": "right",
        "FullOuterJoin": "outer"
      }
    },
    {
      "factory": "org.knime.base.node.rules.engine.RuleEngineNodeFactory",
      "name": "Rule Engine",
      "category": "logic",
      "status": "OFFICIAL",
      "complexity": "high",
      "requires_ai_for_rules": true,
      "settings_extraction": {
        "rules": "model/rules",
        "output_column": "model/new-column-name",
        "append_column": "model/append-column"
      },
      "python_template": "# Rule Engine: requires AI interpretation\n{ai_generated_code}",
      "ai_prompt_hint": "Convert KNIME Rule Engine syntax to pandas np.where or np.select"
    }
  ]
}
```

---

### 3. candidates.json (Candidatos em Validação)

```json
{
  "candidates": [
    {
      "factory": "org.knime.ext.jep.JEPNodeFactory",
      "name": "Math Formula",
      "status": "CANDIDATE",
      "generated_by": "vertex-ai-gemini-1.5",
      "generated_at": "2025-12-15T10:00:00Z",
      "source_workflow": "workflow_cet_validation.knwf",
      "settings_snapshot": {
        "expression": "$VrPrestacaoContrato$ / pow((1 + ($TxCetAnualContrato$/100)),($dj-d0$/365))"
      },
      "generated_code": "df['Resultado'] = df['VrPrestacaoContrato'] / np.power((1 + (df['TxCetAnualContrato']/100)), (df['dj-d0']/365))",
      "validation_results": [
        {
          "workflow": "workflow_cet_validation.knwf",
          "result": "PASS",
          "schema_match": true,
          "validated_at": "2025-12-15T10:05:00Z"
        }
      ],
      "promotion_criteria": {
        "min_validations": 3,
        "current_validations": 1,
        "required_pass_rate": 1.0
      }
    }
  ]
}
```

---

## 📋 PLANO DE ETAPAS DETALHADO

### FASE 1: EXTRAÇÃO

| Etapa | Descrição | Input | Output | Critério de Sucesso |
|-------|-----------|-------|--------|---------------------|
| 1.1 | Descompactar ZIP | .knwf | pasta temporária | Todos arquivos extraídos |
| 1.2 | Parsear workflow.knime | XML | dict{nodes, connections} | Nenhum node com ID nulo |
| 1.3 | Parsear settings.xml | XML por node | dict{factory, model, state} | Factory extraído para 100% |
| 1.4 | Parsear spec.xml | XML por porta | dict{columns, types, count} | Schema extraído para nodes EXECUTED |
| 1.5 | Expandir metanodes | pastas de metanodes | WorkflowIR recursivo | Todos metanodes expandidos |

---

### FASE 2: REPRESENTAÇÃO INTERMEDIÁRIA

| Etapa | Descrição | Input | Output | Critério de Sucesso |
|-------|-----------|-------|--------|---------------------|
| 2.1 | Construir grafo | nodes + edges | DiGraph (NetworkX) | Grafo conectado |
| 2.2 | Ordenação topológica | DiGraph | lista ordenada | Sem exceção de ciclo (exceto loops) |
| 2.3 | Anotar nodes | grafo + specs | grafo anotado | Cada node tem input/output schema |
| 2.4 | Identificar estruturas | grafo | loops[], branches[] | Loops têm start/end pareados |
| 2.5 | Exportar IR | grafo anotado | workflow_ir.json | JSON válido, < 10MB |

---

### FASE 3: CLASSIFICAÇÃO

| Etapa | Descrição | Input | Output | Critério de Sucesso |
|-------|-----------|-------|--------|---------------------|
| 3.1 | Carregar mappings | node_mapping.json | dict em memória | Arquivo carregado sem erro |
| 3.2 | Classificar nodes | IR + mappings | {MAPPED, CANDIDATE, UNKNOWN} | 100% classificados |
| 3.3 | Relatório cobertura | classificação | coverage_report.json | % mapeado calculado |

---

### FASE 4A: TRADUÇÃO DETERMINÍSTICA

| Etapa | Descrição | Input | Output | Critério de Sucesso |
|-------|-----------|-------|--------|---------------------|
| 4A.1 | Extrair parâmetros | settings + extraction_rules | dict de parâmetros | Todos params extraídos |
| 4A.2 | Aplicar template | template + params | código Python | Código sintaxe válida |
| 4A.3 | Resolver variáveis | código + contexto | código com vars corretas | Nenhum placeholder restante |

---

### FASE 4B: INTERPRETAÇÃO IA

| Etapa | Descrição | Input | Output | Critério de Sucesso |
|-------|-----------|-------|--------|---------------------|
| 4B.1 | Montar prompt | node + schema + settings | prompt estruturado | < 4000 tokens |
| 4B.2 | Chamar Vertex AI | prompt | resposta com código | Resposta em < 30s |
| 4B.3 | Validar sintaxe | código gerado | AST parse | ast.parse() sem erro |
| 4B.4 | Registrar candidato | código válido | entrada em candidates.json | Candidato salvo |

---

### FASE 5: GERAÇÃO DE CÓDIGO

| Etapa | Descrição | Input | Output | Critério de Sucesso |
|-------|-----------|-------|--------|---------------------|
| 5.1 | Coletar imports | todos os nodes | lista de imports | Sem duplicatas |
| 5.2 | Gerar funções | código por node | funções Python | Cada node = 1 bloco |
| 5.3 | Gerar main() | ordem topológica | função principal | Respeita ordem |
| 5.4 | Adicionar docs | metadados | docstrings | Rastreabilidade KNIME→Python |

---

### FASE 6: VALIDAÇÃO CRUZADA

| Etapa | Descrição | Input | Output | Critério de Sucesso |
|-------|-----------|-------|--------|---------------------|
| 6.1 | Comparar schemas | código + spec.xml | divergências | Lista de diferenças |
| 6.2 | Gerar relatório | divergências | validation_report.json | Relatório completo |
| 6.3 | Classificar resultado | divergências | PASS/FAIL | 0 divergências = PASS |

---

### FASE 7: CICLO DE APRENDIZADO

| Etapa | Descrição | Input | Output | Critério de Sucesso |
|-------|-----------|-------|--------|---------------------|
| 7.1 | Promover candidatos | PASS + criteria | OFFICIAL em node_mapping | validation_count ≥ 3 |
| 7.2 | Rejeitar falhas | FAIL | REJECTED em rejected.json | Log com motivo |
| 7.3 | Atualizar mappings | promoções | node_mapping.json atualizado | Arquivo versionado |
| 7.4 | Calcular métricas | histórico | metrics.json | Taxa de IA diminuindo |

---

## 📊 MÉTRICAS DE EVOLUÇÃO DO SISTEMA

```json
{
  "metrics": {
    "total_workflows_processed": 50,
    "total_nodes_processed": 1250,
    "mapping_coverage": {
      "initial": 0.45,
      "current": 0.78,
      "target": 0.95
    },
    "ai_calls": {
      "first_week": 450,
      "current_week": 120,
      "reduction_rate": 0.73
    },
    "validation_success_rate": {
      "deterministic": 1.0,
      "ai_generated": 0.87
    },
    "promotions": {
      "total_candidates": 35,
      "promoted_to_official": 28,
      "rejected": 7
    }
  }
}
```

---

## 🎯 ORDEM DE IMPLEMENTAÇÃO REVISADA

```
┌─────────────────────────────────────────────────────────────────┐
│  SPRINT 1 (Semana 1-2): FUNDAÇÃO                                │
│  ├── Etapa 1.1: Extrator ZIP                                    │
│  ├── Etapa 1.2: Parser workflow.knime                           │
│  ├── Etapa 1.3: Parser settings.xml                             │
│  └── Etapa 1.4: Parser spec.xml                                 │
│  ENTREGÁVEL: Extração completa funcionando                      │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  SPRINT 2 (Semana 3): REPRESENTAÇÃO INTERMEDIÁRIA               │
│  ├── Etapa 2.1-2.2: Grafo + Ordenação                           │
│  ├── Etapa 2.3: Anotações de schema                             │
│  └── Etapa 2.5: Exportar workflow_ir.json                       │
│  ENTREGÁVEL: workflow_ir.json gerado corretamente               │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  SPRINT 3 (Semana 4): MAPEAMENTO DETERMINÍSTICO                 │
│  ├── Criar node_mapping.json inicial (20 nodes comuns)          │
│  ├── Etapa 3.1-3.3: Classificador                               │
│  └── Etapa 4A: Tradução determinística                          │
│  ENTREGÁVEL: 70% dos nodes traduzidos sem IA                    │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  SPRINT 4 (Semana 5): INTEGRAÇÃO COM IA                         │
│  ├── Etapa 4B.1: Prompt builder                                 │
│  ├── Etapa 4B.2: Cliente Vertex AI                              │
│  └── Etapa 4B.3-4B.4: Validação + Candidatos                    │
│  ENTREGÁVEL: Nodes desconhecidos interpretados por IA           │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  SPRINT 5 (Semana 6): GERAÇÃO DE CÓDIGO                         │
│  ├── Etapa 5.1-5.4: Gerador Python completo                     │
│  └── Etapa 1.5 + 2.4: Metanodes e Loops                         │
│  ENTREGÁVEL: Código Python executável gerado                    │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  SPRINT 6 (Semana 7): VALIDAÇÃO E APRENDIZADO                   │
│  ├── Etapa 6.1-6.3: Validação cruzada                           │
│  └── Etapa 7.1-7.4: Ciclo de aprendizado                        │
│  ENTREGÁVEL: Sistema auto-evolutivo funcionando                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## ✅ CRITÉRIOS DE SUCESSO GLOBAIS

| Métrica | Meta | Como Medir |
|---------|------|------------|
| Cobertura determinística | ≥ 80% | nodes MAPPED / total |
| Precisão validação | ≥ 95% | schemas corretos / total |
| Redução chamadas IA | -70% em 30 dias | chamadas semana 4 / semana 1 |
| Tempo por workflow | < 60s | end-to-end processing |
| Taxa promoção candidatos | ≥ 80% | promoted / total candidates |

---

Deseja que eu inicie a implementação pela **Etapa 1.1 (Extrator ZIP)** ou prefere primeiro que eu crie o arquivo **node_mapping.json inicial** com os 20 nodes mais comuns?