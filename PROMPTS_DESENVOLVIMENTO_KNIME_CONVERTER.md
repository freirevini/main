# 📋 PROMPTS DE DESENVOLVIMENTO - KNIME TO PYTHON CONVERTER

## Guia de Prompts para Implementação Etapa por Etapa

**Ferramenta:** Windsurf com Claude Opus 4.5  
**Método:** Enviar um prompt por vez, validar resultado, prosseguir para próximo  
**Data:** 2025-12-15

---

## 🔴 PROMPT ZERO - EXTRAÇÃO DO PROJETO LEGADO

> **OBJETIVO:** Extrair conhecimento do código legado antes de começar o novo projeto.  
> **QUANDO USAR:** Execute este prompt PRIMEIRO, em um chat separado, apontando para o arquivo do projeto legado.

```
Analise o arquivo Python anexado que é um conversor KNIME para Python (versão legada).

Preciso que você extraia e organize as seguintes informações em formato JSON estruturado:

## 1. MAPEAMENTO DE NODES

Para cada tipo de node que o código processa, extraia:
- `factory`: classe Java completa (ex: org.knime.base.node.io.csvreader.CSVReaderNodeFactory)
- `name`: nome legível do node
- `category`: categoria (io, manipulation, transformation, logic, flow_control, database)
- `settings_extraction`: quais campos do settings.xml são lidos e como (xpath ou chave)
- `python_template`: código Python gerado para este node (template com placeholders)
- `imports`: bibliotecas necessárias

## 2. IDENTIFICAÇÃO DE METANODES

Extraia a lógica usada para:
- Como identificar se uma pasta é um metanode
- Como acessar o workflow.knime interno do metanode
- Como tratar variáveis expostas pelo metanode
- Padrões de nome que indicam metanodes especiais (ex: "CRIA DATA")

## 3. CONEXÕES ENTRE NODES

Extraia a lógica de:
- Como o código lê as conexões do workflow.knime
- Estrutura de dados usada para armazenar conexões (source_id, dest_id, ports)
- Como determinar a ordem de execução (algoritmo de ordenação topológica)
- Como tratar múltiplas entradas em um node (joins, concatenates)

## 4. CONEXÕES DE BANCO DE DADOS

Extraia informações sobre:
- Nodes de conexão suportados (Database Connector, BigQuery, etc.)
- Como extrair JDBC URLs e credenciais
- Como mapear queries SQL para pandas
- Tratamento de variáveis de fluxo em queries (${variable})

## 5. MAPEAMENTO DE TIPOS

Extraia o dicionário completo de:
- Tipos KNIME (cell classes) → tipos Python/Pandas
- Tratamento de tipos especiais (datas, coleções, blobs)

## 6. PADRÕES DE EXPRESSÕES

Extraia mapeamentos de:
- Funções matemáticas KNIME → NumPy (pow, sqrt, etc.)
- Sintaxe de referência a colunas ($coluna$)
- Operadores de comparação e lógicos

## FORMATO DE SAÍDA

Gere 3 arquivos JSON:

### node_mapping_extracted.json
{
  "version": "extracted_from_legacy",
  "extraction_date": "2025-12-15",
  "mappings": [
    {
      "factory": "...",
      "name": "...",
      "category": "...",
      "settings_extraction": {},
      "python_template": "...",
      "imports": []
    }
  ]
}

### knime_types_extracted.json
{
  "type_mappings": {
    "org.knime.core.data.def.DoubleCell": "Float64",
    ...
  },
  "expression_mappings": {
    "pow": "np.power",
    ...
  },
  "column_reference_pattern": "\\$([^$]+)\\$"
}

### connection_patterns_extracted.json
{
  "metanode_detection": {
    "indicators": ["..."],
    "special_patterns": ["CRIA DATA", ...]
  },
  "edge_extraction": {
    "xml_path": "...",
    "structure": {}
  },
  "database_nodes": [
    {
      "factory": "...",
      "connection_extraction": {}
    }
  ]
}

Seja exaustivo - extraia TODOS os nodes, tipos e padrões que encontrar no código.
Para cada item, inclua um comentário sobre onde no código legado está a implementação (número da linha aproximado ou nome da função).
```

---

## 🟢 SPRINT 1: FUNDAÇÃO (Extração)

### PROMPT 1.1 - Estrutura do Projeto e Extrator ZIP

```
Crie a estrutura inicial de um projeto Python para processamento de arquivos.

## ESTRUTURA DE DIRETÓRIOS

Crie a seguinte estrutura:

knime_converter/
├── config/
│   └── .gitkeep
├── output/
│   └── .gitkeep
├── src/
│   ├── __init__.py
│   ├── extractors/
│   │   ├── __init__.py
│   │   └── zip_extractor.py
│   └── utils/
│       ├── __init__.py
│       └── logger.py
├── tests/
│   ├── __init__.py
│   └── test_zip_extractor.py
├── main.py
├── requirements.txt
└── README.md

## ARQUIVO: src/utils/logger.py

Crie um módulo de logging com:
- Configuração de logging para console e arquivo
- Níveis: DEBUG, INFO, WARNING, ERROR
- Formato: timestamp | nível | módulo | mensagem
- Arquivo de log em: output/converter.log

## ARQUIVO: src/extractors/zip_extractor.py

Crie uma classe `KnimeZipExtractor` com:

```python
class KnimeZipExtractor:
    """
    Extrai conteúdo de arquivos .knwf (KNIME Workflow).
    Arquivos .knwf são ZIPs renomeados contendo a estrutura do workflow.
    """
    
    def __init__(self, file_path: str):
        """
        Inicializa o extrator.
        
        Args:
            file_path: Caminho para arquivo .knwf ou pasta já extraída
        """
        pass
    
    def extract(self, output_dir: str = None) -> Path:
        """
        Extrai o arquivo .knwf para uma pasta temporária.
        
        Args:
            output_dir: Diretório de destino (opcional, usa tempdir se None)
            
        Returns:
            Path para a pasta extraída
            
        Raises:
            FileNotFoundError: Se arquivo não existe
            zipfile.BadZipFile: Se não é um ZIP válido
        """
        pass
    
    def get_workflow_structure(self) -> dict:
        """
        Retorna a estrutura de arquivos do workflow.
        
        Returns:
            Dict com:
            - root_path: caminho raiz
            - workflow_file: caminho do workflow.knime
            - node_folders: lista de pastas de nodes
            - has_metanodes: bool indicando presença de metanodes
        """
        pass
    
    def cleanup(self):
        """Remove pasta temporária se foi criada."""
        pass
```

## REQUISITOS TÉCNICOS

1. Usar `pathlib.Path` para todos os caminhos
2. Usar `tempfile.mkdtemp()` para pasta temporária
3. Validar se o arquivo é um ZIP válido antes de extrair
4. Logar cada etapa do processo
5. Tratar arquivos .knwf e pastas já extraídas
6. Identificar pastas de nodes pelo padrão: nome contendo "(#N)" onde N é número

## ARQUIVO: tests/test_zip_extractor.py

Crie testes unitários com pytest:
- test_init_with_valid_file
- test_init_with_invalid_file
- test_init_with_folder
- test_extract_creates_folder
- test_get_workflow_structure_returns_dict
- test_cleanup_removes_temp_folder

## ARQUIVO: requirements.txt

```
pytest>=7.0.0
pathlib>=1.0.1
```

## ARQUIVO: README.md

Crie documentação básica com:
- Nome do projeto
- Descrição breve
- Como instalar dependências
- Como rodar testes

Implemente todos os arquivos com código completo e funcional.
Adicione docstrings detalhadas e comentários explicando a lógica.
Use type hints em todas as funções.
```

---

### PROMPT 1.2 - Parser do workflow.knime

```
Adicione ao projeto existente um parser para o arquivo workflow.knime.

## ARQUIVO: src/extractors/workflow_parser.py

Crie uma classe `WorkflowParser` que lê o arquivo XML workflow.knime e extrai nodes e conexões.

### CONTEXTO TÉCNICO DO XML

O arquivo workflow.knime usa namespace: "http://www.knime.org/2008/09/XMLConfig"

Estrutura relevante:
- Nodes estão em: <config key="nodes"><config key="node_N">...</config></config>
- Conexões estão em: <config key="connections"><config key="connection_N">...</config></config>

Cada node tem:
- <entry key="id" type="xint" value="N"/>
- <entry key="node_settings_file" type="xstring" value="Nome do Node (#N)/settings.xml"/>
- <entry key="node_is_meta" type="xboolean" value="true/false"/>

Cada conexão tem:
- <entry key="sourceID" type="xint" value="N"/>
- <entry key="destID" type="xint" value="N"/>
- <entry key="sourcePort" type="xint" value="N"/>
- <entry key="destPort" type="xint" value="N"/>

### IMPLEMENTAÇÃO

```python
from dataclasses import dataclass
from typing import List, Dict, Optional
from pathlib import Path
import xml.etree.ElementTree as ET

@dataclass
class NodeInfo:
    """Informações básicas de um node extraídas do workflow.knime."""
    id: str
    display_name: str
    settings_path: str
    is_metanode: bool
    position_x: int = 0
    position_y: int = 0

@dataclass
class ConnectionInfo:
    """Informação de uma conexão entre nodes."""
    source_id: str
    dest_id: str
    source_port: int
    dest_port: int
    connection_type: str = "data"  # "data" ou "flow"

class WorkflowParser:
    """
    Parser para arquivo workflow.knime (XML).
    Extrai lista de nodes e conexões do workflow.
    """
    
    NAMESPACE = {"k": "http://www.knime.org/2008/09/XMLConfig"}
    
    def __init__(self, workflow_path: Path):
        """
        Args:
            workflow_path: Caminho para arquivo workflow.knime
        """
        pass
    
    def parse(self) -> Dict:
        """
        Faz o parsing completo do workflow.
        
        Returns:
            Dict com:
            - metadata: informações do workflow (autor, versão, etc)
            - nodes: Dict[str, NodeInfo] mapeando id -> NodeInfo
            - connections: List[ConnectionInfo]
            - annotations: List[str] (textos de anotações no workflow)
        """
        pass
    
    def _parse_with_namespace(self, root: ET.Element) -> Dict:
        """Tenta parsing com namespace KNIME."""
        pass
    
    def _parse_without_namespace(self, root: ET.Element) -> Dict:
        """Fallback: parsing sem namespace (versões antigas)."""
        pass
    
    def _extract_nodes(self, nodes_config: ET.Element) -> Dict[str, NodeInfo]:
        """Extrai todos os nodes do elemento <config key="nodes">."""
        pass
    
    def _extract_connections(self, connections_config: ET.Element) -> List[ConnectionInfo]:
        """Extrai todas as conexões do elemento <config key="connections">."""
        pass
    
    def _extract_metadata(self, root: ET.Element) -> Dict:
        """Extrai metadados: autor, versão, credenciais."""
        pass
    
    def get_node_by_id(self, node_id: str) -> Optional[NodeInfo]:
        """Retorna node pelo ID."""
        pass
    
    def get_predecessors(self, node_id: str) -> List[str]:
        """Retorna IDs dos nodes que enviam dados para este node."""
        pass
    
    def get_successors(self, node_id: str) -> List[str]:
        """Retorna IDs dos nodes que recebem dados deste node."""
        pass
```

### TRATAMENTO DE ERROS

1. Se arquivo não existe: raise FileNotFoundError com mensagem clara
2. Se XML inválido: raise ValueError("workflow.knime inválido: {detalhes}")
3. Se namespace não reconhecido: tentar sem namespace (fallback)
4. Logar warnings para elementos não reconhecidos

### ARQUIVO: tests/test_workflow_parser.py

Crie testes com um XML de exemplo inline:
- test_parse_returns_dict_with_required_keys
- test_extract_nodes_returns_node_info
- test_extract_connections_returns_connection_info
- test_get_predecessors_returns_correct_ids
- test_get_successors_returns_correct_ids
- test_parse_without_namespace_fallback

### XML DE TESTE (use como fixture)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<config xmlns="http://www.knime.org/2008/09/XMLConfig" key="workflow.knime">
    <entry key="created_by" type="xstring" value="4.5.2"/>
    <config key="nodes">
        <config key="node_1">
            <entry key="id" type="xint" value="1"/>
            <entry key="node_settings_file" type="xstring" value="CSV Reader (#1)/settings.xml"/>
            <entry key="node_is_meta" type="xboolean" value="false"/>
        </config>
        <config key="node_2">
            <entry key="id" type="xint" value="2"/>
            <entry key="node_settings_file" type="xstring" value="Column Filter (#2)/settings.xml"/>
            <entry key="node_is_meta" type="xboolean" value="false"/>
        </config>
    </config>
    <config key="connections">
        <config key="connection_0">
            <entry key="sourceID" type="xint" value="1"/>
            <entry key="destID" type="xint" value="2"/>
            <entry key="sourcePort" type="xint" value="1"/>
            <entry key="destPort" type="xint" value="1"/>
        </config>
    </config>
</config>
```

Implemente código completo, funcional, com todos os métodos implementados.
```

---

### PROMPT 1.3 - Parser do settings.xml

```
Adicione ao projeto existente um parser para arquivos settings.xml dos nodes.

## ARQUIVO: src/extractors/settings_parser.py

Crie uma classe `SettingsParser` que extrai configurações de cada node.

### CONTEXTO TÉCNICO

Cada node tem uma pasta com settings.xml contendo:
- `factory`: classe Java que identifica o tipo de node
- `node-name`: nome legível do node
- `state`: EXECUTED, IDLE, CONFIGURED
- `model`: configurações específicas do node (varia por tipo)
- `nodeAnnotation`: anotação/comentário do usuário

### IMPLEMENTAÇÃO

```python
from dataclasses import dataclass, field
from typing import Dict, Any, Optional, List
from pathlib import Path
import xml.etree.ElementTree as ET

@dataclass
class NodeSettings:
    """Configurações extraídas do settings.xml de um node."""
    factory: str
    node_name: str
    state: str
    annotation: str = ""
    model: Dict[str, Any] = field(default_factory=dict)
    
    # Campos calculados
    node_type: str = ""  # Categoria inferida do factory
    is_loop_start: bool = False
    is_loop_end: bool = False
    is_variable_node: bool = False

class SettingsParser:
    """
    Parser para arquivos settings.xml de nodes KNIME.
    Extrai factory, configurações e modelo do node.
    """
    
    NAMESPACE = {"k": "http://www.knime.org/2008/09/XMLConfig"}
    
    # Padrões para identificar tipos especiais de nodes
    LOOP_START_PATTERNS = ["LoopStart", "GroupLoopStart", "ChunkLoopStart"]
    LOOP_END_PATTERNS = ["LoopEnd", "VariableLoopEnd"]
    VARIABLE_NODE_PATTERNS = ["Variable", "FlowVariable", "TableRow"]
    
    def __init__(self, settings_path: Path):
        """
        Args:
            settings_path: Caminho para arquivo settings.xml
        """
        pass
    
    def parse(self) -> NodeSettings:
        """
        Faz parsing do settings.xml.
        
        Returns:
            NodeSettings com todas as informações extraídas
        """
        pass
    
    def _extract_factory(self, root: ET.Element) -> str:
        """Extrai a classe factory do node."""
        pass
    
    def _extract_node_name(self, root: ET.Element) -> str:
        """Extrai o nome legível do node."""
        pass
    
    def _extract_state(self, root: ET.Element) -> str:
        """Extrai o estado de execução."""
        pass
    
    def _extract_annotation(self, root: ET.Element) -> str:
        """Extrai a anotação/comentário do node."""
        pass
    
    def _extract_model(self, root: ET.Element) -> Dict[str, Any]:
        """
        Extrai o modelo/configurações do node.
        
        O modelo é específico de cada tipo de node.
        Esta função extrai recursivamente todos os valores.
        """
        pass
    
    def _parse_config_recursive(self, element: ET.Element) -> Dict[str, Any]:
        """
        Parse recursivo de elementos <config> e <entry>.
        
        Converte:
        - <entry type="xstring" value="X"> -> str
        - <entry type="xint" value="N"> -> int
        - <entry type="xdouble" value="N.N"> -> float
        - <entry type="xboolean" value="true"> -> bool
        - <config> com filhos -> dict recursivo
        - <entry> com array-size -> list
        """
        pass
    
    def _infer_node_type(self, factory: str) -> str:
        """
        Infere categoria do node pelo factory.
        
        Categorias: io, manipulation, transformation, logic, 
                    flow_control, database, other
        """
        pass
    
    def _check_special_patterns(self, factory: str, node_name: str) -> Dict[str, bool]:
        """Verifica se é loop start/end ou variable node."""
        pass
    
    def get_model_value(self, path: str, default: Any = None) -> Any:
        """
        Obtém valor do modelo por caminho.
        
        Exemplo: get_model_value("column-filter/included_names")
        
        Args:
            path: Caminho separado por "/" 
            default: Valor se não encontrar
        """
        pass
```

### TRATAMENTO DE TIPOS XML

O KNIME usa tipos específicos no atributo `type`:
- `xstring` → str
- `xint` → int
- `xlong` → int
- `xdouble` → float
- `xboolean` → bool (converte "true"/"false")
- `xchar` → str (único caractere)

### ARQUIVO: tests/test_settings_parser.py

Crie testes com XMLs de exemplo para diferentes tipos de nodes:

1. CSV Reader (io)
2. Column Filter (manipulation)  
3. Math Formula (transformation)
4. Rule Engine (logic)
5. Group Loop Start (flow_control)

Para cada um, teste:
- Extração do factory
- Extração do model com valores corretos
- Inferência correta do node_type
- Detecção de patterns especiais (loop, variable)

### XML DE TESTE - CSV Reader

```xml
<?xml version="1.0" encoding="UTF-8"?>
<config xmlns="http://www.knime.org/2008/09/XMLConfig" key="settings.xml">
    <entry key="factory" type="xstring" value="org.knime.base.node.io.csvreader.CSVReaderNodeFactory"/>
    <entry key="node-name" type="xstring" value="CSV Reader"/>
    <entry key="state" type="xstring" value="EXECUTED"/>
    <config key="model">
        <entry key="url" type="xstring" value="/data/input.csv"/>
        <entry key="colDelimiter" type="xstring" value=","/>
        <entry key="hasColHeader" type="xboolean" value="true"/>
        <entry key="hasRowHeader" type="xboolean" value="false"/>
    </config>
    <config key="nodeAnnotation">
        <entry key="text" type="xstring" value="Lê arquivo de contratos"/>
    </config>
</config>
```

### XML DE TESTE - Column Filter

```xml
<?xml version="1.0" encoding="UTF-8"?>
<config xmlns="http://www.knime.org/2008/09/XMLConfig" key="settings.xml">
    <entry key="factory" type="xstring" value="org.knime.base.node.preproc.filter.column.DataColumnSpecFilterNodeFactory"/>
    <entry key="node-name" type="xstring" value="Column Filter"/>
    <entry key="state" type="xstring" value="EXECUTED"/>
    <config key="model">
        <config key="column-filter">
            <entry key="filter-type" type="xstring" value="STANDARD"/>
            <config key="included_names">
                <entry key="array-size" type="xint" value="3"/>
                <entry key="0" type="xstring" value="NuContrato"/>
                <entry key="1" type="xstring" value="NmProduto"/>
                <entry key="2" type="xstring" value="VrContrato"/>
            </config>
            <config key="excluded_names">
                <entry key="array-size" type="xint" value="1"/>
                <entry key="0" type="xstring" value="TempColumn"/>
            </config>
        </config>
    </config>
</config>
```

Implemente código completo com todos os métodos funcionais.
```

---

### PROMPT 1.4 - Parser do spec.xml

```
Adicione ao projeto existente um parser para arquivos spec.xml (schema de saída dos nodes).

## ARQUIVO: src/extractors/spec_parser.py

Crie uma classe `SpecParser` que extrai o schema de saída de cada porta de um node.

### CONTEXTO TÉCNICO

Nodes executados têm pastas `port_N/` contendo:
- `spec.xml`: schema das colunas de saída
- `data.xml`: dados em si (opcional, pode ser binário)

O spec.xml contém:
- `number_columns`: quantidade de colunas
- `column_spec_N`: especificação de cada coluna
  - `column_name`: nome da coluna
  - `column_type/cell_class`: tipo KNIME da coluna
  - `column_domain`: domínio (min/max, valores possíveis)

### IMPLEMENTAÇÃO

```python
from dataclasses import dataclass, field
from typing import Dict, Any, Optional, List
from pathlib import Path
import xml.etree.ElementTree as ET

@dataclass
class ColumnSpec:
    """Especificação de uma coluna."""
    name: str
    knime_type: str
    python_type: str
    domain: Dict[str, Any] = field(default_factory=dict)
    
@dataclass  
class PortSpec:
    """Especificação de uma porta de saída."""
    port_index: int
    columns: List[ColumnSpec]
    row_count: Optional[int] = None
    
    @property
    def column_count(self) -> int:
        return len(self.columns)
    
    @property
    def column_names(self) -> List[str]:
        return [c.name for c in self.columns]

class SpecParser:
    """
    Parser para arquivos spec.xml de portas de saída KNIME.
    Extrai schema de colunas com nomes, tipos e domínios.
    """
    
    NAMESPACE = {"k": "http://www.knime.org/2008/09/XMLConfig"}
    
    # Mapeamento de tipos KNIME para Python/Pandas
    TYPE_MAPPING = {
        # Numéricos
        "org.knime.core.data.def.DoubleCell": "Float64",
        "org.knime.core.data.def.IntCell": "Int64",
        "org.knime.core.data.def.LongCell": "Int64",
        "org.knime.core.data.DoubleValue": "Float64",
        "org.knime.core.data.IntValue": "Int64",
        "org.knime.core.data.LongValue": "Int64",
        
        # String
        "org.knime.core.data.def.StringCell": "str",
        "org.knime.core.data.StringValue": "str",
        
        # Boolean
        "org.knime.core.data.def.BooleanCell": "bool",
        "org.knime.core.data.BooleanValue": "bool",
        
        # Data/Tempo
        "org.knime.core.data.date.DateAndTimeCell": "datetime64[ns]",
        "org.knime.core.data.date.DateAndTimeValue": "datetime64[ns]",
        "org.knime.core.data.time.localdate.LocalDateCell": "datetime64[ns]",
        "org.knime.core.data.v2.time.LocalDateTimeValueFactory": "datetime64[ns]",
        "org.knime.core.data.v2.time.LocalDateValueFactory": "datetime64[ns]",
        
        # Coleções e outros
        "org.knime.core.data.collection.ListCell": "object",
        "org.knime.core.data.collection.SetCell": "object",
        "org.knime.core.data.blob.BinaryObjectCell": "object",
        "org.knime.core.data.xml.XMLCell": "str",
    }
    
    def __init__(self, spec_path: Path):
        """
        Args:
            spec_path: Caminho para arquivo spec.xml
        """
        pass
    
    def parse(self) -> PortSpec:
        """
        Faz parsing do spec.xml.
        
        Returns:
            PortSpec com todas as colunas e seus tipos
        """
        pass
    
    def _extract_columns(self, root: ET.Element) -> List[ColumnSpec]:
        """Extrai especificação de todas as colunas."""
        pass
    
    def _extract_column_spec(self, column_config: ET.Element) -> ColumnSpec:
        """Extrai especificação de uma coluna."""
        pass
    
    def _extract_domain(self, domain_config: ET.Element) -> Dict[str, Any]:
        """
        Extrai domínio da coluna.
        
        Para numéricos: min, max
        Para strings: possible_values (se definido)
        """
        pass
    
    def _map_knime_type(self, knime_type: str) -> str:
        """
        Mapeia tipo KNIME para tipo Python/Pandas.
        
        Se tipo não conhecido, tenta inferir por padrões no nome.
        Fallback: "object"
        """
        pass
    
    @classmethod
    def parse_node_ports(cls, node_folder: Path) -> Dict[int, PortSpec]:
        """
        Parseia todos os specs de portas de um node.
        
        Args:
            node_folder: Pasta do node
            
        Returns:
            Dict mapeando port_index -> PortSpec
        """
        pass

class SchemaRegistry:
    """
    Registro central de schemas para todos os nodes do workflow.
    Permite consultar schema de entrada/saída de qualquer node.
    """
    
    def __init__(self):
        self._schemas: Dict[str, Dict[int, PortSpec]] = {}
    
    def register(self, node_id: str, port_index: int, spec: PortSpec):
        """Registra schema de uma porta de um node."""
        pass
    
    def get_output_schema(self, node_id: str, port_index: int = 0) -> Optional[PortSpec]:
        """Obtém schema de saída de um node."""
        pass
    
    def get_input_schema(self, node_id: str, connections: List, port_index: int = 0) -> Optional[PortSpec]:
        """
        Obtém schema de entrada de um node.
        Baseado no schema de saída do node predecessor.
        """
        pass
    
    def to_dict(self) -> Dict:
        """Exporta todos os schemas como dict."""
        pass
```

### ARQUIVO: tests/test_spec_parser.py

Crie testes com spec.xml de exemplo:
- test_parse_returns_port_spec
- test_extract_columns_correct_count
- test_map_knime_type_double
- test_map_knime_type_string
- test_map_knime_type_date
- test_map_knime_type_unknown_fallback
- test_extract_domain_numeric
- test_extract_domain_categorical
- test_parse_node_ports_multiple

### XML DE TESTE

```xml
<?xml version="1.0" encoding="UTF-8"?>
<config xmlns="http://www.knime.org/2008/09/XMLConfig" key="spec.xml">
    <entry key="spec_name" type="xstring" value="default"/>
    <entry key="number_columns" type="xint" value="3"/>
    <config key="column_spec_0">
        <entry key="column_name" type="xstring" value="NuContrato"/>
        <config key="column_type">
            <entry key="cell_class" type="xstring" value="org.knime.core.data.def.DoubleCell"/>
        </config>
        <config key="column_domain">
            <config key="lower_bound">
                <entry key="datacell" type="xstring" value="org.knime.core.data.def.DoubleCell"/>
                <config key="org.knime.core.data.def.DoubleCell">
                    <entry key="DoubleCell" type="xdouble" value="1000.0"/>
                </config>
            </config>
            <config key="upper_bound">
                <entry key="datacell" type="xstring" value="org.knime.core.data.def.DoubleCell"/>
                <config key="org.knime.core.data.def.DoubleCell">
                    <entry key="DoubleCell" type="xdouble" value="9999.0"/>
                </config>
            </config>
        </config>
    </config>
    <config key="column_spec_1">
        <entry key="column_name" type="xstring" value="NmProduto"/>
        <config key="column_type">
            <entry key="cell_class" type="xstring" value="org.knime.core.data.def.StringCell"/>
        </config>
        <config key="column_domain">
            <config key="possible_values">
                <entry key="array-size" type="xint" value="2"/>
                <config key="0">
                    <entry key="datacell" type="xstring" value="org.knime.core.data.def.StringCell"/>
                    <config key="org.knime.core.data.def.StringCell">
                        <entry key="StringCell" type="xstring" value="Veículos"/>
                    </config>
                </config>
                <config key="1">
                    <entry key="datacell" type="xstring" value="org.knime.core.data.def.StringCell"/>
                    <config key="org.knime.core.data.def.StringCell">
                        <entry key="StringCell" type="xstring" value="Imóveis"/>
                    </config>
                </config>
            </config>
        </config>
    </config>
    <config key="column_spec_2">
        <entry key="column_name" type="xstring" value="DtLiberacao"/>
        <config key="column_type">
            <entry key="cell_class" type="xstring" value="org.knime.core.data.time.localdate.LocalDateCell"/>
        </config>
        <config key="column_domain"/>
    </config>
</config>
```

Implemente código completo com todos os métodos funcionais.
```

---

### PROMPT 1.5 - Expansão de Metanodes

```
Adicione ao projeto existente suporte para expansão recursiva de metanodes.

## CONTEXTO

Metanodes são sub-workflows encapsulados. Cada metanode:
- É identificado por `node_is_meta="true"` no workflow.knime
- Tem sua própria pasta com workflow.knime interno
- Pode conter outros metanodes (recursivo)
- Expõe variáveis de fluxo para o workflow pai

## ARQUIVO: src/extractors/metanode_expander.py

```python
from dataclasses import dataclass, field
from typing import Dict, List, Optional, Any
from pathlib import Path

@dataclass
class MetanodeInfo:
    """Informações de um metanode."""
    id: str
    name: str
    folder: Path
    parent_workflow: Optional[str] = None
    internal_workflow: Optional[Dict] = None
    exposed_variables: List[Dict[str, str]] = field(default_factory=list)
    depth: int = 0  # Nível de aninhamento

class MetanodeExpander:
    """
    Expande metanodes recursivamente, parseando seus workflows internos.
    """
    
    # Padrões de metanodes especiais (data de referência, etc)
    SPECIAL_METANODE_PATTERNS = [
        "CRIA DATA",
        "Data Ref",
        "DataReferencia", 
        "Data de Referência"
    ]
    
    def __init__(self, workflow_root: Path, max_depth: int = 10):
        """
        Args:
            workflow_root: Pasta raiz do workflow principal
            max_depth: Profundidade máxima de recursão (proteção)
        """
        self.workflow_root = workflow_root
        self.max_depth = max_depth
        self._metanodes: Dict[str, MetanodeInfo] = {}
    
    def expand_all(self, nodes: Dict, connections: List) -> Dict[str, MetanodeInfo]:
        """
        Expande todos os metanodes encontrados.
        
        Args:
            nodes: Dict de nodes do workflow principal
            connections: Lista de conexões
            
        Returns:
            Dict mapeando node_id -> MetanodeInfo com workflow interno
        """
        pass
    
    def expand_metanode(self, node_id: str, node_info, depth: int = 0) -> MetanodeInfo:
        """
        Expande um metanode específico.
        
        Args:
            node_id: ID do metanode
            node_info: NodeInfo do metanode
            depth: Profundidade atual de recursão
            
        Returns:
            MetanodeInfo com workflow interno parseado
        """
        pass
    
    def _find_metanode_folder(self, node_info) -> Optional[Path]:
        """Encontra a pasta do metanode baseado no settings_path."""
        pass
    
    def _parse_internal_workflow(self, metanode_folder: Path) -> Dict:
        """
        Parseia o workflow interno do metanode.
        Usa os parsers existentes recursivamente.
        """
        pass
    
    def _extract_exposed_variables(self, metanode_folder: Path) -> List[Dict[str, str]]:
        """
        Extrai variáveis de fluxo expostas pelo metanode.
        
        Busca em:
        - workflow.knime -> flow_stack
        - Nodes internos do tipo Variable/FlowVariable
        """
        pass
    
    def _is_special_metanode(self, name: str) -> bool:
        """Verifica se é um metanode especial (data de referência, etc)."""
        pass
    
    def get_metanode(self, node_id: str) -> Optional[MetanodeInfo]:
        """Obtém metanode expandido por ID."""
        pass
    
    def get_all_nodes_flat(self) -> Dict[str, Any]:
        """
        Retorna todos os nodes (incluindo internos de metanodes) 
        em uma estrutura flat com IDs qualificados.
        
        Exemplo: metanode_10/node_5 para node 5 dentro do metanode 10
        """
        pass
    
    def to_dict(self) -> Dict:
        """Exporta todos os metanodes como dict serializável."""
        pass
```

## ARQUIVO: tests/test_metanode_expander.py

Crie testes:
- test_expand_all_finds_metanodes
- test_expand_metanode_parses_internal_workflow
- test_recursive_expansion_respects_max_depth
- test_extract_exposed_variables
- test_is_special_metanode_true
- test_is_special_metanode_false
- test_get_all_nodes_flat_includes_internal

## INTEGRAÇÃO

Atualize o `src/extractors/__init__.py` para exportar:
```python
from .zip_extractor import KnimeZipExtractor
from .workflow_parser import WorkflowParser, NodeInfo, ConnectionInfo
from .settings_parser import SettingsParser, NodeSettings
from .spec_parser import SpecParser, PortSpec, ColumnSpec, SchemaRegistry
from .metanode_expander import MetanodeExpander, MetanodeInfo
```

Implemente código completo. O MetanodeExpander deve usar os parsers existentes (WorkflowParser, SettingsParser, SpecParser) internamente.
```

---

## 🟡 SPRINT 2: REPRESENTAÇÃO INTERMEDIÁRIA

### PROMPT 2.1 - Construtor de Grafo

```
Crie o módulo de construção de grafo de execução do workflow.

## ARQUIVO: src/ir/__init__.py

Crie arquivo vazio para o módulo.

## ARQUIVO: src/ir/graph_builder.py

```python
from dataclasses import dataclass, field
from typing import Dict, List, Set, Optional, Any, Tuple
from pathlib import Path
import json

# Opcional: usar networkx se disponível, senão implementar próprio
try:
    import networkx as nx
    HAS_NETWORKX = True
except ImportError:
    HAS_NETWORKX = False

@dataclass
class IRNode:
    """Representação intermediária de um node."""
    id: str
    name: str
    display_name: str
    factory: str
    category: str
    
    # Configurações extraídas
    settings: Dict[str, Any] = field(default_factory=dict)
    annotation: str = ""
    
    # Estrutura
    is_metanode: bool = False
    is_loop_start: bool = False
    is_loop_end: bool = False
    is_variable_node: bool = False
    
    # Schema
    input_ports: List[Dict] = field(default_factory=list)
    output_ports: List[Dict] = field(default_factory=list)
    
    # Grafo
    predecessors: List[str] = field(default_factory=list)
    successors: List[str] = field(default_factory=list)
    
    # Processamento
    classification: str = "UNKNOWN"  # MAPPED, CANDIDATE, UNKNOWN
    python_code: str = ""
    
    # Metanode específico
    internal_workflow: Optional[Dict] = None

@dataclass
class IREdge:
    """Representação intermediária de uma conexão."""
    id: str
    source_node: str
    target_node: str
    source_port: int
    target_port: int
    edge_type: str = "data"  # "data" ou "flow"

@dataclass
class WorkflowIR:
    """Representação intermediária completa do workflow."""
    name: str
    source_file: str
    
    # Metadados
    metadata: Dict[str, Any] = field(default_factory=dict)
    
    # Estrutura
    nodes: Dict[str, IRNode] = field(default_factory=dict)
    edges: List[IREdge] = field(default_factory=list)
    
    # Calculados
    execution_order: List[str] = field(default_factory=list)
    loops: List[Dict] = field(default_factory=list)
    
    # Estatísticas
    statistics: Dict[str, Any] = field(default_factory=dict)

class GraphBuilder:
    """
    Constrói grafo de execução a partir dos dados extraídos.
    """
    
    def __init__(self):
        self._nodes: Dict[str, IRNode] = {}
        self._edges: List[IREdge] = []
        self._graph = None  # NetworkX DiGraph se disponível
    
    def build(self, 
              parsed_workflow: Dict,
              node_settings: Dict[str, Any],
              node_specs: Dict[str, Any],
              metanodes: Dict[str, Any]) -> WorkflowIR:
        """
        Constrói IR completo do workflow.
        
        Args:
            parsed_workflow: Resultado do WorkflowParser.parse()
            node_settings: Dict node_id -> NodeSettings parseados
            node_specs: Dict node_id -> PortSpec de cada node
            metanodes: Dict node_id -> MetanodeInfo expandidos
            
        Returns:
            WorkflowIR completo
        """
        pass
    
    def _create_ir_nodes(self, 
                         parsed_nodes: Dict,
                         settings: Dict,
                         specs: Dict,
                         metanodes: Dict) -> Dict[str, IRNode]:
        """Cria IRNode para cada node do workflow."""
        pass
    
    def _create_ir_edges(self, connections: List) -> List[IREdge]:
        """Cria IREdge para cada conexão."""
        pass
    
    def _build_adjacency(self):
        """Constrói listas de predecessors/successors em cada node."""
        pass
    
    def _build_networkx_graph(self) -> Any:
        """Se networkx disponível, cria DiGraph para análises."""
        pass
    
    def _calculate_statistics(self) -> Dict[str, Any]:
        """Calcula estatísticas do workflow."""
        pass
    
    def get_node(self, node_id: str) -> Optional[IRNode]:
        """Obtém node por ID."""
        return self._nodes.get(node_id)
    
    def get_predecessors(self, node_id: str) -> List[IRNode]:
        """Retorna nodes predecessores."""
        pass
    
    def get_successors(self, node_id: str) -> List[IRNode]:
        """Retorna nodes sucessores."""
        pass
    
    def get_roots(self) -> List[IRNode]:
        """Retorna nodes sem predecessores (início do fluxo)."""
        pass
    
    def get_leaves(self) -> List[IRNode]:
        """Retorna nodes sem sucessores (fim do fluxo)."""
        pass
    
    def to_dict(self) -> Dict:
        """Exporta grafo como dict serializável."""
        pass
```

## ARQUIVO: tests/test_graph_builder.py

Crie testes:
- test_build_creates_workflow_ir
- test_create_ir_nodes_maps_all_nodes
- test_create_ir_edges_maps_all_connections
- test_build_adjacency_correct_predecessors
- test_build_adjacency_correct_successors
- test_get_roots_returns_source_nodes
- test_get_leaves_returns_sink_nodes
- test_to_dict_is_serializable

## ATUALIZAÇÃO: requirements.txt

Adicione:
```
networkx>=3.0  # Opcional mas recomendado
```

Implemente código completo. Se networkx não estiver disponível, implemente as funcionalidades básicas sem ele.
```

---

### PROMPT 2.2 - Ordenação Topológica e Detecção de Loops

```
Adicione ao projeto ordenação topológica e detecção de loops.

## ARQUIVO: src/ir/topological_sort.py

```python
from typing import Dict, List, Set, Tuple, Optional
from collections import deque
from dataclasses import dataclass

@dataclass
class LoopInfo:
    """Informações sobre um loop detectado."""
    id: str
    start_node: str
    end_node: str
    internal_nodes: List[str]
    loop_type: str  # "GroupLoop", "ChunkLoop", "GenericLoop"
    loop_variables: List[Dict[str, str]]

class TopologicalSorter:
    """
    Calcula ordem topológica de execução do workflow.
    Detecta e trata loops corretamente.
    """
    
    def __init__(self, nodes: Dict, edges: List):
        """
        Args:
            nodes: Dict node_id -> IRNode
            edges: Lista de IREdge
        """
        self.nodes = nodes
        self.edges = edges
        self._loops: List[LoopInfo] = []
        self._execution_order: List[str] = []
    
    def sort(self) -> Tuple[List[str], List[LoopInfo]]:
        """
        Calcula ordem topológica e detecta loops.
        
        Returns:
            Tuple de:
            - Lista de node_ids na ordem de execução
            - Lista de LoopInfo para loops detectados
            
        Raises:
            ValueError: Se houver ciclo não tratado
        """
        pass
    
    def _detect_loops(self) -> List[LoopInfo]:
        """
        Detecta loops baseado em padrões de nodes.
        
        Procura pares de LoopStart/LoopEnd e identifica nodes internos.
        """
        pass
    
    def _find_loop_pairs(self) -> List[Tuple[str, str]]:
        """Encontra pares (start_id, end_id) de loops."""
        pass
    
    def _find_internal_nodes(self, start_id: str, end_id: str) -> List[str]:
        """
        Encontra nodes dentro de um loop.
        
        Nodes internos são aqueles alcançáveis a partir do start
        e que alcançam o end, sem sair do loop.
        """
        pass
    
    def _extract_loop_variables(self, start_id: str) -> List[Dict[str, str]]:
        """Extrai variáveis de loop do node start."""
        pass
    
    def _kahn_sort(self, 
                   nodes: Set[str], 
                   exclude_back_edges: Set[Tuple[str, str]] = None) -> List[str]:
        """
        Algoritmo de Kahn para ordenação topológica.
        
        Args:
            nodes: Conjunto de node_ids a ordenar
            exclude_back_edges: Arestas de retorno de loops a ignorar
            
        Returns:
            Lista ordenada de node_ids
        """
        pass
    
    def _build_indegree(self, 
                        nodes: Set[str],
                        exclude_edges: Set[Tuple[str, str]] = None) -> Dict[str, int]:
        """Calcula in-degree de cada node."""
        pass
    
    def _build_adjacency_list(self,
                              nodes: Set[str],
                              exclude_edges: Set[Tuple[str, str]] = None) -> Dict[str, List[str]]:
        """Constrói lista de adjacência."""
        pass
    
    def _handle_isolated_nodes(self, 
                               ordered: List[str], 
                               all_nodes: Set[str]) -> List[str]:
        """Adiciona nodes isolados (sem conexões) ao final."""
        pass
    
    def get_execution_order(self) -> List[str]:
        """Retorna ordem de execução calculada."""
        return self._execution_order
    
    def get_loops(self) -> List[LoopInfo]:
        """Retorna loops detectados."""
        return self._loops
    
    def validate_order(self) -> bool:
        """
        Valida se a ordem está correta.
        
        Para cada node, todos os predecessores devem vir antes.
        """
        pass
```

## ARQUIVO: tests/test_topological_sort.py

Crie testes:
- test_sort_simple_linear_workflow
- test_sort_workflow_with_branches
- test_sort_workflow_with_merge
- test_detect_loops_finds_group_loop
- test_find_internal_nodes_correct
- test_kahn_sort_respects_dependencies
- test_handle_isolated_nodes
- test_validate_order_true_for_valid
- test_validate_order_false_for_invalid

## CENÁRIOS DE TESTE

### Workflow Linear
```
1 -> 2 -> 3 -> 4
Ordem esperada: [1, 2, 3, 4]
```

### Workflow com Branch
```
    -> 2 ->
1 ->       -> 4
    -> 3 ->
Ordem esperada: [1, 2, 3, 4] ou [1, 3, 2, 4]
```

### Workflow com Loop
```
1 -> 2(LoopStart) -> 3 -> 4(LoopEnd) -> 5
                     ^       |
                     +-------+

Loops: [{start: 2, end: 4, internal: [3]}]
Ordem: [1, 2, 3, 4, 5]
```

Implemente código completo.
```

---

### PROMPT 2.3 - Exportador de IR para JSON

```
Adicione ao projeto o exportador de IR para JSON.

## ARQUIVO: src/ir/ir_exporter.py

```python
from typing import Dict, Any, Optional
from pathlib import Path
from datetime import datetime
import json

class IRExporter:
    """
    Exporta WorkflowIR para arquivo JSON estruturado.
    """
    
    def __init__(self, workflow_ir):
        """
        Args:
            workflow_ir: WorkflowIR a exportar
        """
        self.workflow_ir = workflow_ir
    
    def export(self, output_path: Path) -> Path:
        """
        Exporta IR para arquivo JSON.
        
        Args:
            output_path: Caminho do arquivo de saída
            
        Returns:
            Path do arquivo criado
        """
        pass
    
    def to_dict(self) -> Dict[str, Any]:
        """
        Converte IR para dicionário serializável.
        
        Estrutura:
        {
            "metadata": {...},
            "graph": {
                "nodes": {...},
                "edges": [...],
                "execution_order": [...],
                "loops": [...],
                "metanodes": {...}
            },
            "statistics": {...}
        }
        """
        pass
    
    def _serialize_metadata(self) -> Dict[str, Any]:
        """Serializa metadados do workflow."""
        pass
    
    def _serialize_nodes(self) -> Dict[str, Dict]:
        """Serializa todos os nodes."""
        pass
    
    def _serialize_node(self, node) -> Dict[str, Any]:
        """
        Serializa um IRNode.
        
        Inclui:
        - Identificação (id, name, factory)
        - Configurações (settings, annotation)
        - Estrutura (is_metanode, is_loop_*)
        - Schema (input_ports, output_ports)
        - Grafo (predecessors, successors)
        - Processamento (classification, python_code)
        """
        pass
    
    def _serialize_edges(self) -> List[Dict]:
        """Serializa todas as arestas."""
        pass
    
    def _serialize_loops(self) -> List[Dict]:
        """Serializa informações de loops."""
        pass
    
    def _serialize_metanodes(self) -> Dict[str, Dict]:
        """Serializa metanodes com seus workflows internos."""
        pass
    
    def _calculate_statistics(self) -> Dict[str, Any]:
        """
        Calcula estatísticas do IR:
        - total_nodes
        - total_edges
        - total_metanodes
        - total_loops
        - nodes_by_category
        - nodes_by_classification
        """
        pass
    
    @staticmethod
    def load(input_path: Path) -> Dict[str, Any]:
        """
        Carrega IR de arquivo JSON.
        
        Args:
            input_path: Caminho do arquivo
            
        Returns:
            Dict com IR carregado
        """
        pass
    
    @staticmethod
    def validate_ir(ir_dict: Dict) -> bool:
        """
        Valida estrutura do IR.
        
        Verifica:
        - Campos obrigatórios presentes
        - Tipos corretos
        - Referências válidas (edges apontam para nodes existentes)
        """
        pass

class IRSummaryGenerator:
    """
    Gera resumo legível do IR para logs e debugging.
    """
    
    def __init__(self, ir_dict: Dict):
        self.ir = ir_dict
    
    def generate_summary(self) -> str:
        """
        Gera resumo textual do workflow.
        
        Formato:
        ```
        WORKFLOW: nome_do_workflow
        Fonte: arquivo.knwf
        
        ESTATÍSTICAS:
        - Nodes: 25 (18 MAPPED, 5 CANDIDATE, 2 UNKNOWN)
        - Edges: 24
        - Loops: 1
        - Metanodes: 3
        
        ORDEM DE EXECUÇÃO:
        1. CSV Reader (#1) [MAPPED]
        2. Column Filter (#2) [MAPPED]
        ...
        
        LOOPS:
        - Loop 1: nodes 5-8 (GroupLoop)
        
        NODES NÃO MAPEADOS:
        - Java Snippet (#22) [UNKNOWN]
        ```
        """
        pass
    
    def generate_node_table(self) -> str:
        """Gera tabela de nodes formatada."""
        pass
    
    def generate_coverage_report(self) -> str:
        """Gera relatório de cobertura."""
        pass
```

## ARQUIVO: tests/test_ir_exporter.py

Crie testes:
- test_export_creates_file
- test_to_dict_has_required_keys
- test_serialize_node_complete
- test_serialize_edges_correct_format
- test_load_returns_same_structure
- test_validate_ir_true_for_valid
- test_validate_ir_false_for_missing_fields
- test_summary_generator_readable_output

## ATUALIZAÇÃO: src/ir/__init__.py

```python
from .graph_builder import GraphBuilder, IRNode, IREdge, WorkflowIR
from .topological_sort import TopologicalSorter, LoopInfo
from .ir_exporter import IRExporter, IRSummaryGenerator
```

Implemente código completo com encoding UTF-8 e formatação JSON indentada.
```

---

## 🔶 CONTINUA NOS PRÓXIMOS SPRINTS...

> **NOTA:** Os prompts dos Sprints 3 a 6 seguem o mesmo padrão detalhado.
> Por questões de tamanho, veja o documento completo KNIME_TO_PYTHON_CONVERTER_ARCHITECTURE.md para a especificação completa de cada módulo.

---

## 📝 NOTAS DE USO

### Ordem de Execução dos Prompts

1. **PROMPT ZERO** (em chat separado) - Extrair conhecimento do código legado
2. Criar novo projeto e enviar prompts na ordem:
   - Sprint 1: 1.1 → 1.2 → 1.3 → 1.4 → 1.5
   - Sprint 2: 2.1 → 2.2 → 2.3
   - Sprint 3: 3.1 → 3.2 → 3.3
   - Sprint 4: 4.1 → 4.2 → 4.3
   - Sprint 5: 5.1
   - Sprint 6: 6.1 → 6.2
   - Final: Orquestrador

### Validação entre Etapas

Após cada prompt, execute:
```bash
# Verifica sintaxe
python -m py_compile src/modulo/arquivo.py

# Executa testes
pytest tests/test_arquivo.py -v
```

### Arquivos de Configuração do Projeto Legado

Após executar o PROMPT ZERO, copie os JSONs gerados para a pasta `config/` do novo projeto:
- `node_mapping_extracted.json` → merge com `node_mapping.json`
- `knime_types_extracted.json` → `knime_types.json`
- `connection_patterns_extracted.json` → referência para implementação

---

**Documento gerado em:** 2025-12-15  
**Versão:** 1.0  
**Objetivo:** Guiar implementação no Windsurf com Claude Opus 4.5
