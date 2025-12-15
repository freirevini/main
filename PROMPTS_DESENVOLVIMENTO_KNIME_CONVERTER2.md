# 📋 PROMPTS DE DESENVOLVIMENTO - KNIME TO PYTHON CONVERTER

## Guia de Prompts para Implementação no Windsurf (Claude Opus 4.5)

**Instruções de Uso:**
1. Envie cada prompt separadamente
2. Valide o funcionamento antes de enviar o próximo
3. Os prompts são independentes e focados em cada etapa
4. Não pule etapas - cada uma depende da anterior

---

# 🔧 PROMPT 0: EXTRAÇÃO DE CONHECIMENTO DO PROJETO LEGADO

> **QUANDO USAR:** Execute este prompt PRIMEIRO em um chat separado, apontando para o arquivo `knime_to_python_converter_banking.txt`. O objetivo é extrair o conhecimento já implementado e gerar arquivos de configuração reutilizáveis.

```
Você é um Engenheiro de Software especializado em análise de código legado e extração de conhecimento.

TAREFA:
Analise o arquivo Python fornecido (knime_to_python_converter_banking.txt) e extraia TODAS as informações de mapeamento existentes para gerar arquivos JSON estruturados.

CONTEXTO:
Este é um conversor KNIME → Python legado que funciona parcialmente. Preciso extrair o conhecimento já codificado para reutilizar em um novo projeto com arquitetura melhorada.

EXTRAÇÕES NECESSÁRIAS:

1. **node_mapping_extracted.json** - Extraia TODOS os mapeamentos de nodes encontrados no código:
   - Procure por dicionários, switch/case, if/elif que mapeiam factory classes para código Python
   - Para cada node encontrado, extraia:
     - factory (classe Java completa, ex: "org.knime.base.node.io.csvreader.CSVReaderNodeFactory")
     - name (nome legível do node)
     - category (io, manipulation, transformation, logic, flow_control, database)
     - python_template (código Python que gera)
     - settings_extraction (caminhos XML para extrair parâmetros)
     - imports (bibliotecas necessárias)

2. **type_mappings_extracted.json** - Extraia o mapeamento de tipos:
   - Tipos KNIME (cell classes) → tipos Python/Pandas
   - Ex: "org.knime.core.data.def.DoubleCell" → "Float64"

3. **metanode_patterns_extracted.json** - Extraia padrões de identificação de metanodes:
   - Como o código identifica se uma pasta é metanode
   - Padrões de nomes de metanodes especiais (ex: "CRIA DATA")
   - Variáveis de fluxo expostas por metanodes

4. **connection_patterns_extracted.json** - Extraia lógica de conexões:
   - Como o código lê conexões do workflow.knime
   - Como identifica source/target de cada edge
   - Como diferencia conexões de dados vs conexões de fluxo

5. **database_patterns_extracted.json** - Extraia padrões de banco de dados:
   - Mapeamento de JDBC URLs para tipos de banco (BigQuery, MySQL, Sybase, Oracle, SAP IQ)
   - Templates de conexão para cada banco
   - Padrões de extração de queries SQL dos nodes

FORMATO DE SAÍDA:
Gere cada arquivo JSON separadamente, com estrutura limpa e documentada.

Para node_mapping_extracted.json, use este formato:
```json
{
  "extracted_from": "knime_to_python_converter_banking.txt",
  "extraction_date": "2025-12-15",
  "nodes": [
    {
      "factory": "org.knime.base.node...",
      "name": "Nome do Node",
      "category": "categoria",
      "complexity": "simple|medium|high",
      "python_template": "código Python com {placeholders}",
      "settings_extraction": {
        "param_name": "caminho/no/xml"
      },
      "imports": ["import necessário"],
      "notes": "observações sobre este mapeamento"
    }
  ]
}
```

INSTRUÇÕES ADICIONAIS:
1. Seja EXAUSTIVO - extraia TODOS os nodes mapeados, mesmo os comentados
2. Preserve a lógica original - se há condições especiais, documente
3. Identifique nodes que usam Regex vs XML parsing
4. Marque nodes que têm tratamento especial (loops, metanodes, database)
5. Se encontrar código de validação, extraia também

Comece analisando o arquivo e gerando os JSONs um por um.
```

---

# 🏗️ SPRINT 1: FUNDAÇÃO (EXTRAÇÃO)

## PROMPT 1.1: Estrutura do Projeto e Extrator ZIP

```
Você é um Engenheiro de Software Python criando um módulo de extração de arquivos.

TAREFA:
Criar a estrutura base do projeto e o módulo de extração de arquivos KNIME (.knwf).

ESTRUTURA DE PASTAS A CRIAR:
```
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
└── requirements.txt
```

IMPLEMENTAÇÃO DETALHADA:

1. **requirements.txt** - Dependências iniciais:
   - Apenas bibliotecas nativas por enquanto (zipfile, pathlib, logging)

2. **src/utils/logger.py** - Sistema de logging:
   - Função `setup_logger(name, log_file=None)` que retorna logger configurado
   - Formato: `[TIMESTAMP] [LEVEL] [MODULE] - MESSAGE`
   - Suporte a console + arquivo opcional
   - Níveis: DEBUG para desenvolvimento, INFO para produção

3. **src/extractors/zip_extractor.py** - Classe ZipExtractor:

```python
class ZipExtractor:
    """
    Extrai conteúdo de arquivos KNIME (.knwf).
    
    Arquivos .knwf são ZIPs renomeados contendo:
    - workflow.knime (XML principal)
    - Pastas de nodes (Node 1/, Node 2/, etc.)
    - Cada pasta de node contém settings.xml e opcionalmente port_N/spec.xml
    """
    
    def __init__(self, knwf_path: str, output_dir: str = None):
        """
        Args:
            knwf_path: Caminho para arquivo .knwf
            output_dir: Diretório de saída (default: temp dir)
        """
        pass
    
    def validate_knwf(self) -> bool:
        """
        Valida se o arquivo é um KNWF válido.
        
        Verificações:
        1. Arquivo existe
        2. É um ZIP válido
        3. Contém workflow.knime na raiz
        
        Returns:
            True se válido, False caso contrário
        """
        pass
    
    def extract(self) -> Path:
        """
        Extrai o conteúdo do KNWF.
        
        Returns:
            Path do diretório extraído
            
        Raises:
            ValueError: Se arquivo inválido
            IOError: Se erro na extração
        """
        pass
    
    def list_node_folders(self) -> List[Dict]:
        """
        Lista todas as pastas de nodes encontradas.
        
        Returns:
            Lista de dicts com:
            - folder_name: Nome da pasta
            - folder_path: Path completo
            - has_settings: Se contém settings.xml
            - has_spec: Se contém algum spec.xml
            - is_metanode: Se parece ser metanode (contém workflow.knime)
        """
        pass
    
    def get_workflow_xml_path(self) -> Path:
        """Retorna path do workflow.knime principal."""
        pass
    
    def cleanup(self):
        """Remove diretório temporário se criado automaticamente."""
        pass
```

4. **tests/test_zip_extractor.py** - Testes unitários:
   - test_validate_knwf_valid_file
   - test_validate_knwf_invalid_file
   - test_extract_success
   - test_list_node_folders
   - Usar mocks para não depender de arquivo real

5. **main.py** - Script principal básico:
   - Argparse para receber caminho do arquivo
   - Chamar ZipExtractor
   - Printar lista de nodes encontrados

COMENTÁRIOS OBRIGATÓRIOS:
- Cada método deve ter docstring completa
- Comentários inline explicando lógica não óbvia
- Type hints em todos os parâmetros e retornos

TRATAMENTO DE ERROS:
- Validar inputs antes de processar
- Mensagens de erro claras e acionáveis
- Logging em pontos críticos

Gere todos os arquivos completos, prontos para execução.
```

---

## PROMPT 1.2: Parser de workflow.knime

```
Você é um Engenheiro de Software Python criando um parser XML especializado.

TAREFA:
Criar o módulo que parseia o arquivo workflow.knime e extrai nodes e conexões.

CONTEXTO:
O workflow.knime é um XML com namespace "http://www.knime.org/2008/09/XMLConfig".
Ele contém a estrutura do workflow: lista de nodes e suas conexões.

ARQUIVO A CRIAR: src/extractors/workflow_parser.py

ESTRUTURA DO XML workflow.knime:
```xml
<config xmlns="http://www.knime.org/2008/09/XMLConfig" key="workflow.knime">
    <entry key="name" type="xstring" value="Nome do Workflow"/>
    <config key="nodes">
        <config key="node_1">
            <entry key="id" type="xint" value="1"/>
            <entry key="node_settings_file" type="xstring" value="Node Name (#1)/settings.xml"/>
            <entry key="node_is_meta" type="xboolean" value="false"/>
            <entry key="node_type" type="xstring" value="NativeNode"/>
            <config key="ui_settings">
                <config key="extrainfo.node.name">
                    <entry key="name" type="xstring" value="CSV Reader"/>
                </config>
            </config>
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

IMPLEMENTAÇÃO DETALHADA:

```python
from dataclasses import dataclass, field
from typing import List, Dict, Optional
from pathlib import Path
import xml.etree.ElementTree as ET

# Namespace KNIME para parsing XML
KNIME_NS = {"k": "http://www.knime.org/2008/09/XMLConfig"}


@dataclass
class KnimeNode:
    """Representa um node do workflow KNIME."""
    id: str
    name: str
    display_name: str
    settings_file: str
    is_metanode: bool
    node_type: str  # NativeNode, MetaNode, SubNode
    position: Dict[str, int] = field(default_factory=dict)
    
    @property
    def folder_name(self) -> str:
        """Retorna nome da pasta do node (ex: 'CSV Reader (#1)')."""
        pass


@dataclass  
class KnimeConnection:
    """Representa uma conexão entre nodes."""
    source_id: str
    dest_id: str
    source_port: int
    dest_port: int
    connection_type: str = "data"  # data ou flow


@dataclass
class WorkflowMetadata:
    """Metadados do workflow."""
    name: str
    knime_version: str
    author: str
    created_at: str
    last_modified_by: str
    last_modified_at: str


class WorkflowParser:
    """
    Parser para arquivos workflow.knime.
    
    Extrai:
    - Lista de nodes com seus metadados
    - Lista de conexões (edges do grafo)
    - Metadados do workflow
    """
    
    def __init__(self, workflow_path: Path):
        """
        Args:
            workflow_path: Caminho para workflow.knime
        """
        self.workflow_path = Path(workflow_path)
        self._tree = None
        self._root = None
        
    def parse(self) -> Dict:
        """
        Parseia o workflow.knime completo.
        
        Returns:
            Dict com:
            - metadata: WorkflowMetadata
            - nodes: List[KnimeNode]
            - connections: List[KnimeConnection]
            - annotations: List[Dict] (anotações visuais)
        """
        pass
    
    def _parse_with_namespace(self) -> ET.Element:
        """
        Tenta parsear com namespace KNIME.
        Fallback para parsing sem namespace se falhar.
        """
        pass
    
    def _extract_metadata(self) -> WorkflowMetadata:
        """Extrai metadados do cabeçalho do workflow."""
        pass
    
    def _extract_nodes(self) -> List[KnimeNode]:
        """
        Extrai todos os nodes do workflow.
        
        Lógica:
        1. Encontrar config key="nodes"
        2. Para cada config filho (node_N):
           - Extrair id, settings_file, is_meta, node_type
           - Extrair nome do ui_settings
           - Criar KnimeNode
        """
        pass
    
    def _extract_connections(self) -> List[KnimeConnection]:
        """
        Extrai todas as conexões do workflow.
        
        Lógica:
        1. Encontrar config key="connections"
        2. Para cada connection_N:
           - Extrair sourceID, destID, sourcePort, destPort
           - Identificar tipo (porta 0 geralmente é flow variable)
        """
        pass
    
    def _extract_annotations(self) -> List[Dict]:
        """Extrai anotações/comentários visuais do workflow."""
        pass
    
    def _find_entry(self, parent: ET.Element, key: str) -> Optional[str]:
        """
        Busca entry por key e retorna value.
        
        Tenta com namespace primeiro, depois sem.
        """
        pass
    
    def _find_config(self, parent: ET.Element, key: str) -> Optional[ET.Element]:
        """
        Busca config por key.
        
        Tenta com namespace primeiro, depois sem.
        """
        pass


def parse_workflow(workflow_path: Path) -> Dict:
    """Função de conveniência para parsear workflow."""
    parser = WorkflowParser(workflow_path)
    return parser.parse()
```

TESTES A CRIAR em tests/test_workflow_parser.py:
- test_parse_nodes_with_namespace
- test_parse_nodes_without_namespace (fallback)
- test_parse_connections
- test_extract_metadata
- test_identify_metanode
- test_node_folder_name_property

CRITÉRIOS DE VALIDAÇÃO:
1. Todos os nodes do XML devem ser extraídos
2. Todas as conexões devem ter source e dest válidos
3. Metanodes devem ser identificados corretamente (is_metanode=True)
4. Funcionar com e sem namespace XML

Gere o arquivo completo com todos os métodos implementados.
```

---

## PROMPT 1.3: Parser de settings.xml

```
Você é um Engenheiro de Software Python criando um parser de configurações.

TAREFA:
Criar o módulo que parseia arquivos settings.xml de cada node KNIME.

CONTEXTO:
Cada pasta de node contém um settings.xml com:
- factory: Classe Java que identifica o tipo de node
- node-name: Nome legível
- model: Configurações específicas (colunas, expressões, queries, etc.)
- state: EXECUTED, IDLE, CONFIGURED
- ports: Informações das portas de saída

ARQUIVO A CRIAR: src/extractors/settings_parser.py

ESTRUTURA DO XML settings.xml (exemplo Math Formula):
```xml
<config xmlns="http://www.knime.org/2008/09/XMLConfig" key="settings.xml">
    <entry key="factory" type="xstring" value="org.knime.ext.jep.JEPNodeFactory"/>
    <entry key="node-name" type="xstring" value="Math Formula"/>
    <entry key="state" type="xstring" value="EXECUTED"/>
    <config key="model">
        <entry key="expression" type="xstring" value="$Col1$ + $Col2$"/>
        <entry key="replaced_column" type="xstring" value="Result"/>
        <entry key="append_column" type="xboolean" value="false"/>
    </config>
    <config key="nodeAnnotation">
        <entry key="text" type="xstring" value="Soma colunas"/>
    </config>
    <config key="ports">
        <config key="port_1">
            <entry key="index" type="xint" value="1"/>
            <entry key="port_dir_location" type="xstring" value="port_1"/>
        </config>
    </config>
</config>
```

IMPLEMENTAÇÃO DETALHADA:

```python
from dataclasses import dataclass, field
from typing import Dict, List, Optional, Any
from pathlib import Path
import xml.etree.ElementTree as ET

KNIME_NS = {"k": "http://www.knime.org/2008/09/XMLConfig"}


@dataclass
class PortInfo:
    """Informação de uma porta do node."""
    index: int
    port_type: str  # data, flow_variable, database
    location: str   # Pasta onde está spec.xml


@dataclass
class NodeSettings:
    """Configurações extraídas de um node."""
    factory: str
    node_name: str
    state: str
    annotation: str
    model: Dict[str, Any]
    ports: List[PortInfo]
    flow_variables: Dict[str, Any]
    
    @property
    def is_executed(self) -> bool:
        return self.state == "EXECUTED"
    
    @property
    def category(self) -> str:
        """Infere categoria pelo factory."""
        factory_lower = self.factory.lower()
        if 'io' in factory_lower or 'reader' in factory_lower or 'writer' in factory_lower:
            return 'io'
        elif 'filter' in factory_lower or 'joiner' in factory_lower or 'sorter' in factory_lower:
            return 'manipulation'
        elif 'rule' in factory_lower or 'switch' in factory_lower:
            return 'logic'
        elif 'loop' in factory_lower:
            return 'flow_control'
        elif 'db' in factory_lower or 'database' in factory_lower:
            return 'database'
        else:
            return 'transformation'


class SettingsParser:
    """
    Parser para arquivos settings.xml de nodes KNIME.
    
    Responsabilidades:
    - Extrair identificação do node (factory, nome)
    - Extrair configurações do model (específicas por tipo)
    - Extrair informações de portas
    - Extrair variáveis de fluxo
    """
    
    def __init__(self, settings_path: Path):
        self.settings_path = Path(settings_path)
        self._tree = None
        self._root = None
    
    def parse(self) -> NodeSettings:
        """
        Parseia settings.xml e retorna NodeSettings.
        
        Returns:
            NodeSettings com todas informações extraídas
        """
        pass
    
    def _extract_factory(self) -> str:
        """Extrai a factory class do node."""
        pass
    
    def _extract_node_name(self) -> str:
        """Extrai o nome legível do node."""
        pass
    
    def _extract_state(self) -> str:
        """Extrai o estado do node (EXECUTED, IDLE, etc.)."""
        pass
    
    def _extract_annotation(self) -> str:
        """Extrai anotação/comentário do node."""
        pass
    
    def _extract_model(self) -> Dict[str, Any]:
        """
        Extrai configurações do model.
        
        O model contém configurações específicas de cada tipo de node:
        - CSV Reader: file path, delimiter, encoding
        - Column Filter: included/excluded columns
        - Rule Engine: lista de regras
        - Math Formula: expressão matemática
        - Joiner: chaves de join, tipo de join
        
        Retorna dict recursivo com toda estrutura do model.
        """
        pass
    
    def _extract_ports(self) -> List[PortInfo]:
        """Extrai informações das portas de saída."""
        pass
    
    def _extract_flow_variables(self) -> Dict[str, Any]:
        """
        Extrai variáveis de fluxo do flow_stack.
        
        Variáveis de fluxo são usadas para:
        - Passar parâmetros entre nodes
        - Controlar loops
        - Variáveis de data de referência
        """
        pass
    
    def _parse_config_recursive(self, element: ET.Element) -> Dict[str, Any]:
        """
        Parseia um elemento config recursivamente.
        
        Converte estrutura XML em dict Python:
        - entry com type="xstring" → str
        - entry com type="xint" → int
        - entry com type="xdouble" → float
        - entry com type="xboolean" → bool
        - config aninhado → dict recursivo
        - array-size → lista
        """
        pass
    
    def _convert_value(self, value: str, value_type: str) -> Any:
        """Converte valor string para tipo Python apropriado."""
        pass
    
    def get_setting(self, path: str, default: Any = None) -> Any:
        """
        Busca configuração por caminho (ex: "model/expression").
        
        Args:
            path: Caminho separado por /
            default: Valor default se não encontrar
            
        Returns:
            Valor encontrado ou default
        """
        pass


def parse_settings(settings_path: Path) -> NodeSettings:
    """Função de conveniência para parsear settings."""
    parser = SettingsParser(settings_path)
    return parser.parse()
```

TESTES A CRIAR em tests/test_settings_parser.py:
- test_extract_factory
- test_extract_model_simple
- test_extract_model_nested (model com configs aninhados)
- test_extract_flow_variables
- test_parse_array_values (included_names com array-size)
- test_convert_types (string, int, double, boolean)
- test_get_setting_by_path

CASOS ESPECIAIS A TRATAR:
1. Valores null (isnull="true")
2. Arrays (array-size + entries numeradas)
3. Configs aninhados múltiplos níveis
4. Escape de caracteres especiais em strings

Gere o arquivo completo com todos os métodos implementados.
```

---

## PROMPT 1.4: Parser de spec.xml

```
Você é um Engenheiro de Software Python criando um parser de schemas.

TAREFA:
Criar o módulo que parseia arquivos spec.xml para extrair schema de saída dos nodes.

CONTEXTO:
Cada porta de saída executada tem uma pasta port_N/ contendo:
- spec.xml: Schema das colunas (nomes e tipos)
- data.xml: Dados em si (não precisamos parsear)

O spec.xml define as colunas que o node produz como output.

ARQUIVO A CRIAR: src/extractors/spec_parser.py

ESTRUTURA DO XML spec.xml:
```xml
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
        </config>
    </config>
    <config key="column_spec_1">
        <entry key="column_name" type="xstring" value="NmProduto"/>
        <config key="column_type">
            <entry key="cell_class" type="xstring" value="org.knime.core.data.def.StringCell"/>
        </config>
    </config>
</config>
```

IMPLEMENTAÇÃO DETALHADA:

```python
from dataclasses import dataclass, field
from typing import Dict, List, Optional, Any, Union
from pathlib import Path
import xml.etree.ElementTree as ET

KNIME_NS = {"k": "http://www.knime.org/2008/09/XMLConfig"}

# Mapeamento de tipos KNIME para Python/Pandas
KNIME_TYPE_MAP = {
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
    "org.knime.core.data.time.localdate.LocalDateCell": "datetime64[ns]",
    "org.knime.core.data.v2.time.LocalDateTimeValueFactory": "datetime64[ns]",
    "org.knime.core.data.v2.time.LocalDateValueFactory": "datetime64[ns]",
    
    # Outros
    "org.knime.core.data.blob.BinaryObjectCell": "object",
    "org.knime.core.data.xml.XMLCell": "str",
    "org.knime.core.data.collection.ListCell": "object",
    "org.knime.core.data.collection.SetCell": "object",
}


@dataclass
class ColumnDomain:
    """Domínio de valores de uma coluna."""
    lower_bound: Optional[Any] = None
    upper_bound: Optional[Any] = None
    possible_values: List[Any] = field(default_factory=list)


@dataclass
class ColumnSpec:
    """Especificação de uma coluna."""
    name: str
    knime_type: str
    python_type: str
    domain: Optional[ColumnDomain] = None
    
    @classmethod
    def from_knime_type(cls, name: str, knime_type: str, domain: ColumnDomain = None):
        """Cria ColumnSpec convertendo tipo KNIME para Python."""
        python_type = KNIME_TYPE_MAP.get(knime_type, "object")
        return cls(name=name, knime_type=knime_type, python_type=python_type, domain=domain)


@dataclass
class TableSchema:
    """Schema completo de uma tabela/porta."""
    columns: List[ColumnSpec]
    column_count: int
    spec_name: str = "default"
    
    def get_column(self, name: str) -> Optional[ColumnSpec]:
        """Busca coluna por nome."""
        for col in self.columns:
            if col.name == name:
                return col
        return None
    
    def get_column_names(self) -> List[str]:
        """Retorna lista de nomes de colunas."""
        return [col.name for col in self.columns]
    
    def get_dtypes_dict(self) -> Dict[str, str]:
        """Retorna dict de tipos para pd.DataFrame.astype()."""
        return {col.name: col.python_type for col in self.columns}
    
    def to_dict(self) -> Dict:
        """Serializa para dicionário."""
        return {
            "spec_name": self.spec_name,
            "column_count": self.column_count,
            "columns": [
                {
                    "name": col.name,
                    "knime_type": col.knime_type,
                    "python_type": col.python_type,
                    "domain": {
                        "lower_bound": col.domain.lower_bound if col.domain else None,
                        "upper_bound": col.domain.upper_bound if col.domain else None,
                        "possible_values": col.domain.possible_values if col.domain else []
                    } if col.domain else None
                }
                for col in self.columns
            ]
        }


class SpecParser:
    """
    Parser para arquivos spec.xml de portas KNIME.
    
    Extrai:
    - Lista de colunas com nomes e tipos
    - Domínio de valores (min/max para numéricos, possíveis para categóricos)
    - Contagem de colunas
    """
    
    def __init__(self, spec_path: Path):
        self.spec_path = Path(spec_path)
        self._tree = None
        self._root = None
    
    def parse(self) -> TableSchema:
        """
        Parseia spec.xml e retorna TableSchema.
        
        Returns:
            TableSchema com todas colunas e tipos
        """
        pass
    
    def _extract_column_count(self) -> int:
        """Extrai número de colunas declarado."""
        pass
    
    def _extract_columns(self, count: int) -> List[ColumnSpec]:
        """
        Extrai especificação de cada coluna.
        
        Itera column_spec_0 até column_spec_{count-1}.
        """
        pass
    
    def _parse_column_spec(self, config: ET.Element) -> ColumnSpec:
        """
        Parseia um column_spec individual.
        
        Extrai:
        - column_name
        - cell_class (tipo KNIME)
        - domain (opcional)
        """
        pass
    
    def _parse_column_domain(self, domain_config: ET.Element) -> Optional[ColumnDomain]:
        """
        Parseia domínio de uma coluna.
        
        Tipos de domínio:
        - Numérico: lower_bound, upper_bound
        - Categórico: possible_values (lista)
        """
        pass
    
    @staticmethod
    def map_knime_type(knime_type: str) -> str:
        """
        Mapeia tipo KNIME para tipo Python/Pandas.
        
        Args:
            knime_type: Classe Java do KNIME (ex: org.knime.core.data.def.DoubleCell)
            
        Returns:
            Tipo Python equivalente (ex: Float64)
        """
        if not knime_type:
            return "object"
        
        # Busca direta
        if knime_type in KNIME_TYPE_MAP:
            return KNIME_TYPE_MAP[knime_type]
        
        # Busca parcial por palavras-chave
        knime_lower = knime_type.lower()
        if "double" in knime_lower or "float" in knime_lower:
            return "Float64"
        if "int" in knime_lower or "long" in knime_lower:
            return "Int64"
        if "string" in knime_lower:
            return "str"
        if "bool" in knime_lower:
            return "bool"
        if "date" in knime_lower or "time" in knime_lower:
            return "datetime64[ns]"
        
        return "object"


def parse_spec(spec_path: Path) -> TableSchema:
    """Função de conveniência para parsear spec."""
    parser = SpecParser(spec_path)
    return parser.parse()


def find_and_parse_specs(node_folder: Path) -> Dict[int, TableSchema]:
    """
    Encontra e parseia todos os spec.xml de um node.
    
    Args:
        node_folder: Pasta do node
        
    Returns:
        Dict com port_index → TableSchema
    """
    specs = {}
    for port_dir in node_folder.glob("port_*"):
        if port_dir.is_dir():
            spec_file = port_dir / "spec.xml"
            if spec_file.exists():
                # Extrair índice da porta do nome da pasta
                port_index = int(port_dir.name.split("_")[1])
                specs[port_index] = parse_spec(spec_file)
    return specs
```

TESTES A CRIAR em tests/test_spec_parser.py:
- test_parse_basic_schema
- test_extract_column_types
- test_map_knime_type_direct
- test_map_knime_type_partial_match
- test_parse_numeric_domain
- test_parse_categorical_domain
- test_find_and_parse_specs
- test_schema_to_dict

Gere o arquivo completo com todos os métodos implementados.
```

---

## PROMPT 1.5: Integração dos Parsers + Expansão de Metanodes

```
Você é um Engenheiro de Software Python integrando módulos de um sistema.

TAREFA:
Criar o módulo integrador que usa todos os parsers e expande metanodes recursivamente.

ARQUIVOS A CRIAR/MODIFICAR:

1. **src/extractors/workflow_extractor.py** - Integrador principal:

```python
from dataclasses import dataclass, field
from typing import Dict, List, Optional, Any
from pathlib import Path
import json

from .zip_extractor import ZipExtractor
from .workflow_parser import WorkflowParser, KnimeNode, KnimeConnection, WorkflowMetadata
from .settings_parser import SettingsParser, NodeSettings
from .spec_parser import find_and_parse_specs, TableSchema


@dataclass
class EnrichedNode:
    """Node enriquecido com todas as informações extraídas."""
    id: str
    name: str
    display_name: str
    folder_path: Path
    
    # Do workflow_parser
    is_metanode: bool
    node_type: str
    position: Dict[str, int]
    
    # Do settings_parser
    factory: str
    state: str
    annotation: str
    model: Dict[str, Any]
    category: str
    
    # Do spec_parser
    output_schemas: Dict[int, TableSchema]  # port_index → schema
    
    # Para metanodes
    internal_workflow: Optional['ExtractedWorkflow'] = None
    
    def to_dict(self) -> Dict:
        """Serializa para dicionário."""
        pass


@dataclass
class ExtractedWorkflow:
    """Workflow completo extraído."""
    source_file: str
    extraction_path: Path
    metadata: WorkflowMetadata
    nodes: Dict[str, EnrichedNode]  # id → EnrichedNode
    connections: List[KnimeConnection]
    annotations: List[Dict]
    
    def get_node(self, node_id: str) -> Optional[EnrichedNode]:
        """Busca node por ID."""
        return self.nodes.get(str(node_id))
    
    def get_predecessors(self, node_id: str) -> List[str]:
        """Retorna IDs dos nodes predecessores."""
        pass
    
    def get_successors(self, node_id: str) -> List[str]:
        """Retorna IDs dos nodes sucessores."""
        pass
    
    def to_dict(self) -> Dict:
        """Serializa para dicionário (workflow_ir.json)."""
        pass
    
    def save_as_json(self, output_path: Path):
        """Salva como JSON."""
        pass


class WorkflowExtractor:
    """
    Extrator completo de workflows KNIME.
    
    Integra todos os parsers:
    1. ZipExtractor - descompacta .knwf
    2. WorkflowParser - extrai nodes e conexões
    3. SettingsParser - extrai configurações de cada node
    4. SpecParser - extrai schemas de saída
    
    Expande metanodes recursivamente.
    """
    
    def __init__(self, knwf_path: str, output_dir: str = None):
        self.knwf_path = Path(knwf_path)
        self.output_dir = Path(output_dir) if output_dir else None
        self._zip_extractor = None
        self._extraction_path = None
    
    def extract(self) -> ExtractedWorkflow:
        """
        Extrai workflow completo.
        
        Fluxo:
        1. Descompactar ZIP
        2. Parsear workflow.knime principal
        3. Para cada node:
           a. Parsear settings.xml
           b. Parsear spec.xml (se existir)
           c. Se metanode, extrair recursivamente
        4. Montar ExtractedWorkflow
        
        Returns:
            ExtractedWorkflow com todos dados
        """
        pass
    
    def _extract_from_folder(self, folder: Path, is_root: bool = True) -> ExtractedWorkflow:
        """
        Extrai workflow de uma pasta (raiz ou metanode).
        
        Reutilizado para metanodes (recursivo).
        """
        pass
    
    def _enrich_node(self, node: KnimeNode, base_folder: Path) -> EnrichedNode:
        """
        Enriquece KnimeNode com settings e specs.
        
        Args:
            node: Node básico do workflow_parser
            base_folder: Pasta base do workflow
            
        Returns:
            EnrichedNode com todas informações
        """
        pass
    
    def _expand_metanode(self, node: EnrichedNode) -> Optional[ExtractedWorkflow]:
        """
        Expande um metanode, extraindo seu workflow interno.
        
        Metanodes contêm workflow.knime próprio na sua pasta.
        """
        pass
    
    def _find_node_folder(self, base_folder: Path, settings_file: str) -> Optional[Path]:
        """
        Encontra pasta do node a partir do settings_file path.
        
        settings_file é algo como "CSV Reader (#1)/settings.xml"
        """
        pass
    
    def cleanup(self):
        """Remove arquivos temporários."""
        if self._zip_extractor:
            self._zip_extractor.cleanup()


def extract_workflow(knwf_path: str, output_dir: str = None) -> ExtractedWorkflow:
    """
    Função de conveniência para extrair workflow.
    
    Args:
        knwf_path: Caminho para .knwf
        output_dir: Diretório para extração (opcional)
        
    Returns:
        ExtractedWorkflow completo
    """
    extractor = WorkflowExtractor(knwf_path, output_dir)
    try:
        return extractor.extract()
    finally:
        extractor.cleanup()
```

2. **Atualizar src/extractors/__init__.py**:

```python
from .zip_extractor import ZipExtractor
from .workflow_parser import WorkflowParser, KnimeNode, KnimeConnection, WorkflowMetadata, parse_workflow
from .settings_parser import SettingsParser, NodeSettings, parse_settings
from .spec_parser import SpecParser, TableSchema, ColumnSpec, parse_spec, find_and_parse_specs
from .workflow_extractor import WorkflowExtractor, ExtractedWorkflow, EnrichedNode, extract_workflow

__all__ = [
    'ZipExtractor',
    'WorkflowParser', 'KnimeNode', 'KnimeConnection', 'WorkflowMetadata', 'parse_workflow',
    'SettingsParser', 'NodeSettings', 'parse_settings',
    'SpecParser', 'TableSchema', 'ColumnSpec', 'parse_spec', 'find_and_parse_specs',
    'WorkflowExtractor', 'ExtractedWorkflow', 'EnrichedNode', 'extract_workflow',
]
```

3. **Atualizar main.py** para usar o integrador:

```python
import argparse
import json
from pathlib import Path

from src.extractors import extract_workflow
from src.utils.logger import setup_logger

logger = setup_logger(__name__)


def main():
    parser = argparse.ArgumentParser(description='KNIME to Python Converter - Extração')
    parser.add_argument('knwf_file', help='Caminho para arquivo .knwf')
    parser.add_argument('-o', '--output', help='Diretório de saída', default='output')
    parser.add_argument('-v', '--verbose', action='store_true', help='Modo verbose')
    
    args = parser.parse_args()
    
    # Extrair workflow
    logger.info(f"Extraindo workflow: {args.knwf_file}")
    workflow = extract_workflow(args.knwf_file)
    
    # Exibir resumo
    print(f"\n{'='*60}")
    print(f"WORKFLOW: {workflow.source_file}")
    print(f"{'='*60}")
    print(f"Nodes: {len(workflow.nodes)}")
    print(f"Conexões: {len(workflow.connections)}")
    print(f"Metanodes: {sum(1 for n in workflow.nodes.values() if n.is_metanode)}")
    
    # Listar nodes
    print(f"\n{'='*60}")
    print("NODES:")
    print(f"{'='*60}")
    for node_id, node in sorted(workflow.nodes.items(), key=lambda x: int(x[0])):
        meta_flag = " [METANODE]" if node.is_metanode else ""
        print(f"  [{node_id}] {node.display_name}{meta_flag}")
        print(f"      Factory: {node.factory}")
        print(f"      Category: {node.category}")
        print(f"      State: {node.state}")
        if node.output_schemas:
            for port, schema in node.output_schemas.items():
                print(f"      Output Port {port}: {schema.column_count} colunas")
    
    # Salvar JSON
    output_path = Path(args.output) / "workflow_extracted.json"
    output_path.parent.mkdir(parents=True, exist_ok=True)
    workflow.save_as_json(output_path)
    logger.info(f"Workflow salvo em: {output_path}")


if __name__ == "__main__":
    main()
```

TESTES A CRIAR em tests/test_workflow_extractor.py:
- test_extract_simple_workflow
- test_extract_with_metanode
- test_enrich_node_with_settings
- test_enrich_node_with_spec
- test_recursive_metanode_expansion
- test_to_dict_serialization

CRITÉRIOS DE VALIDAÇÃO:
1. Workflow extraído contém todos os nodes
2. Cada node tem factory e settings extraídos
3. Nodes executados têm schema de saída
4. Metanodes têm internal_workflow preenchido
5. JSON de saída é válido e completo

Gere todos os arquivos completos e funcionais.
```

---

# 🏗️ SPRINT 2: REPRESENTAÇÃO INTERMEDIÁRIA (IR)

## PROMPT 2.1: Construtor de Grafo e Ordenação Topológica

```
Você é um Engenheiro de Software Python trabalhando com grafos direcionados.

TAREFA:
Criar o módulo que constrói o grafo de execução e calcula a ordem topológica.

ARQUIVOS A CRIAR:

1. **src/ir/__init__.py**
2. **src/ir/graph_builder.py**
3. **src/ir/topological_sort.py**

IMPLEMENTAÇÃO DE graph_builder.py:

```python
from dataclasses import dataclass, field
from typing import Dict, List, Set, Tuple, Optional
from collections import defaultdict

# Não usar networkx para manter dependências mínimas
# Implementar grafo próprio


@dataclass
class GraphNode:
    """Nó do grafo de execução."""
    id: str
    name: str
    factory: str
    category: str
    is_metanode: bool
    is_loop_start: bool = False
    is_loop_end: bool = False
    predecessors: Set[str] = field(default_factory=set)
    successors: Set[str] = field(default_factory=set)
    input_ports: Dict[int, Tuple[str, int]] = field(default_factory=dict)  # port → (source_node, source_port)
    output_ports: Dict[int, List[Tuple[str, int]]] = field(default_factory=dict)  # port → [(dest_node, dest_port)]
    
    # Dados adicionais
    settings: Dict = field(default_factory=dict)
    output_schemas: Dict = field(default_factory=dict)


@dataclass
class GraphEdge:
    """Aresta do grafo."""
    source_id: str
    target_id: str
    source_port: int
    target_port: int
    edge_type: str = "data"  # data, flow


@dataclass
class LoopContext:
    """Contexto de um loop no workflow."""
    loop_id: str
    start_node_id: str
    end_node_id: str
    internal_nodes: Set[str]
    loop_variables: List[Dict]


class ExecutionGraph:
    """
    Grafo de execução do workflow KNIME.
    
    Responsabilidades:
    - Armazenar nodes e conexões
    - Identificar predecessores/sucessores
    - Detectar loops
    - Fornecer iteração na ordem correta
    """
    
    def __init__(self):
        self.nodes: Dict[str, GraphNode] = {}
        self.edges: List[GraphEdge] = []
        self.loops: List[LoopContext] = []
        self._adjacency: Dict[str, Set[str]] = defaultdict(set)  # node → successors
        self._reverse_adj: Dict[str, Set[str]] = defaultdict(set)  # node → predecessors
    
    def add_node(self, node: GraphNode):
        """Adiciona um node ao grafo."""
        self.nodes[node.id] = node
    
    def add_edge(self, edge: GraphEdge):
        """
        Adiciona uma aresta ao grafo.
        
        Atualiza:
        - Lista de edges
        - Adjacência direta e reversa
        - input_ports e output_ports dos nodes
        """
        self.edges.append(edge)
        self._adjacency[edge.source_id].add(edge.target_id)
        self._reverse_adj[edge.target_id].add(edge.source_id)
        
        # Atualizar ports dos nodes
        if edge.source_id in self.nodes:
            source_node = self.nodes[edge.source_id]
            if edge.source_port not in source_node.output_ports:
                source_node.output_ports[edge.source_port] = []
            source_node.output_ports[edge.source_port].append((edge.target_id, edge.target_port))
            source_node.successors.add(edge.target_id)
        
        if edge.target_id in self.nodes:
            target_node = self.nodes[edge.target_id]
            target_node.input_ports[edge.target_port] = (edge.source_id, edge.source_port)
            target_node.predecessors.add(edge.source_id)
    
    def get_node(self, node_id: str) -> Optional[GraphNode]:
        """Retorna node por ID."""
        return self.nodes.get(node_id)
    
    def get_predecessors(self, node_id: str) -> Set[str]:
        """Retorna IDs dos predecessores."""
        return self._reverse_adj.get(node_id, set())
    
    def get_successors(self, node_id: str) -> Set[str]:
        """Retorna IDs dos sucessores."""
        return self._adjacency.get(node_id, set())
    
    def get_root_nodes(self) -> List[str]:
        """Retorna nodes sem predecessores (entrada do workflow)."""
        return [nid for nid in self.nodes if not self._reverse_adj.get(nid)]
    
    def get_leaf_nodes(self) -> List[str]:
        """Retorna nodes sem sucessores (saída do workflow)."""
        return [nid for nid in self.nodes if not self._adjacency.get(nid)]
    
    def detect_loops(self) -> List[LoopContext]:
        """
        Detecta estruturas de loop no workflow.
        
        Loops são identificados por:
        1. Node com factory contendo "LoopStart" → is_loop_start = True
        2. Node com factory contendo "LoopEnd" → is_loop_end = True
        3. Nodes entre start e end são internal_nodes
        
        Returns:
            Lista de LoopContext encontrados
        """
        loops = []
        
        # Encontrar nodes de início de loop
        loop_starts = [
            nid for nid, node in self.nodes.items()
            if 'loopstart' in node.factory.lower() or 'loop start' in node.name.lower()
        ]
        
        for start_id in loop_starts:
            # Marcar como loop start
            self.nodes[start_id].is_loop_start = True
            
            # Encontrar o loop end correspondente
            end_id = self._find_loop_end(start_id)
            if end_id:
                self.nodes[end_id].is_loop_end = True
                
                # Encontrar nodes internos
                internal = self._find_loop_internal_nodes(start_id, end_id)
                
                loops.append(LoopContext(
                    loop_id=f"loop_{start_id}_{end_id}",
                    start_node_id=start_id,
                    end_node_id=end_id,
                    internal_nodes=internal,
                    loop_variables=[]  # Será preenchido depois
                ))
        
        self.loops = loops
        return loops
    
    def _find_loop_end(self, start_id: str) -> Optional[str]:
        """Encontra o Loop End correspondente a um Loop Start."""
        # BFS para encontrar o loop end
        visited = set()
        queue = [start_id]
        
        while queue:
            current = queue.pop(0)
            if current in visited:
                continue
            visited.add(current)
            
            node = self.nodes.get(current)
            if node and ('loopend' in node.factory.lower() or 'loop end' in node.name.lower()):
                return current
            
            queue.extend(self._adjacency.get(current, []))
        
        return None
    
    def _find_loop_internal_nodes(self, start_id: str, end_id: str) -> Set[str]:
        """Encontra todos os nodes entre loop start e end."""
        internal = set()
        visited = set()
        queue = list(self._adjacency.get(start_id, []))
        
        while queue:
            current = queue.pop(0)
            if current in visited or current == end_id:
                continue
            visited.add(current)
            internal.add(current)
            queue.extend(self._adjacency.get(current, []))
        
        return internal
    
    def to_dict(self) -> Dict:
        """Serializa grafo para dicionário."""
        return {
            "nodes": {
                nid: {
                    "id": node.id,
                    "name": node.name,
                    "factory": node.factory,
                    "category": node.category,
                    "is_metanode": node.is_metanode,
                    "is_loop_start": node.is_loop_start,
                    "is_loop_end": node.is_loop_end,
                    "predecessors": list(node.predecessors),
                    "successors": list(node.successors),
                }
                for nid, node in self.nodes.items()
            },
            "edges": [
                {
                    "source": e.source_id,
                    "target": e.target_id,
                    "source_port": e.source_port,
                    "target_port": e.target_port,
                    "type": e.edge_type
                }
                for e in self.edges
            ],
            "loops": [
                {
                    "loop_id": loop.loop_id,
                    "start_node": loop.start_node_id,
                    "end_node": loop.end_node_id,
                    "internal_nodes": list(loop.internal_nodes)
                }
                for loop in self.loops
            ]
        }


class GraphBuilder:
    """
    Construtor de ExecutionGraph a partir de ExtractedWorkflow.
    """
    
    def __init__(self, extracted_workflow):
        """
        Args:
            extracted_workflow: ExtractedWorkflow do módulo extractors
        """
        self.workflow = extracted_workflow
    
    def build(self) -> ExecutionGraph:
        """
        Constrói o grafo de execução.
        
        Fluxo:
        1. Criar GraphNode para cada EnrichedNode
        2. Criar GraphEdge para cada KnimeConnection
        3. Detectar loops
        
        Returns:
            ExecutionGraph completo
        """
        graph = ExecutionGraph()
        
        # Adicionar nodes
        for node_id, enriched_node in self.workflow.nodes.items():
            graph_node = GraphNode(
                id=node_id,
                name=enriched_node.name,
                factory=enriched_node.factory,
                category=enriched_node.category,
                is_metanode=enriched_node.is_metanode,
                settings=enriched_node.model,
                output_schemas={
                    port: schema.to_dict() 
                    for port, schema in enriched_node.output_schemas.items()
                }
            )
            graph.add_node(graph_node)
        
        # Adicionar edges
        for conn in self.workflow.connections:
            edge = GraphEdge(
                source_id=str(conn.source_id),
                target_id=str(conn.dest_id),
                source_port=conn.source_port,
                target_port=conn.dest_port,
                edge_type=conn.connection_type
            )
            graph.add_edge(edge)
        
        # Detectar loops
        graph.detect_loops()
        
        return graph


def build_graph(extracted_workflow) -> ExecutionGraph:
    """Função de conveniência."""
    builder = GraphBuilder(extracted_workflow)
    return builder.build()
```

IMPLEMENTAÇÃO DE topological_sort.py:

```python
from typing import List, Dict, Set
from collections import deque

from .graph_builder import ExecutionGraph


class TopologicalSorter:
    """
    Ordena nodes do grafo em ordem topológica (ordem de execução).
    
    Usa algoritmo de Kahn (BFS).
    Trata loops como unidades atômicas.
    """
    
    def __init__(self, graph: ExecutionGraph):
        self.graph = graph
    
    def sort(self) -> List[str]:
        """
        Calcula ordem topológica.
        
        Algoritmo de Kahn:
        1. Calcular in-degree de cada node
        2. Adicionar nodes com in-degree=0 à fila
        3. Para cada node na fila:
           - Adicionar à ordem
           - Decrementar in-degree dos sucessores
           - Se in-degree do sucessor == 0, adicionar à fila
        
        Returns:
            Lista de node_ids na ordem de execução
            
        Raises:
            ValueError: Se detectar ciclo (que não seja loop declarado)
        """
        # Calcular in-degree
        in_degree = {nid: 0 for nid in self.graph.nodes}
        
        for edge in self.graph.edges:
            if edge.target_id in in_degree:
                in_degree[edge.target_id] += 1
        
        # Fila inicial: nodes sem predecessores
        queue = deque([nid for nid, deg in in_degree.items() if deg == 0])
        order = []
        
        while queue:
            node_id = queue.popleft()
            order.append(node_id)
            
            # Decrementar in-degree dos sucessores
            for successor_id in self.graph.get_successors(node_id):
                in_degree[successor_id] -= 1
                if in_degree[successor_id] == 0:
                    queue.append(successor_id)
        
        # Verificar se todos os nodes foram incluídos
        if len(order) != len(self.graph.nodes):
            # Nodes não incluídos podem indicar ciclo
            missing = set(self.graph.nodes.keys()) - set(order)
            
            # Verificar se nodes faltantes estão em loops declarados
            loop_nodes = set()
            for loop in self.graph.loops:
                loop_nodes.add(loop.start_node_id)
                loop_nodes.add(loop.end_node_id)
                loop_nodes.update(loop.internal_nodes)
            
            unexpected = missing - loop_nodes
            if unexpected:
                raise ValueError(f"Ciclo detectado envolvendo nodes: {unexpected}")
            
            # Adicionar nodes de loop no final (serão tratados especialmente)
            for nid in missing:
                if nid not in order:
                    order.append(nid)
        
        return order
    
    def get_execution_groups(self) -> List[List[str]]:
        """
        Agrupa nodes que podem ser executados em paralelo.
        
        Nodes no mesmo grupo não têm dependência entre si.
        
        Returns:
            Lista de grupos, cada grupo é uma lista de node_ids
        """
        groups = []
        remaining = set(self.graph.nodes.keys())
        executed = set()
        
        while remaining:
            # Encontrar nodes cujos predecessores já foram executados
            ready = []
            for nid in remaining:
                preds = self.graph.get_predecessors(nid)
                if preds.issubset(executed):
                    ready.append(nid)
            
            if not ready:
                # Todos restantes têm dependências não satisfeitas (ciclo)
                break
            
            groups.append(ready)
            executed.update(ready)
            remaining -= set(ready)
        
        return groups


def topological_sort(graph: ExecutionGraph) -> List[str]:
    """Função de conveniência."""
    sorter = TopologicalSorter(graph)
    return sorter.sort()


def get_execution_order(graph: ExecutionGraph) -> List[str]:
    """Alias para topological_sort."""
    return topological_sort(graph)
```

TESTES A CRIAR em tests/test_ir.py:
- test_build_graph_nodes
- test_build_graph_edges
- test_graph_predecessors_successors
- test_detect_loops
- test_topological_sort_simple
- test_topological_sort_with_branches
- test_topological_sort_cycle_detection
- test_execution_groups

Gere todos os arquivos completos.
```

---

## PROMPT 2.2: Exportador de IR (workflow_ir.json)

```
Você é um Engenheiro de Software Python criando um serializador de dados.

TAREFA:
Criar o módulo que exporta a Representação Intermediária completa para JSON.

ARQUIVO A CRIAR: src/ir/ir_exporter.py

```python
from dataclasses import dataclass, field, asdict
from typing import Dict, List, Optional, Any
from pathlib import Path
from datetime import datetime
import json

from .graph_builder import ExecutionGraph, GraphNode, LoopContext
from .topological_sort import topological_sort


@dataclass
class NodeIR:
    """Representação Intermediária de um Node."""
    id: str
    name: str
    display_name: str
    factory: str
    category: str
    annotation: str
    
    # Estrutura
    is_metanode: bool
    is_loop_start: bool
    is_loop_end: bool
    position: Dict[str, int]
    
    # Estado
    state: str
    classification: str  # MAPPED, CANDIDATE, UNKNOWN
    
    # Configurações
    settings: Dict[str, Any]
    
    # Portas
    input_ports: List[Dict]
    output_ports: List[Dict]
    
    # Schema de saída
    output_schemas: Dict[int, Dict]
    
    # Código gerado (preenchido depois)
    python_code: Optional[str] = None
    imports: List[str] = field(default_factory=list)
    
    # Metanode interno
    internal_workflow_ir: Optional['WorkflowIR'] = None


@dataclass
class EdgeIR:
    """Representação Intermediária de uma Conexão."""
    id: str
    source_node: str
    source_port: int
    target_node: str
    target_port: int
    edge_type: str


@dataclass
class LoopIR:
    """Representação Intermediária de um Loop."""
    id: str
    start_node: str
    end_node: str
    internal_nodes: List[str]
    loop_variables: List[Dict]


@dataclass
class WorkflowIR:
    """Representação Intermediária completa do Workflow."""
    
    # Metadados
    source_file: str
    knime_version: str
    extracted_at: str
    author: str
    last_editor: str
    description: str
    
    # Grafo
    nodes: Dict[str, NodeIR]
    edges: List[EdgeIR]
    execution_order: List[str]
    loops: List[LoopIR]
    
    # Estatísticas
    statistics: Dict[str, Any] = field(default_factory=dict)
    
    def to_dict(self) -> Dict:
        """Converte para dicionário serializável."""
        return {
            "metadata": {
                "source_file": self.source_file,
                "knime_version": self.knime_version,
                "extracted_at": self.extracted_at,
                "author": self.author,
                "last_editor": self.last_editor,
                "description": self.description
            },
            "graph": {
                "nodes": {
                    nid: {
                        "id": node.id,
                        "name": node.name,
                        "display_name": node.display_name,
                        "factory": node.factory,
                        "category": node.category,
                        "annotation": node.annotation,
                        "is_metanode": node.is_metanode,
                        "is_loop_start": node.is_loop_start,
                        "is_loop_end": node.is_loop_end,
                        "position": node.position,
                        "state": node.state,
                        "classification": node.classification,
                        "settings": node.settings,
                        "input_ports": node.input_ports,
                        "output_ports": node.output_ports,
                        "output_schemas": node.output_schemas,
                        "python_code": node.python_code,
                        "imports": node.imports,
                        "internal_workflow": node.internal_workflow_ir.to_dict() if node.internal_workflow_ir else None
                    }
                    for nid, node in self.nodes.items()
                },
                "edges": [
                    {
                        "id": edge.id,
                        "source_node": edge.source_node,
                        "source_port": edge.source_port,
                        "target_node": edge.target_node,
                        "target_port": edge.target_port,
                        "type": edge.edge_type
                    }
                    for edge in self.edges
                ],
                "execution_order": self.execution_order,
                "loops": [
                    {
                        "id": loop.id,
                        "start_node": loop.start_node,
                        "end_node": loop.end_node,
                        "internal_nodes": loop.internal_nodes,
                        "loop_variables": loop.loop_variables
                    }
                    for loop in self.loops
                ]
            },
            "statistics": self.statistics
        }
    
    def save(self, output_path: Path):
        """Salva IR como JSON."""
        output_path = Path(output_path)
        output_path.parent.mkdir(parents=True, exist_ok=True)
        
        with open(output_path, 'w', encoding='utf-8') as f:
            json.dump(self.to_dict(), f, indent=2, ensure_ascii=False)
    
    @classmethod
    def load(cls, input_path: Path) -> 'WorkflowIR':
        """Carrega IR de arquivo JSON."""
        with open(input_path, 'r', encoding='utf-8') as f:
            data = json.load(f)
        
        return cls._from_dict(data)
    
    @classmethod
    def _from_dict(cls, data: Dict) -> 'WorkflowIR':
        """Reconstrói WorkflowIR a partir de dicionário."""
        metadata = data["metadata"]
        graph = data["graph"]
        
        # Reconstruir nodes
        nodes = {}
        for nid, node_data in graph["nodes"].items():
            internal_ir = None
            if node_data.get("internal_workflow"):
                internal_ir = cls._from_dict(node_data["internal_workflow"])
            
            nodes[nid] = NodeIR(
                id=node_data["id"],
                name=node_data["name"],
                display_name=node_data["display_name"],
                factory=node_data["factory"],
                category=node_data["category"],
                annotation=node_data.get("annotation", ""),
                is_metanode=node_data["is_metanode"],
                is_loop_start=node_data.get("is_loop_start", False),
                is_loop_end=node_data.get("is_loop_end", False),
                position=node_data.get("position", {}),
                state=node_data.get("state", "UNKNOWN"),
                classification=node_data.get("classification", "UNKNOWN"),
                settings=node_data.get("settings", {}),
                input_ports=node_data.get("input_ports", []),
                output_ports=node_data.get("output_ports", []),
                output_schemas=node_data.get("output_schemas", {}),
                python_code=node_data.get("python_code"),
                imports=node_data.get("imports", []),
                internal_workflow_ir=internal_ir
            )
        
        # Reconstruir edges
        edges = [
            EdgeIR(
                id=e["id"],
                source_node=e["source_node"],
                source_port=e["source_port"],
                target_node=e["target_node"],
                target_port=e["target_port"],
                edge_type=e.get("type", "data")
            )
            for e in graph["edges"]
        ]
        
        # Reconstruir loops
        loops = [
            LoopIR(
                id=l["id"],
                start_node=l["start_node"],
                end_node=l["end_node"],
                internal_nodes=l["internal_nodes"],
                loop_variables=l.get("loop_variables", [])
            )
            for l in graph.get("loops", [])
        ]
        
        return cls(
            source_file=metadata["source_file"],
            knime_version=metadata.get("knime_version", ""),
            extracted_at=metadata["extracted_at"],
            author=metadata.get("author", ""),
            last_editor=metadata.get("last_editor", ""),
            description=metadata.get("description", ""),
            nodes=nodes,
            edges=edges,
            execution_order=graph["execution_order"],
            loops=loops,
            statistics=data.get("statistics", {})
        )


class IRExporter:
    """
    Exporta ExtractedWorkflow + ExecutionGraph para WorkflowIR.
    
    Combina dados de:
    - ExtractedWorkflow (nodes, settings, schemas)
    - ExecutionGraph (conexões, ordem, loops)
    """
    
    def __init__(self, extracted_workflow, execution_graph: ExecutionGraph):
        self.workflow = extracted_workflow
        self.graph = execution_graph
    
    def export(self) -> WorkflowIR:
        """
        Gera WorkflowIR completo.
        
        Returns:
            WorkflowIR pronto para serialização
        """
        # Calcular ordem topológica
        execution_order = topological_sort(self.graph)
        
        # Exportar nodes
        nodes = {}
        for node_id, enriched_node in self.workflow.nodes.items():
            graph_node = self.graph.get_node(node_id)
            
            # Construir input_ports
            input_ports = []
            if graph_node:
                for port_idx, (src_node, src_port) in graph_node.input_ports.items():
                    input_ports.append({
                        "index": port_idx,
                        "source_node": src_node,
                        "source_port": src_port,
                        "type": "data"
                    })
            
            # Construir output_ports
            output_ports = []
            if graph_node:
                for port_idx, destinations in graph_node.output_ports.items():
                    schema = enriched_node.output_schemas.get(port_idx)
                    output_ports.append({
                        "index": port_idx,
                        "destinations": [{"node": d[0], "port": d[1]} for d in destinations],
                        "schema": schema.to_dict() if schema else None,
                        "type": "data"
                    })
            
            # Determinar classificação inicial (UNKNOWN, será atualizado depois)
            classification = "UNKNOWN"
            
            nodes[node_id] = NodeIR(
                id=node_id,
                name=enriched_node.name,
                display_name=enriched_node.display_name,
                factory=enriched_node.factory,
                category=enriched_node.category,
                annotation=enriched_node.annotation,
                is_metanode=enriched_node.is_metanode,
                is_loop_start=graph_node.is_loop_start if graph_node else False,
                is_loop_end=graph_node.is_loop_end if graph_node else False,
                position=enriched_node.position,
                state=enriched_node.state,
                classification=classification,
                settings=enriched_node.model,
                input_ports=input_ports,
                output_ports=output_ports,
                output_schemas={
                    port: schema.to_dict()
                    for port, schema in enriched_node.output_schemas.items()
                },
                internal_workflow_ir=self._export_metanode(enriched_node) if enriched_node.is_metanode else None
            )
        
        # Exportar edges
        edges = []
        for i, conn in enumerate(self.workflow.connections):
            edges.append(EdgeIR(
                id=f"e{i}",
                source_node=str(conn.source_id),
                source_port=conn.source_port,
                target_node=str(conn.dest_id),
                target_port=conn.dest_port,
                edge_type=conn.connection_type
            ))
        
        # Exportar loops
        loops = []
        for loop in self.graph.loops:
            loops.append(LoopIR(
                id=loop.loop_id,
                start_node=loop.start_node_id,
                end_node=loop.end_node_id,
                internal_nodes=list(loop.internal_nodes),
                loop_variables=loop.loop_variables
            ))
        
        # Calcular estatísticas
        statistics = self._calculate_statistics(nodes, edges, loops)
        
        return WorkflowIR(
            source_file=self.workflow.source_file,
            knime_version=self.workflow.metadata.knime_version if self.workflow.metadata else "",
            extracted_at=datetime.now().isoformat(),
            author=self.workflow.metadata.author if self.workflow.metadata else "",
            last_editor=self.workflow.metadata.last_modified_by if self.workflow.metadata else "",
            description="",
            nodes=nodes,
            edges=edges,
            execution_order=execution_order,
            loops=loops,
            statistics=statistics
        )
    
    def _export_metanode(self, enriched_node) -> Optional[WorkflowIR]:
        """Exporta workflow interno de metanode."""
        if not enriched_node.internal_workflow:
            return None
        
        # Recursão: criar graph e exportar
        from .graph_builder import build_graph
        
        internal_graph = build_graph(enriched_node.internal_workflow)
        internal_exporter = IRExporter(enriched_node.internal_workflow, internal_graph)
        return internal_exporter.export()
    
    def _calculate_statistics(self, nodes: Dict, edges: List, loops: List) -> Dict:
        """Calcula estatísticas do workflow."""
        categories = {}
        classifications = {"UNKNOWN": 0, "MAPPED": 0, "CANDIDATE": 0}
        
        for node in nodes.values():
            categories[node.category] = categories.get(node.category, 0) + 1
            classifications[node.classification] = classifications.get(node.classification, 0) + 1
        
        return {
            "total_nodes": len(nodes),
            "total_edges": len(edges),
            "total_loops": len(loops),
            "total_metanodes": sum(1 for n in nodes.values() if n.is_metanode),
            "by_category": categories,
            "by_classification": classifications
        }


def export_ir(extracted_workflow, execution_graph: ExecutionGraph, output_path: Path = None) -> WorkflowIR:
    """
    Função de conveniência para exportar IR.
    
    Args:
        extracted_workflow: ExtractedWorkflow do módulo extractors
        execution_graph: ExecutionGraph do módulo ir
        output_path: Caminho para salvar JSON (opcional)
        
    Returns:
        WorkflowIR exportado
    """
    exporter = IRExporter(extracted_workflow, execution_graph)
    ir = exporter.export()
    
    if output_path:
        ir.save(output_path)
    
    return ir
```

ATUALIZAR src/ir/__init__.py:
```python
from .graph_builder import ExecutionGraph, GraphNode, GraphEdge, LoopContext, GraphBuilder, build_graph
from .topological_sort import TopologicalSorter, topological_sort, get_execution_order
from .ir_exporter import WorkflowIR, NodeIR, EdgeIR, LoopIR, IRExporter, export_ir

__all__ = [
    'ExecutionGraph', 'GraphNode', 'GraphEdge', 'LoopContext', 'GraphBuilder', 'build_graph',
    'TopologicalSorter', 'topological_sort', 'get_execution_order',
    'WorkflowIR', 'NodeIR', 'EdgeIR', 'LoopIR', 'IRExporter', 'export_ir',
]
```

ATUALIZAR main.py para gerar workflow_ir.json:

```python
# Adicionar ao main.py existente:

from src.ir import build_graph, export_ir

def main():
    # ... código existente de extração ...
    
    # Construir grafo
    logger.info("Construindo grafo de execução...")
    graph = build_graph(workflow)
    
    # Exportar IR
    logger.info("Exportando Representação Intermediária...")
    output_path = Path(args.output) / "workflow_ir.json"
    ir = export_ir(workflow, graph, output_path)
    
    # Exibir estatísticas
    print(f"\n{'='*60}")
    print("ESTATÍSTICAS:")
    print(f"{'='*60}")
    for key, value in ir.statistics.items():
        print(f"  {key}: {value}")
    
    print(f"\nIR salvo em: {output_path}")
```

TESTES A CRIAR em tests/test_ir_exporter.py:
- test_export_basic_workflow
- test_export_with_metanode
- test_export_with_loops
- test_save_and_load_json
- test_statistics_calculation

Gere todos os arquivos completos.
```

---

# 🏗️ SPRINT 3: CLASSIFICAÇÃO E MAPEAMENTO DETERMINÍSTICO

## PROMPT 3.1: Estrutura de node_mapping.json e Classificador

```
Você é um Engenheiro de Software Python criando um sistema de classificação.

TAREFA:
Criar o arquivo node_mapping.json inicial com 25 nodes comuns e o módulo classificador.

ARQUIVOS A CRIAR:

1. **config/node_mapping.json** - Mapeamentos determinísticos dos 25 nodes mais comuns

2. **src/mapping/__init__.py**

3. **src/mapping/classifier.py** - Classifica nodes em MAPPED/CANDIDATE/UNKNOWN

4. **src/mapping/mapping_loader.py** - Carrega e valida node_mapping.json

NODES A MAPEAR (25 mais comuns em workflows bancários):

**I/O (5):**
1. CSV Reader - org.knime.base.node.io.csvreader.CSVReaderNodeFactory
2. CSV Writer - org.knime.base.node.io.csvwriter.CSVWriterNodeFactory  
3. Excel Reader - org.knime.ext.poi2.node.read.ExcelTableReaderNodeFactory
4. Table Creator - org.knime.base.node.io.tablecreator.TableCreatorNodeFactory
5. File Reader - org.knime.base.node.io.fileReaderNodeFactory

**Manipulação (8):**
6. Column Filter - org.knime.base.node.preproc.filter.column.DataColumnSpecFilterNodeFactory
7. Row Filter - org.knime.base.node.preproc.filter.row.RowFilterNodeFactory
8. Joiner - org.knime.base.node.preproc.joiner.Joiner2NodeFactory
9. Concatenate - org.knime.base.node.preproc.append.AppendNodeFactory
10. Sorter - org.knime.base.node.preproc.sorter.SorterNodeFactory
11. Column Rename - org.knime.base.node.preproc.rename.RenameNodeFactory
12. GroupBy - org.knime.base.node.preproc.groupby.GroupByNodeFactory
13. Pivoting - org.knime.base.node.preproc.pivot.Pivot2NodeFactory

**Transformação (6):**
14. Math Formula - org.knime.ext.jep.JEPNodeFactory
15. String Manipulation - org.knime.base.node.preproc.stringmanipulation.StringManipulationNodeFactory
16. Column Expressions - org.knime.base.node.preproc.expressions.ExpressionNodeFactory
17. Missing Value - org.knime.base.node.preproc.missingval.MissingValueHandlerNodeFactory
18. Type Conversion - org.knime.base.node.preproc.typeconv.TypeConversionNodeFactory
19. Date&Time Shift - org.knime.time.node.shift.DateTimeShiftNodeFactory

**Lógica (3):**
20. Rule Engine - org.knime.base.node.rules.engine.RuleEngineNodeFactory
21. IF Switch - org.knime.base.node.flowcontrol.ifswitch.IFSwitchNodeFactory
22. CASE Switch - org.knime.base.node.flowcontrol.caseswitch.CaseSwitchNodeFactory

**Flow Control (3):**
23. Table Row to Variable - org.knime.base.node.flowvariable.tablerowtovariable.TableRowToVariableNodeFactory
24. Variable to Table Row - org.knime.base.node.flowvariable.variabletotablerow.VariableToTableRowNodeFactory
25. Group Loop Start - org.knime.base.node.meta.looper.group.GroupLoopStartNodeFactory

ESTRUTURA DO node_mapping.json:

```json
{
  "version": "1.0.0",
  "last_updated": "2025-12-15",
  "statistics": {
    "total_mappings": 25,
    "by_category": {},
    "by_status": {}
  },
  "type_mappings": {
    "org.knime.core.data.def.DoubleCell": "Float64",
    // ... outros tipos
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
        "file_path": {"path": "model/url", "type": "string", "required": true},
        "delimiter": {"path": "model/colDelimiter", "type": "string", "default": ","},
        "has_header": {"path": "model/hasColHeader", "type": "boolean", "default": true}
      },
      "python_template": "df_{output_var} = pd.read_csv(r'{file_path}', sep='{delimiter}', header={header_int})",
      "template_variables": {
        "header_int": {"condition": "has_header", "true_value": "0", "false_value": "None"}
      },
      "imports": ["pandas as pd"],
      "output_behavior": "creates_dataframe"
    }
    // ... outros 24 nodes
  ]
}
```

IMPLEMENTAÇÃO DE classifier.py:

```python
from enum import Enum
from dataclasses import dataclass
from typing import Dict, List, Optional, Tuple
from pathlib import Path

from .mapping_loader import MappingLoader, NodeMapping


class Classification(Enum):
    """Classificação de um node para tradução."""
    MAPPED = "MAPPED"      # Template determinístico disponível
    CANDIDATE = "CANDIDATE"  # Mapeamento em validação
    UNKNOWN = "UNKNOWN"     # Requer interpretação IA


@dataclass
class ClassificationResult:
    """Resultado da classificação de um node."""
    node_id: str
    factory: str
    classification: Classification
    mapping: Optional[NodeMapping]  # Se MAPPED ou CANDIDATE
    confidence: float
    reason: str


class NodeClassifier:
    """
    Classifica nodes do workflow para determinar método de tradução.
    
    Fluxo:
    1. Buscar factory no node_mapping.json (OFFICIAL)
    2. Se não encontrar, buscar em candidates.json
    3. Se não encontrar, classificar como UNKNOWN
    """
    
    def __init__(self, mapping_dir: Path = None):
        """
        Args:
            mapping_dir: Diretório com arquivos de mapeamento
        """
        self.mapping_dir = mapping_dir or Path("config")
        self.loader = MappingLoader(self.mapping_dir)
        self._mappings = None
        self._candidates = None
    
    def load_mappings(self):
        """Carrega mapeamentos do disco."""
        self._mappings = self.loader.load_official_mappings()
        self._candidates = self.loader.load_candidates()
    
    def classify(self, node_id: str, factory: str) -> ClassificationResult:
        """
        Classifica um node.
        
        Args:
            node_id: ID do node
            factory: Factory class do node
            
        Returns:
            ClassificationResult com classificação e mapeamento
        """
        if self._mappings is None:
            self.load_mappings()
        
        # Buscar em mapeamentos oficiais
        mapping = self._mappings.get(factory)
        if mapping and mapping.status == "OFFICIAL":
            return ClassificationResult(
                node_id=node_id,
                factory=factory,
                classification=Classification.MAPPED,
                mapping=mapping,
                confidence=mapping.confidence,
                reason="Mapeamento oficial encontrado"
            )
        
        # Buscar em candidatos
        candidate = self._candidates.get(factory)
        if candidate:
            return ClassificationResult(
                node_id=node_id,
                factory=factory,
                classification=Classification.CANDIDATE,
                mapping=candidate,
                confidence=candidate.confidence,
                reason="Mapeamento candidato em validação"
            )
        
        # Não encontrado
        return ClassificationResult(
            node_id=node_id,
            factory=factory,
            classification=Classification.UNKNOWN,
            mapping=None,
            confidence=0.0,
            reason="Sem mapeamento - requer IA"
        )
    
    def classify_workflow(self, workflow_ir) -> Dict[str, ClassificationResult]:
        """
        Classifica todos os nodes de um workflow.
        
        Args:
            workflow_ir: WorkflowIR com nodes a classificar
            
        Returns:
            Dict node_id → ClassificationResult
        """
        results = {}
        for node_id, node in workflow_ir.nodes.items():
            result = self.classify(node_id, node.factory)
            results[node_id] = result
            
            # Atualizar classificação no node
            node.classification = result.classification.value
        
        return results
    
    def generate_coverage_report(self, results: Dict[str, ClassificationResult]) -> Dict:
        """
        Gera relatório de cobertura.
        
        Returns:
            Dict com estatísticas de classificação
        """
        total = len(results)
        by_classification = {c.value: 0 for c in Classification}
        by_category = {}
        unknown_factories = []
        
        for result in results.values():
            by_classification[result.classification.value] += 1
            
            if result.mapping:
                cat = result.mapping.category
                by_category[cat] = by_category.get(cat, 0) + 1
            
            if result.classification == Classification.UNKNOWN:
                if result.factory not in unknown_factories:
                    unknown_factories.append(result.factory)
        
        return {
            "total_nodes": total,
            "coverage": {
                "mapped": by_classification["MAPPED"],
                "mapped_pct": by_classification["MAPPED"] / total * 100 if total > 0 else 0,
                "candidate": by_classification["CANDIDATE"],
                "candidate_pct": by_classification["CANDIDATE"] / total * 100 if total > 0 else 0,
                "unknown": by_classification["UNKNOWN"],
                "unknown_pct": by_classification["UNKNOWN"] / total * 100 if total > 0 else 0,
            },
            "by_category": by_category,
            "unknown_factories": unknown_factories,
            "ai_required": by_classification["UNKNOWN"] > 0
        }
```

IMPLEMENTAÇÃO DE mapping_loader.py:

```python
from dataclasses import dataclass, field
from typing import Dict, List, Optional, Any
from pathlib import Path
import json


@dataclass
class SettingsExtraction:
    """Regra de extração de um parâmetro."""
    path: str
    param_type: str
    required: bool = False
    default: Any = None


@dataclass
class NodeMapping:
    """Mapeamento de um node KNIME para Python."""
    factory: str
    name: str
    category: str
    status: str  # OFFICIAL, CANDIDATE
    confidence: float
    complexity: str  # simple, medium, high
    
    settings_extraction: Dict[str, SettingsExtraction]
    python_template: str
    template_variables: Dict[str, Dict] = field(default_factory=dict)
    imports: List[str] = field(default_factory=list)
    output_behavior: str = "passthrough"
    
    # Para nodes que precisam de IA para parte da lógica
    requires_ai: bool = False
    ai_prompt_hint: str = ""


class MappingLoader:
    """
    Carrega e valida arquivos de mapeamento.
    """
    
    def __init__(self, config_dir: Path):
        self.config_dir = Path(config_dir)
    
    def load_official_mappings(self) -> Dict[str, NodeMapping]:
        """Carrega node_mapping.json."""
        mapping_file = self.config_dir / "node_mapping.json"
        if not mapping_file.exists():
            return {}
        
        with open(mapping_file, 'r', encoding='utf-8') as f:
            data = json.load(f)
        
        mappings = {}
        for item in data.get("mappings", []):
            mapping = self._parse_mapping(item)
            mappings[mapping.factory] = mapping
        
        return mappings
    
    def load_candidates(self) -> Dict[str, NodeMapping]:
        """Carrega candidates.json."""
        candidates_file = self.config_dir / "candidates.json"
        if not candidates_file.exists():
            return {}
        
        with open(candidates_file, 'r', encoding='utf-8') as f:
            data = json.load(f)
        
        candidates = {}
        for item in data.get("candidates", []):
            mapping = self._parse_candidate(item)
            candidates[mapping.factory] = mapping
        
        return candidates
    
    def _parse_mapping(self, data: Dict) -> NodeMapping:
        """Parseia um item de mapeamento."""
        settings_extraction = {}
        for param, config in data.get("settings_extraction", {}).items():
            settings_extraction[param] = SettingsExtraction(
                path=config["path"],
                param_type=config.get("type", "string"),
                required=config.get("required", False),
                default=config.get("default")
            )
        
        return NodeMapping(
            factory=data["factory"],
            name=data["name"],
            category=data["category"],
            status=data.get("status", "OFFICIAL"),
            confidence=data.get("confidence", 1.0),
            complexity=data.get("complexity", "simple"),
            settings_extraction=settings_extraction,
            python_template=data["python_template"],
            template_variables=data.get("template_variables", {}),
            imports=data.get("imports", []),
            output_behavior=data.get("output_behavior", "passthrough"),
            requires_ai=data.get("requires_ai", False),
            ai_prompt_hint=data.get("ai_prompt_hint", "")
        )
    
    def _parse_candidate(self, data: Dict) -> NodeMapping:
        """Parseia um candidato."""
        return NodeMapping(
            factory=data["factory"],
            name=data.get("name", "Unknown"),
            category=data.get("category", "unknown"),
            status="CANDIDATE",
            confidence=data.get("confidence", 0.8),
            complexity="high",
            settings_extraction={},
            python_template=data.get("generated_code", "# TODO: implement"),
            imports=data.get("imports_required", []),
            requires_ai=True
        )
    
    def get_type_mappings(self) -> Dict[str, str]:
        """Retorna mapeamento de tipos KNIME → Python."""
        mapping_file = self.config_dir / "node_mapping.json"
        if not mapping_file.exists():
            return {}
        
        with open(mapping_file, 'r', encoding='utf-8') as f:
            data = json.load(f)
        
        return data.get("type_mappings", {})
```

Gere:
1. config/node_mapping.json completo com os 25 nodes
2. src/mapping/classifier.py
3. src/mapping/mapping_loader.py
4. src/mapping/__init__.py
5. Testes em tests/test_classifier.py
```

---

## PROMPT 3.2: Tradutor Determinístico

```
Você é um Engenheiro de Software Python criando um tradutor de código.

TAREFA:
Criar o módulo que traduz nodes MAPPED para código Python usando templates.

ARQUIVO A CRIAR: src/mapping/deterministic_translator.py

```python
from dataclasses import dataclass
from typing import Dict, List, Any, Optional
from pathlib import Path
import re

from .mapping_loader import NodeMapping, SettingsExtraction


@dataclass
class TranslationResult:
    """Resultado da tradução de um node."""
    node_id: str
    success: bool
    python_code: str
    imports: List[str]
    errors: List[str]
    warnings: List[str]
    
    # Variáveis usadas
    input_vars: List[str]
    output_var: str


class DeterministicTranslator:
    """
    Traduz nodes MAPPED para código Python usando templates.
    
    Fluxo:
    1. Extrair parâmetros do settings usando extraction_rules
    2. Resolver variáveis de template (condicionais)
    3. Substituir placeholders no template
    4. Resolver referências de input/output (df_node_X)
    """
    
    def __init__(self):
        self._var_pattern = re.compile(r'\{(\w+)\}')
    
    def translate(
        self,
        node_id: str,
        mapping: NodeMapping,
        settings: Dict[str, Any],
        input_node_ids: List[str],
        output_var_name: str = None
    ) -> TranslationResult:
        """
        Traduz um node usando seu mapeamento.
        
        Args:
            node_id: ID do node sendo traduzido
            mapping: NodeMapping com template e regras
            settings: Settings extraídos do node (model)
            input_node_ids: IDs dos nodes predecessores (para df_{input_var})
            output_var_name: Nome da variável de saída (default: df_node_{id})
            
        Returns:
            TranslationResult com código gerado
        """
        errors = []
        warnings = []
        
        # 1. Extrair parâmetros
        params, extract_errors = self._extract_parameters(mapping, settings)
        errors.extend(extract_errors)
        
        # 2. Resolver variáveis de template
        template_vars = self._resolve_template_variables(mapping, params)
        params.update(template_vars)
        
        # 3. Adicionar variáveis de contexto
        output_var = output_var_name or f"node_{node_id}"
        input_vars = [f"node_{nid}" for nid in input_node_ids]
        
        params["output_var"] = output_var
        if input_vars:
            params["input_var"] = input_vars[0]  # Primeiro input
            for i, var in enumerate(input_vars):
                params[f"input_var_{i}"] = var
        
        # 4. Substituir placeholders no template
        code, sub_errors = self._substitute_template(mapping.python_template, params)
        errors.extend(sub_errors)
        
        # 5. Verificar placeholders não substituídos
        remaining = self._var_pattern.findall(code)
        if remaining:
            warnings.append(f"Placeholders não substituídos: {remaining}")
        
        return TranslationResult(
            node_id=node_id,
            success=len(errors) == 0,
            python_code=code,
            imports=mapping.imports.copy(),
            errors=errors,
            warnings=warnings,
            input_vars=input_vars,
            output_var=output_var
        )
    
    def _extract_parameters(
        self,
        mapping: NodeMapping,
        settings: Dict[str, Any]
    ) -> tuple[Dict[str, Any], List[str]]:
        """
        Extrai parâmetros do settings usando regras de extração.
        
        Returns:
            Tuple de (params dict, lista de erros)
        """
        params = {}
        errors = []
        
        for param_name, extraction in mapping.settings_extraction.items():
            value = self._get_nested_value(settings, extraction.path)
            
            if value is None:
                if extraction.required:
                    errors.append(f"Parâmetro obrigatório não encontrado: {param_name} (path: {extraction.path})")
                else:
                    value = extraction.default
            
            # Converter tipo se necessário
            value = self._convert_param_type(value, extraction.param_type)
            params[param_name] = value
        
        return params, errors
    
    def _get_nested_value(self, data: Dict, path: str) -> Any:
        """
        Busca valor em dict aninhado por caminho.
        
        Ex: "model/expression" busca data["model"]["expression"]
        """
        if not data or not path:
            return None
        
        parts = path.split("/")
        current = data
        
        for part in parts:
            if isinstance(current, dict):
                current = current.get(part)
            else:
                return None
            
            if current is None:
                return None
        
        return current
    
    def _convert_param_type(self, value: Any, param_type: str) -> Any:
        """Converte valor para tipo esperado."""
        if value is None:
            return None
        
        try:
            if param_type == "string":
                return str(value)
            elif param_type == "integer":
                return int(value)
            elif param_type == "float":
                return float(value)
            elif param_type == "boolean":
                if isinstance(value, bool):
                    return value
                return str(value).lower() in ("true", "1", "yes")
            elif param_type == "array":
                if isinstance(value, list):
                    return value
                return [value]
            else:
                return value
        except (ValueError, TypeError):
            return value
    
    def _resolve_template_variables(
        self,
        mapping: NodeMapping,
        params: Dict[str, Any]
    ) -> Dict[str, Any]:
        """
        Resolve variáveis de template condicionais.
        
        Ex: header_int depende de has_header ser true/false
        """
        resolved = {}
        
        for var_name, config in mapping.template_variables.items():
            condition_param = config.get("condition")
            
            if condition_param and condition_param in params:
                condition_value = params[condition_param]
                if condition_value:
                    resolved[var_name] = config.get("true_value", "")
                else:
                    resolved[var_name] = config.get("false_value", "")
            else:
                # Sem condição, usar valor default
                resolved[var_name] = config.get("default", "")
        
        return resolved
    
    def _substitute_template(
        self,
        template: str,
        params: Dict[str, Any]
    ) -> tuple[str, List[str]]:
        """
        Substitui placeholders no template.
        
        Returns:
            Tuple de (código gerado, lista de erros)
        """
        errors = []
        code = template
        
        # Tratar listas (ex: included_columns)
        for key, value in params.items():
            if isinstance(value, list):
                # Formatar como lista Python
                params[key] = repr(value)
        
        # Substituir placeholders
        try:
            # Usar format_map para permitir placeholders faltantes
            code = template.format_map(SafeDict(params))
        except KeyError as e:
            errors.append(f"Placeholder não encontrado: {e}")
        except Exception as e:
            errors.append(f"Erro na substituição: {e}")
        
        return code, errors


class SafeDict(dict):
    """Dict que retorna o próprio placeholder se chave não existe."""
    def __missing__(self, key):
        return "{" + key + "}"


def translate_node(
    node_id: str,
    mapping: NodeMapping,
    settings: Dict,
    input_node_ids: List[str]
) -> TranslationResult:
    """Função de conveniência."""
    translator = DeterministicTranslator()
    return translator.translate(node_id, mapping, settings, input_node_ids)
```

ADICIONAR ao src/mapping/__init__.py:
```python
from .classifier import NodeClassifier, Classification, ClassificationResult
from .mapping_loader import MappingLoader, NodeMapping, SettingsExtraction
from .deterministic_translator import DeterministicTranslator, TranslationResult, translate_node
```

TESTES A CRIAR em tests/test_translator.py:
- test_translate_csv_reader
- test_translate_column_filter
- test_translate_joiner
- test_extract_nested_parameters
- test_resolve_conditional_variables
- test_handle_missing_parameters
- test_list_formatting

Gere todos os arquivos completos.
```

---

# 📋 PROMPTS RESTANTES (RESUMO)

Os prompts das Sprints 4-6 seguem o mesmo padrão. Devido ao tamanho, aqui está um resumo:

## SPRINT 4: Integração com IA (Prompts 4.1-4.3)

- **4.1** - Prompt Builder: Construir prompts estruturados para Vertex AI
- **4.2** - Vertex Client: Cliente para chamar API do Google Vertex AI
- **4.3** - Response Validator: Validar código gerado pela IA

## SPRINT 5: Geração de Código (Prompts 5.1-5.2)

- **5.1** - Python Generator: Gerar arquivo .py completo
- **5.2** - Docstring Generator: Adicionar documentação de rastreabilidade

## SPRINT 6: Validação e Aprendizado (Prompts 6.1-6.3)

- **6.1** - Schema Comparator: Comparar output vs spec.xml
- **6.2** - Report Generator: Gerar relatórios de validação
- **6.3** - Learning Promoter: Promover candidatos validados

---

# 🔄 PROMPT FINAL: INTEGRAÇÃO COMPLETA

```
Você é um Engenheiro de Software Python finalizando a integração de um sistema.

TAREFA:
Atualizar main.py para executar o pipeline completo de conversão.

O pipeline deve:
1. Extrair workflow KNIME
2. Construir grafo e IR
3. Classificar nodes
4. Traduzir nodes MAPPED
5. (Opcional) Chamar IA para UNKNOWN
6. Gerar código Python
7. Validar contra spec.xml
8. Gerar relatórios

IMPLEMENTAÇÃO DE main.py COMPLETO:

[código detalhado do orquestrador]

IMPLEMENTAÇÃO DE cli.py COM COMANDOS:

- `convert <arquivo.knwf>` - Converte workflow completo
- `extract <arquivo.knwf>` - Apenas extrai (gera IR)
- `classify <workflow_ir.json>` - Classifica nodes
- `validate <generated.py> <workflow_ir.json>` - Valida código gerado
- `report <workflow_ir.json>` - Gera relatórios

Gere main.py e cli.py completos e funcionais.
```

---

## 📝 NOTAS DE USO

1. **Ordem de execução**: Execute os prompts na ordem (0, 1.1, 1.2, ...)
2. **Validação entre etapas**: Teste cada módulo antes de prosseguir
3. **Prompt 0 primeiro**: Execute em chat separado para extrair conhecimento do legado
4. **Ajustes necessários**: Adapte paths e configurações conforme seu ambiente

---

**Documento gerado em:** 2025-12-15  
**Versão:** 1.0  
**Para uso com:** Windsurf + Claude Opus 4.5
