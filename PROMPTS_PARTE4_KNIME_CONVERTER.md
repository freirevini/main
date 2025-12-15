# 📋 PROMPTS COMPLETOS - PARTE 4 (Sprints 4 a 7 Detalhados)

---

## SPRINT 4: INTEGRAÇÃO COM IA

<a name="prompt-41"></a>
### PROMPT 4.1 - Construtor de Prompts para IA

```
Crie o módulo de construção de prompts estruturados para interpretação por IA.

## ARQUIVO: src/ai/__init__.py

```python
"""
Módulo AI - Integração com Inteligência Artificial
==================================================

Este módulo gerencia a interpretação de nodes complexos via IA:
- PromptBuilder: Constrói prompts estruturados
- VertexAIClient: Cliente para Google Vertex AI
- ResponseValidator: Valida respostas da IA
- CandidateManager: Gerencia candidatos a mapeamento
"""

from .prompt_builder import PromptBuilder, PromptContext
from .vertex_client import VertexAIClient, MockVertexClient, AIResponse
from .response_validator import ResponseValidator
from .candidate_manager import CandidateManager

__all__ = [
    'PromptBuilder',
    'PromptContext',
    'VertexAIClient',
    'MockVertexClient',
    'AIResponse',
    'ResponseValidator',
    'CandidateManager'
]
```

## ARQUIVO: src/ai/prompt_builder.py

```python
"""
Construtor de Prompts - KNIME to Python Converter
=================================================

Constrói prompts estruturados para interpretação de nodes por IA.
Cada tipo de node complexo tem um template específico otimizado.

O prompt inclui:
- Contexto do node (factory, configurações)
- Schema de entrada/saída
- Exemplos de conversão quando disponíveis
- Restrições e formato esperado

Uso:
    builder = PromptBuilder()
    context = PromptContext(node_id="5", factory="...", settings={...})
    prompt = builder.build(context)
"""

from dataclasses import dataclass, field
from typing import Dict, Any, List, Optional
from string import Template
import json

from src.utils.logger import setup_logger

logger = setup_logger("PromptBuilder")


@dataclass
class PromptContext:
    """
    Contexto completo para construção do prompt.
    
    Attributes:
        node_id: ID do node no workflow
        node_name: Nome legível do node
        factory: Factory class completa
        annotation: Comentário/descrição do usuário
        settings: Dict com model do settings.xml
        input_schema: Lista de colunas de entrada [{name, type}, ...]
        output_schema: Lista de colunas de saída (se conhecida)
        predecessors: IDs dos nodes predecessores
        input_var: Nome da variável de entrada
        output_var: Nome da variável de saída
    """
    node_id: str
    node_name: str
    factory: str
    annotation: str = ""
    settings: Dict[str, Any] = field(default_factory=dict)
    input_schema: List[Dict[str, str]] = field(default_factory=list)
    output_schema: Optional[List[Dict[str, str]]] = None
    predecessors: List[str] = field(default_factory=list)
    input_var: str = "df_input"
    output_var: str = "df_output"


class PromptBuilder:
    """
    Constrói prompts estruturados para interpretação de nodes por IA.
    
    Seleciona template apropriado baseado no tipo de node e
    formata todas as informações de contexto.
    """
    
    # =========================================================================
    # SYSTEM PROMPT - Define comportamento geral da IA
    # =========================================================================
    SYSTEM_PROMPT = '''Você é um especialista em migração de workflows KNIME para Python/Pandas.

REGRAS OBRIGATÓRIAS:
1. Retorne APENAS código Python executável
2. NÃO inclua explicações, comentários longos ou markdown
3. Use operações VETORIZADAS do Pandas (evite loops for/while)
4. Preserve os tipos de dados das colunas
5. O código deve ser eficiente e legível
6. Use numpy para operações matemáticas complexas

FORMATO DA RESPOSTA:
```python
# Breve comentário explicando a transformação
df_output = <código>
```

NÃO use:
- Loops explícitos (for, while) quando há alternativa vetorizada
- apply() quando há método nativo do Pandas
- eval() ou exec()
- Imports além de pandas (pd) e numpy (np)'''

    # =========================================================================
    # TEMPLATE GENÉRICO - Para nodes sem template específico
    # =========================================================================
    GENERIC_TEMPLATE = Template('''
## TAREFA

Converta o seguinte node KNIME para código Python/Pandas equivalente.

## INFORMAÇÕES DO NODE

| Campo | Valor |
|-------|-------|
| **Tipo** | $factory |
| **Nome** | $node_name |
| **Descrição** | $annotation |

## CONFIGURAÇÕES DO NODE

```json
$settings_json
```

## SCHEMA DE ENTRADA

DataFrame de entrada: `$input_var`

| Coluna | Tipo Python |
|--------|-------------|
$input_schema_table

## REQUISITOS

1. Receber DataFrame `$input_var` com as colunas acima
2. Retornar DataFrame `$output_var` com a transformação aplicada
3. Preservar tipos de dados
4. Usar operações vetorizadas

## FORMATO DA RESPOSTA

```python
$output_var = <sua implementação>
```
''')

    # =========================================================================
    # TEMPLATE PARA RULE ENGINE
    # =========================================================================
    RULE_ENGINE_TEMPLATE = Template('''
## TAREFA

Converta as regras do KNIME Rule Engine para Python usando `np.select`.

## REGRAS KNIME

```
$rules
```

## CONFIGURAÇÕES

| Campo | Valor |
|-------|-------|
| Coluna de saída | `$output_column` |
| Modo | $append_mode |

## SCHEMA DE ENTRADA

DataFrame: `$input_var`

| Coluna | Tipo |
|--------|------|
$input_schema_table

## SINTAXE KNIME → PYTHON

| KNIME | Python |
|-------|--------|
| `$$coluna$$` | `df['coluna']` |
| `AND` | `&` (com parênteses) |
| `OR` | `\|` (com parênteses) |
| `NOT` | `~` |
| `MISSING $$col$$` | `df['col'].isna()` |
| `TRUE` | condição default |
| `= "texto"` | `== "texto"` |
| `<>` | `!=` |

## FORMATO OBRIGATÓRIO

Use `np.select` para múltiplas condições:

```python
import numpy as np

conditions = [
    (df['col1'] > 100) & (df['col2'] == 'A'),  # Condição 1
    (df['col1'] <= 100),                        # Condição 2
    # ... mais condições
]

choices = ['Resultado1', 'Resultado2']  # Resultados correspondentes

default = 'ValorDefault'

$output_var = $input_var.copy()
$output_var['$output_column'] = np.select(conditions, choices, default=default)
```

IMPORTANTE: Cada comparação deve estar entre parênteses quando usar & ou |
''')

    # =========================================================================
    # TEMPLATE PARA MATH FORMULA
    # =========================================================================
    MATH_FORMULA_TEMPLATE = Template('''
## TAREFA

Converta a fórmula matemática KNIME para Python/NumPy.

## FÓRMULA KNIME

```
$expression
```

## CONFIGURAÇÕES

| Campo | Valor |
|-------|-------|
| Coluna de saída | `$output_column` |
| Substituir coluna | $replace_mode |

## SCHEMA DE ENTRADA

DataFrame: `$input_var`

| Coluna | Tipo |
|--------|------|
$input_schema_table

## MAPEAMENTO DE FUNÇÕES

| KNIME | NumPy/Python |
|-------|--------------|
| `pow(a, b)` | `np.power(a, b)` |
| `sqrt(x)` | `np.sqrt(x)` |
| `abs(x)` | `np.abs(x)` |
| `exp(x)` | `np.exp(x)` |
| `log(x)` | `np.log(x)` |
| `log10(x)` | `np.log10(x)` |
| `round(x, n)` | `np.round(x, n)` |
| `floor(x)` | `np.floor(x)` |
| `ceil(x)` | `np.ceil(x)` |
| `min(a, b)` | `np.minimum(a, b)` |
| `max(a, b)` | `np.maximum(a, b)` |
| `if(cond, a, b)` | `np.where(cond, a, b)` |
| `$$coluna$$` | `df['coluna']` |

## FORMATO DA RESPOSTA

```python
import numpy as np

$output_var = $input_var.copy()
$output_var['$output_column'] = <expressão convertida>
```
''')

    # =========================================================================
    # TEMPLATE PARA STRING MANIPULATION
    # =========================================================================
    STRING_MANIPULATION_TEMPLATE = Template('''
## TAREFA

Converta a manipulação de string KNIME para Python/Pandas.

## EXPRESSÃO KNIME

```
$expression
```

## CONFIGURAÇÕES

| Campo | Valor |
|-------|-------|
| Coluna de saída | `$output_column` |
| Modo | $append_mode |

## SCHEMA DE ENTRADA

DataFrame: `$input_var`

| Coluna | Tipo |
|--------|------|
$input_schema_table

## MAPEAMENTO DE FUNÇÕES STRING

| KNIME | Pandas |
|-------|--------|
| `substr(s, start, len)` | `s.str[start:start+len]` |
| `length(s)` | `s.str.len()` |
| `upperCase(s)` | `s.str.upper()` |
| `lowerCase(s)` | `s.str.lower()` |
| `capitalize(s)` | `s.str.capitalize()` |
| `trim(s)` | `s.str.strip()` |
| `replace(s, old, new)` | `s.str.replace(old, new)` |
| `indexOf(s, sub)` | `s.str.find(sub)` |
| `contains(s, sub)` | `s.str.contains(sub)` |
| `join(sep, s1, s2)` | `s1 + sep + s2` |
| `string(x)` | `x.astype(str)` |
| `toInt(s)` | `pd.to_numeric(s, errors='coerce').astype('Int64')` |
| `toDouble(s)` | `pd.to_numeric(s, errors='coerce')` |
| `$$coluna$$` | `df['coluna']` |

## FORMATO DA RESPOSTA

```python
$output_var = $input_var.copy()
$output_var['$output_column'] = <expressão convertida>
```
''')

    # =========================================================================
    # TEMPLATE PARA JAVA SNIPPET
    # =========================================================================
    JAVA_SNIPPET_TEMPLATE = Template('''
## TAREFA

Converta o Java Snippet KNIME para Python/Pandas equivalente.

## CÓDIGO JAVA

```java
$java_code
```

## COLUNAS DE ENTRADA

$input_columns_list

## COLUNAS DE SAÍDA

$output_columns_list

## SCHEMA DE ENTRADA

DataFrame: `$input_var`

| Coluna | Tipo |
|--------|------|
$input_schema_table

## MAPEAMENTO JAVA → PYTHON

| Java | Python |
|------|--------|
| `c_coluna` (input) | `df['coluna']` |
| `String.valueOf(x)` | `str(x)` |
| `Integer.parseInt(s)` | `int(s)` |
| `Double.parseDouble(s)` | `float(s)` |
| `s.substring(i, j)` | `s[i:j]` |
| `s.length()` | `len(s)` |
| `s.toUpperCase()` | `s.upper()` |
| `s.toLowerCase()` | `s.lower()` |
| `s.trim()` | `s.strip()` |
| `s.replace(a, b)` | `s.replace(a, b)` |
| `s.contains(x)` | `x in s` |
| `Math.pow(a, b)` | `np.power(a, b)` |
| `Math.sqrt(x)` | `np.sqrt(x)` |
| `null` | `None` ou `np.nan` |

## FORMATO DA RESPOSTA

```python
import numpy as np

$output_var = $input_var.copy()
# Implementação da lógica do Java Snippet
```

IMPORTANTE: Se o Java usa loops ou condicionais complexos, use apply() com uma função lambda ou vectorize com numpy.
''')

    # =========================================================================
    # TEMPLATE PARA GROUPBY COMPLEXO
    # =========================================================================
    GROUPBY_TEMPLATE = Template('''
## TAREFA

Converta o GroupBy KNIME para Python/Pandas.

## CONFIGURAÇÕES

### Colunas de Agrupamento
$group_columns

### Agregações
$aggregations_list

## SCHEMA DE ENTRADA

DataFrame: `$input_var`

| Coluna | Tipo |
|--------|------|
$input_schema_table

## MAPEAMENTO DE AGREGAÇÕES

| KNIME | Pandas |
|-------|--------|
| Sum | `sum` |
| Mean | `mean` |
| Count | `count` |
| Min | `min` |
| Max | `max` |
| First | `first` |
| Last | `last` |
| List | `list` |
| Unique count | `nunique` |
| Standard deviation | `std` |
| Variance | `var` |
| Median | `median` |
| Concatenate | `lambda x: ','.join(x.astype(str))` |

## FORMATO DA RESPOSTA

```python
agg_dict = {
    'coluna1': 'sum',
    'coluna2': 'mean',
    # ... mais agregações
}

$output_var = $input_var.groupby($group_columns).agg(agg_dict).reset_index()

# Se precisar renomear colunas:
# $output_var.columns = ['col1', 'col2_sum', 'col3_mean']
```
''')

    def __init__(self, max_tokens: int = 4000):
        """
        Inicializa o construtor de prompts.
        
        Args:
            max_tokens: Limite aproximado de tokens no prompt
        """
        self.max_tokens = max_tokens
        
        # Mapeamento de factory para template
        self._template_map = {
            "RuleEngine": self.RULE_ENGINE_TEMPLATE,
            "JEP": self.MATH_FORMULA_TEMPLATE,
            "MathFormula": self.MATH_FORMULA_TEMPLATE,
            "StringManipulation": self.STRING_MANIPULATION_TEMPLATE,
            "JavaSnippet": self.JAVA_SNIPPET_TEMPLATE,
            "GroupBy": self.GROUPBY_TEMPLATE,
        }
    
    def build(self, context: PromptContext) -> str:
        """
        Constrói prompt completo para o node.
        
        Seleciona template apropriado baseado no tipo de node.
        
        Args:
            context: PromptContext com informações do node
            
        Returns:
            Prompt formatado
        """
        # Seleciona template
        template = self._select_template(context.factory, context.settings)
        
        # Prepara parâmetros
        params = self._prepare_params(context)
        
        # Aplica template
        try:
            prompt = template.safe_substitute(params)
        except Exception as e:
            logger.warning(f"Erro ao aplicar template: {e}")
            prompt = self.GENERIC_TEMPLATE.safe_substitute(params)
        
        # Trunca se necessário
        prompt = self._truncate_if_needed(prompt)
        
        return prompt
    
    def _select_template(self, factory: str, settings: Dict) -> Template:
        """
        Seleciona template mais apropriado para o node.
        
        Args:
            factory: Factory class
            settings: Configurações do node
            
        Returns:
            Template apropriado
        """
        # Busca por padrões no factory
        factory_lower = factory.lower()
        
        for pattern, template in self._template_map.items():
            if pattern.lower() in factory_lower:
                return template
        
        return self.GENERIC_TEMPLATE
    
    def _prepare_params(self, context: PromptContext) -> Dict[str, str]:
        """
        Prepara parâmetros para substituição no template.
        
        Args:
            context: PromptContext
            
        Returns:
            Dict com parâmetros formatados
        """
        params = {
            "node_id": context.node_id,
            "node_name": context.node_name,
            "factory": context.factory,
            "annotation": context.annotation or "Não especificada",
            "input_var": context.input_var,
            "output_var": context.output_var,
        }
        
        # Formata settings como JSON
        params["settings_json"] = self._format_settings(context.settings)
        
        # Formata schema de entrada como tabela
        params["input_schema_table"] = self._format_schema_table(context.input_schema)
        
        # Extrai campos específicos do settings
        settings = context.settings
        
        # Para Rule Engine
        if "rules" in settings:
            rules = settings["rules"]
            if isinstance(rules, list):
                params["rules"] = "\n".join(rules)
            else:
                params["rules"] = str(rules)
        
        params["output_column"] = settings.get("new-column-name", 
                                               settings.get("replaced_column", "Result"))
        params["append_mode"] = "Adicionar coluna" if settings.get("append-column", True) else "Substituir coluna"
        params["replace_mode"] = "Não" if settings.get("append_column", False) else "Sim"
        
        # Para Math Formula
        params["expression"] = settings.get("expression", "")
        
        # Para Java Snippet
        params["java_code"] = settings.get("script", settings.get("expression", ""))
        params["input_columns_list"] = ", ".join(
            [col.get("name", "") for col in context.input_schema]
        )
        params["output_columns_list"] = params["output_column"]
        
        # Para GroupBy
        params["group_columns"] = str(settings.get("grpCols", []))
        params["aggregations_list"] = self._format_aggregations(
            settings.get("aggregationMethods", [])
        )
        
        return params
    
    def _format_settings(self, settings: Dict) -> str:
        """Formata settings como JSON legível."""
        try:
            # Remove campos muito grandes
            filtered = {k: v for k, v in settings.items() 
                       if not isinstance(v, (bytes, bytearray))}
            return json.dumps(filtered, indent=2, ensure_ascii=False, default=str)
        except Exception:
            return str(settings)
    
    def _format_schema_table(self, schema: List[Dict]) -> str:
        """Formata schema como tabela Markdown."""
        if not schema:
            return "| (sem schema disponível) | - |"
        
        lines = []
        for col in schema[:20]:  # Limita a 20 colunas
            name = col.get("name", "?")
            col_type = col.get("python_type", col.get("type", "object"))
            lines.append(f"| {name} | {col_type} |")
        
        if len(schema) > 20:
            lines.append(f"| ... (+{len(schema) - 20} colunas) | ... |")
        
        return "\n".join(lines)
    
    def _format_aggregations(self, aggregations: Any) -> str:
        """Formata lista de agregações."""
        if not aggregations:
            return "Não especificadas"
        
        if isinstance(aggregations, list):
            lines = []
            for agg in aggregations:
                if isinstance(agg, dict):
                    col = agg.get("column", "?")
                    method = agg.get("method", "?")
                    lines.append(f"- {col}: {method}")
                else:
                    lines.append(f"- {agg}")
            return "\n".join(lines)
        
        return str(aggregations)
    
    def _estimate_tokens(self, text: str) -> int:
        """Estima número de tokens (aproximado: 4 chars = 1 token)."""
        return len(text) // 4
    
    def _truncate_if_needed(self, prompt: str) -> str:
        """Trunca prompt se exceder limite de tokens."""
        estimated = self._estimate_tokens(prompt)
        
        if estimated <= self.max_tokens:
            return prompt
        
        # Trunca settings JSON que geralmente é a parte maior
        lines = prompt.split("\n")
        truncated_lines = []
        current_tokens = 0
        in_json = False
        json_lines = 0
        
        for line in lines:
            if "```json" in line:
                in_json = True
            elif "```" in line and in_json:
                in_json = False
                if json_lines > 20:
                    truncated_lines.append("... (truncado)")
            
            if in_json:
                json_lines += 1
                if json_lines > 20:
                    continue
            
            truncated_lines.append(line)
            current_tokens += self._estimate_tokens(line)
            
            if current_tokens > self.max_tokens:
                truncated_lines.append("\n... (prompt truncado)")
                break
        
        return "\n".join(truncated_lines)
    
    def get_system_prompt(self) -> str:
        """Retorna system prompt padrão."""
        return self.SYSTEM_PROMPT
    
    def build_batch(self, contexts: List[PromptContext]) -> List[str]:
        """Constrói prompts para múltiplos nodes."""
        return [self.build(ctx) for ctx in contexts]
```

## ARQUIVO: tests/test_prompt_builder.py

```python
"""
Testes - PromptBuilder
"""

import pytest
from src.ai.prompt_builder import PromptBuilder, PromptContext


class TestPromptBuilder:
    
    @pytest.fixture
    def builder(self):
        return PromptBuilder()
    
    @pytest.fixture
    def rule_engine_context(self):
        return PromptContext(
            node_id="5",
            node_name="Rule Engine",
            factory="org.knime.base.node.rules.engine.RuleEngineNodeFactory",
            settings={
                "rules": [
                    '$Valor$ > 1000 => "Alto"',
                    '$Valor$ <= 1000 => "Baixo"',
                    'TRUE => "Indefinido"'
                ],
                "new-column-name": "Classificacao",
                "append-column": True
            },
            input_schema=[
                {"name": "Valor", "python_type": "Float64"},
                {"name": "Nome", "python_type": "str"}
            ]
        )
    
    def test_build_generic_node(self, builder):
        context = PromptContext(
            node_id="1",
            node_name="Unknown Node",
            factory="com.example.UnknownNodeFactory",
            settings={"param": "value"}
        )
        
        prompt = builder.build(context)
        
        assert "Unknown Node" in prompt
        assert "com.example.UnknownNodeFactory" in prompt
    
    def test_build_rule_engine(self, builder, rule_engine_context):
        prompt = builder.build(rule_engine_context)
        
        assert "Rule Engine" in prompt or "np.select" in prompt
        assert "Valor" in prompt
        assert "Classificacao" in prompt
    
    def test_format_schema_table(self, builder):
        schema = [
            {"name": "Col1", "python_type": "Float64"},
            {"name": "Col2", "python_type": "str"}
        ]
        
        table = builder._format_schema_table(schema)
        
        assert "Col1" in table
        assert "Float64" in table
    
    def test_system_prompt_exists(self, builder):
        system = builder.get_system_prompt()
        
        assert "Python" in system
        assert "KNIME" in system
    
    def test_truncate_long_prompt(self, builder):
        # Cria contexto com settings muito grande
        context = PromptContext(
            node_id="1",
            node_name="Test",
            factory="test.Factory",
            settings={"data": "x" * 50000}
        )
        
        builder.max_tokens = 1000
        prompt = builder.build(context)
        
        # Deve ter truncado
        assert len(prompt) < 50000
```

## VALIDAÇÃO

```bash
python -m py_compile src/ai/prompt_builder.py
pytest tests/test_prompt_builder.py -v
```
```

---

<a name="prompt-42"></a>
### PROMPT 4.2 - Cliente Vertex AI

```
Crie o cliente para chamadas à API do Google Vertex AI.

## ARQUIVO: src/ai/vertex_client.py

```python
"""
Cliente Vertex AI - KNIME to Python Converter
=============================================

Cliente para Google Vertex AI (Gemini) com:
- Retry automático com backoff exponencial
- Extração de código da resposta
- Logging detalhado
- Mock para testes

Pré-requisitos:
1. pip install google-cloud-aiplatform
2. Configurar credenciais GCP (gcloud auth application-default login)
3. Ter projeto GCP com Vertex AI habilitado

Uso:
    client = VertexAIClient(project_id="meu-projeto")
    response = client.generate(prompt, system_prompt)
    print(response.code)
"""

from dataclasses import dataclass
from typing import Dict, Any, Optional, List
import time
import re
import logging

from src.utils.logger import setup_logger

logger = setup_logger("VertexAIClient")

# Tenta importar bibliotecas do Google Cloud
try:
    from google.cloud import aiplatform
    import vertexai
    from vertexai.generative_models import GenerativeModel, GenerationConfig
    HAS_VERTEX = True
except ImportError:
    HAS_VERTEX = False
    logger.warning(
        "google-cloud-aiplatform não instalado. "
        "Execute: pip install google-cloud-aiplatform"
    )


@dataclass
class AIResponse:
    """
    Resposta processada da IA.
    
    Attributes:
        code: Código Python extraído
        raw_response: Resposta completa da IA
        model: Modelo utilizado
        tokens_used: Tokens consumidos (aproximado)
        latency_ms: Tempo de resposta em ms
        success: Se a chamada foi bem sucedida
        error: Mensagem de erro se houver
    """
    code: str
    raw_response: str
    model: str
    tokens_used: int
    latency_ms: int
    success: bool
    error: Optional[str] = None


class VertexAIClient:
    """
    Cliente para Google Vertex AI (Gemini).
    
    Attributes:
        project_id: ID do projeto GCP
        location: Região do Vertex AI
        model_name: Nome do modelo (default: gemini-1.5-pro)
    """
    
    DEFAULT_MODEL = "gemini-1.5-pro"
    DEFAULT_LOCATION = "us-central1"
    DEFAULT_TIMEOUT = 60
    MAX_RETRIES = 3
    RETRY_DELAY = 2
    
    def __init__(
        self,
        project_id: Optional[str] = None,
        location: str = DEFAULT_LOCATION,
        model: Optional[str] = None
    ):
        """
        Inicializa o cliente.
        
        Args:
            project_id: ID do projeto GCP (usa default se None)
            location: Região do Vertex AI
            model: Modelo a usar (default: gemini-1.5-pro)
        """
        if not HAS_VERTEX:
            raise ImportError(
                "google-cloud-aiplatform não instalado. "
                "Execute: pip install google-cloud-aiplatform"
            )
        
        self.project_id = project_id
        self.location = location
        self.model_name = model or self.DEFAULT_MODEL
        self._model = None
        
        self._initialize()
    
    def _initialize(self):
        """Inicializa conexão com Vertex AI."""
        try:
            vertexai.init(
                project=self.project_id,
                location=self.location
            )
            
            self._model = GenerativeModel(self.model_name)
            
            logger.info(
                f"Vertex AI inicializado: "
                f"projeto={self.project_id}, modelo={self.model_name}"
            )
        except Exception as e:
            logger.error(f"Erro ao inicializar Vertex AI: {e}")
            raise
    
    def generate(
        self,
        prompt: str,
        system_prompt: Optional[str] = None,
        temperature: float = 0.1,
        max_output_tokens: int = 2048
    ) -> AIResponse:
        """
        Gera resposta da IA.
        
        Args:
            prompt: Prompt principal com a tarefa
            system_prompt: Instrução de comportamento
            temperature: 0.0-1.0 (menor = mais determinístico)
            max_output_tokens: Máximo de tokens na resposta
            
        Returns:
            AIResponse com código gerado
        """
        start_time = time.time()
        
        # Configura geração
        generation_config = GenerationConfig(
            temperature=temperature,
            max_output_tokens=max_output_tokens,
            top_p=0.95,
            top_k=40
        )
        
        # Monta prompt completo
        full_prompt = prompt
        if system_prompt:
            full_prompt = f"{system_prompt}\n\n---\n\n{prompt}"
        
        try:
            # Chama API com retry
            response = self._call_with_retry(full_prompt, generation_config)
            
            # Extrai código
            raw_text = response.text
            code = self._extract_code(raw_text)
            
            latency = int((time.time() - start_time) * 1000)
            
            return AIResponse(
                code=code,
                raw_response=raw_text,
                model=self.model_name,
                tokens_used=self._estimate_tokens(full_prompt + raw_text),
                latency_ms=latency,
                success=True
            )
            
        except Exception as e:
            latency = int((time.time() - start_time) * 1000)
            logger.error(f"Erro na geração: {e}")
            
            return AIResponse(
                code="",
                raw_response="",
                model=self.model_name,
                tokens_used=0,
                latency_ms=latency,
                success=False,
                error=str(e)
            )
    
    def _call_with_retry(
        self,
        prompt: str,
        config: GenerationConfig
    ) -> Any:
        """
        Chama API com retry e backoff exponencial.
        
        Args:
            prompt: Prompt completo
            config: Configuração de geração
            
        Returns:
            Resposta do modelo
        """
        last_error = None
        
        for attempt in range(self.MAX_RETRIES):
            try:
                response = self._model.generate_content(
                    prompt,
                    generation_config=config
                )
                return response
                
            except Exception as e:
                last_error = e
                wait_time = self.RETRY_DELAY * (2 ** attempt)
                
                logger.warning(
                    f"Tentativa {attempt + 1}/{self.MAX_RETRIES} falhou: {e}. "
                    f"Aguardando {wait_time}s..."
                )
                
                time.sleep(wait_time)
        
        raise last_error
    
    def _extract_code(self, response_text: str) -> str:
        """
        Extrai código Python da resposta.
        
        Remove blocos markdown e texto extra.
        
        Args:
            response_text: Texto completo da resposta
            
        Returns:
            Código Python limpo
        """
        text = response_text.strip()
        
        # Tenta extrair de bloco ```python
        pattern = r'```python\s*(.*?)\s*```'
        matches = re.findall(pattern, text, re.DOTALL)
        
        if matches:
            # Retorna o maior bloco encontrado
            return max(matches, key=len).strip()
        
        # Tenta extrair de bloco ``` genérico
        pattern = r'```\s*(.*?)\s*```'
        matches = re.findall(pattern, text, re.DOTALL)
        
        if matches:
            return max(matches, key=len).strip()
        
        # Se não encontrou blocos, verifica se é código direto
        lines = text.split('\n')
        code_lines = []
        in_code = False
        
        for line in lines:
            # Detecta início de código por indentação ou keywords
            if (line.startswith('import ') or 
                line.startswith('from ') or
                line.startswith('df') or
                line.startswith('#') or
                (line.startswith('    ') and in_code)):
                in_code = True
                code_lines.append(line)
            elif in_code and line.strip():
                code_lines.append(line)
            elif in_code and not line.strip():
                code_lines.append(line)
        
        if code_lines:
            return '\n'.join(code_lines).strip()
        
        # Fallback: retorna texto original
        return text
    
    def _estimate_tokens(self, text: str) -> int:
        """Estima tokens (aproximado: 4 chars = 1 token)."""
        return len(text) // 4
    
    def test_connection(self) -> bool:
        """
        Testa se conexão está funcionando.
        
        Returns:
            True se conectou com sucesso
        """
        try:
            response = self.generate(
                "Responda apenas: OK",
                temperature=0
            )
            return response.success
        except Exception as e:
            logger.error(f"Teste de conexão falhou: {e}")
            return False


class MockVertexClient:
    """
    Cliente mock para testes sem Vertex AI real.
    
    Retorna respostas pré-definidas baseadas no tipo de node.
    Útil para testes unitários e desenvolvimento offline.
    """
    
    def __init__(self):
        self.call_history: List[Dict] = []
    
    def generate(
        self,
        prompt: str,
        system_prompt: Optional[str] = None,
        **kwargs
    ) -> AIResponse:
        """
        Retorna resposta mock baseada no conteúdo do prompt.
        """
        # Registra chamada
        self.call_history.append({
            "prompt": prompt,
            "system_prompt": system_prompt,
            "kwargs": kwargs,
            "timestamp": time.time()
        })
        
        # Detecta tipo de node e retorna mock apropriado
        prompt_lower = prompt.lower()
        
        if "rule engine" in prompt_lower or "np.select" in prompt_lower:
            code = self._mock_rule_engine()
        elif "math formula" in prompt_lower or "expressão" in prompt_lower:
            code = self._mock_math_formula()
        elif "string manipulation" in prompt_lower:
            code = self._mock_string_manipulation()
        elif "groupby" in prompt_lower or "agregação" in prompt_lower:
            code = self._mock_groupby()
        else:
            code = self._mock_generic()
        
        return AIResponse(
            code=code,
            raw_response=f"```python\n{code}\n```",
            model="mock-model",
            tokens_used=len(prompt) // 4 + len(code) // 4,
            latency_ms=50,
            success=True
        )
    
    def _mock_rule_engine(self) -> str:
        return '''import numpy as np

conditions = [
    (df['Valor'] > 1000),
    (df['Valor'] <= 1000)
]
choices = ['Alto', 'Baixo']
default = 'Indefinido'

df_output = df_input.copy()
df_output['Classificacao'] = np.select(conditions, choices, default=default)'''
    
    def _mock_math_formula(self) -> str:
        return '''import numpy as np

df_output = df_input.copy()
df_output['Resultado'] = df_input['VrPrestacao'] / np.power(1 + df_input['Taxa']/100, df_input['Prazo'])'''
    
    def _mock_string_manipulation(self) -> str:
        return '''df_output = df_input.copy()
df_output['NomeFormatado'] = df_input['Nome'].str.upper().str.strip()'''
    
    def _mock_groupby(self) -> str:
        return '''df_output = df_input.groupby(['Categoria']).agg({
    'Valor': 'sum',
    'Quantidade': 'count'
}).reset_index()'''
    
    def _mock_generic(self) -> str:
        return '''# Transformação genérica
df_output = df_input.copy()
# TODO: Implementar lógica específica'''
    
    def get_call_count(self) -> int:
        """Retorna quantidade de chamadas realizadas."""
        return len(self.call_history)
    
    def clear_history(self):
        """Limpa histórico de chamadas."""
        self.call_history.clear()
    
    def test_connection(self) -> bool:
        """Mock sempre retorna True."""
        return True
```

## ARQUIVO: tests/test_vertex_client.py

```python
"""
Testes - VertexAIClient (usando Mock)
"""

import pytest
from src.ai.vertex_client import MockVertexClient, AIResponse


class TestMockVertexClient:
    
    @pytest.fixture
    def client(self):
        return MockVertexClient()
    
    def test_generate_returns_response(self, client):
        response = client.generate("Teste simples")
        
        assert isinstance(response, AIResponse)
        assert response.success is True
        assert response.code != ""
    
    def test_rule_engine_detection(self, client):
        response = client.generate("Converta Rule Engine com np.select")
        
        assert "np.select" in response.code
        assert "conditions" in response.code
    
    def test_math_formula_detection(self, client):
        response = client.generate("Math Formula: expressão matemática")
        
        assert "np.power" in response.code or "df_output" in response.code
    
    def test_call_history(self, client):
        client.generate("Primeiro prompt")
        client.generate("Segundo prompt")
        
        assert client.get_call_count() == 2
        assert "Primeiro" in client.call_history[0]["prompt"]
    
    def test_clear_history(self, client):
        client.generate("Teste")
        client.clear_history()
        
        assert client.get_call_count() == 0
    
    def test_connection(self, client):
        assert client.test_connection() is True
    
    def test_extract_code_in_response(self, client):
        response = client.generate("Qualquer coisa")
        
        # Código não deve ter markdown
        assert "```" not in response.code
```

## VALIDAÇÃO

```bash
python -m py_compile src/ai/vertex_client.py
pytest tests/test_vertex_client.py -v
```
```

---

<a name="prompt-43"></a>
### PROMPT 4.3 - Validador de Respostas e Gerenciador de Candidatos

```
Crie o validador de respostas da IA e gerenciador de candidatos.

## ARQUIVO: src/ai/response_validator.py

```python
"""
Validador de Respostas - KNIME to Python Converter
==================================================

Valida código Python gerado pela IA antes de aceitar:
- Sintaxe Python válida
- Código seguro (sem imports perigosos)
- Variáveis esperadas definidas
- Formato correto

Uso:
    validator = ResponseValidator()
    is_valid, errors = validator.validate(code, context)
    
    if is_valid:
        # Código pode ser usado
    else:
        print(f"Erros: {errors}")
"""

from typing import Dict, Any, List, Optional, Tuple, Set
import ast
import re

from src.utils.logger import setup_logger

logger = setup_logger("ResponseValidator")


class ResponseValidator:
    """
    Valida respostas da IA antes de aceitar como candidato.
    """
    
    # Imports e funções bloqueadas por segurança
    BLOCKED_IMPORTS = {
        'os', 'sys', 'subprocess', 'shutil', 'pathlib',
        'socket', 'requests', 'urllib', 'http',
        'pickle', 'marshal', 'shelve',
        'code', 'codeop', 'compile',
        '__builtins__'
    }
    
    BLOCKED_FUNCTIONS = {
        'eval', 'exec', 'compile', 'open', 'input',
        '__import__', 'getattr', 'setattr', 'delattr',
        'globals', 'locals', 'vars'
    }
    
    # Imports permitidos
    ALLOWED_IMPORTS = {
        'pandas', 'pd', 'numpy', 'np', 're', 'datetime',
        'math', 'collections', 'itertools', 'functools',
        'typing', 'dataclasses'
    }
    
    def __init__(self):
        self.validation_errors: List[str] = []
    
    def validate(
        self, 
        code: str, 
        context: Optional[Dict] = None
    ) -> Tuple[bool, List[str]]:
        """
        Valida código gerado pela IA.
        
        Verificações:
        1. Sintaxe Python válida
        2. Não contém código perigoso
        3. Usa variáveis esperadas
        4. Define output esperado
        
        Args:
            code: Código Python a validar
            context: Contexto com input_var, output_var esperados
            
        Returns:
            Tuple (is_valid, list_of_errors)
        """
        self.validation_errors = []
        context = context or {}
        
        # 1. Verifica sintaxe
        if not self._check_syntax(code):
            return False, self.validation_errors
        
        # 2. Verifica segurança
        if not self._check_safety(code):
            return False, self.validation_errors
        
        # 3. Verifica variáveis (se contexto fornecido)
        if context:
            if not self._check_variables(code, context):
                return False, self.validation_errors
            
            if not self._check_output(code, context):
                return False, self.validation_errors
        
        return len(self.validation_errors) == 0, self.validation_errors
    
    def _check_syntax(self, code: str) -> bool:
        """
        Verifica se código é sintaticamente válido.
        
        Args:
            code: Código Python
            
        Returns:
            True se válido
        """
        try:
            ast.parse(code)
            return True
        except SyntaxError as e:
            self.validation_errors.append(
                f"Erro de sintaxe na linha {e.lineno}: {e.msg}"
            )
            return False
    
    def _check_safety(self, code: str) -> bool:
        """
        Verifica se código é seguro.
        
        Bloqueia:
        - Imports perigosos
        - Funções perigosas (eval, exec, etc)
        - Acesso a arquivos
        
        Args:
            code: Código Python
            
        Returns:
            True se seguro
        """
        is_safe = True
        
        try:
            tree = ast.parse(code)
            
            for node in ast.walk(tree):
                # Verifica imports
                if isinstance(node, ast.Import):
                    for alias in node.names:
                        module = alias.name.split('.')[0]
                        if module in self.BLOCKED_IMPORTS:
                            self.validation_errors.append(
                                f"Import bloqueado: {alias.name}"
                            )
                            is_safe = False
                
                elif isinstance(node, ast.ImportFrom):
                    if node.module:
                        module = node.module.split('.')[0]
                        if module in self.BLOCKED_IMPORTS:
                            self.validation_errors.append(
                                f"Import bloqueado: {node.module}"
                            )
                            is_safe = False
                
                # Verifica chamadas de função
                elif isinstance(node, ast.Call):
                    if isinstance(node.func, ast.Name):
                        if node.func.id in self.BLOCKED_FUNCTIONS:
                            self.validation_errors.append(
                                f"Função bloqueada: {node.func.id}()"
                            )
                            is_safe = False
                    
                    # Verifica __import__
                    elif isinstance(node.func, ast.Attribute):
                        if node.func.attr in self.BLOCKED_FUNCTIONS:
                            self.validation_errors.append(
                                f"Função bloqueada: {node.func.attr}()"
                            )
                            is_safe = False
        
        except Exception as e:
            self.validation_errors.append(f"Erro ao analisar segurança: {e}")
            is_safe = False
        
        return is_safe
    
    def _check_variables(self, code: str, context: Dict) -> bool:
        """
        Verifica se usa variáveis de entrada esperadas.
        
        Args:
            code: Código Python
            context: Dict com input_var
            
        Returns:
            True se usa variáveis corretas
        """
        input_var = context.get("input_var", "df_input")
        
        # Verifica se input_var é usado
        if input_var not in code and "df_input" not in code:
            # Pode usar variação como df_node_X
            if not re.search(r'df_\w+', code):
                self.validation_errors.append(
                    f"Variável de entrada '{input_var}' não utilizada"
                )
                return False
        
        return True
    
    def _check_output(self, code: str, context: Dict) -> bool:
        """
        Verifica se define a variável de saída esperada.
        
        Args:
            code: Código Python
            context: Dict com output_var
            
        Returns:
            True se define output
        """
        output_var = context.get("output_var", "df_output")
        
        # Verifica se output_var é definido
        if f"{output_var} =" not in code and f"{output_var}=" not in code:
            # Pode usar variação
            if not re.search(r'df_\w+\s*=', code):
                self.validation_errors.append(
                    f"Variável de saída '{output_var}' não definida"
                )
                return False
        
        return True
    
    def extract_imports(self, code: str) -> List[str]:
        """
        Extrai imports do código.
        
        Args:
            code: Código Python
            
        Returns:
            Lista de imports
        """
        imports = []
        
        try:
            tree = ast.parse(code)
            
            for node in ast.walk(tree):
                if isinstance(node, ast.Import):
                    for alias in node.names:
                        if alias.asname:
                            imports.append(f"import {alias.name} as {alias.asname}")
                        else:
                            imports.append(f"import {alias.name}")
                
                elif isinstance(node, ast.ImportFrom):
                    module = node.module or ""
                    names = [
                        alias.asname if alias.asname else alias.name
                        for alias in node.names
                    ]
                    imports.append(f"from {module} import {', '.join(names)}")
        
        except Exception:
            pass
        
        return imports
    
    def clean_code(self, code: str) -> str:
        """
        Limpa código removendo elementos desnecessários.
        
        Remove:
        - Blocos markdown
        - Comentários longos de explicação
        - Linhas em branco extras
        
        Args:
            code: Código bruto
            
        Returns:
            Código limpo
        """
        # Remove blocos markdown
        code = re.sub(r'```python\s*', '', code)
        code = re.sub(r'```\s*', '', code)
        
        # Remove linhas com apenas comentários longos de explicação
        lines = code.split('\n')
        cleaned_lines = []
        
        for line in lines:
            stripped = line.strip()
            
            # Mantém comentários curtos (provavelmente úteis)
            if stripped.startswith('#'):
                if len(stripped) < 80:
                    cleaned_lines.append(line)
            else:
                cleaned_lines.append(line)
        
        # Remove linhas em branco consecutivas
        result = []
        prev_empty = False
        
        for line in cleaned_lines:
            is_empty = not line.strip()
            
            if is_empty and prev_empty:
                continue
            
            result.append(line)
            prev_empty = is_empty
        
        return '\n'.join(result).strip()
```

## ARQUIVO: src/ai/candidate_manager.py

```python
"""
Gerenciador de Candidatos - KNIME to Python Converter
=====================================================

Gerencia candidatos a mapeamento gerados pela IA.
Candidatos são armazenados em candidates.json e passam por validação
antes de serem promovidos a mapeamentos oficiais.

Ciclo de vida:
1. IA gera código para node UNKNOWN
2. Código é validado (sintaxe, segurança)
3. Candidato é criado e salvo
4. Em conversões futuras, candidato é testado
5. Se passar em N validações, é promovido para node_mapping.json

Uso:
    manager = CandidateManager()
    candidate_id = manager.add_candidate(factory, code, context, workflow)
    manager.record_validation(candidate_id, workflow, "PASS")
    
    if manager.check_promotion_ready(candidate_id):
        # Candidato pode ser promovido
        pass
"""

from typing import Dict, Any, List, Optional
from pathlib import Path
from datetime import datetime
import json
import hashlib
import uuid

from src.utils.logger import setup_logger

logger = setup_logger("CandidateManager")


class CandidateManager:
    """
    Gerencia candidatos a mapeamento gerados pela IA.
    """
    
    # Critérios para promoção
    DEFAULT_PROMOTION_CRITERIA = {
        "min_validations": 3,
        "min_pass_rate": 1.0,
        "min_workflows": 2
    }
    
    def __init__(
        self, 
        candidates_path: Optional[Path] = None,
        promotion_criteria: Optional[Dict] = None
    ):
        """
        Inicializa o gerenciador.
        
        Args:
            candidates_path: Caminho para candidates.json
            promotion_criteria: Critérios customizados de promoção
        """
        self.candidates_path = candidates_path or Path("config/candidates.json")
        self.promotion_criteria = promotion_criteria or self.DEFAULT_PROMOTION_CRITERIA
        
        self._candidates: Dict[str, Dict] = {}
        self._load()
        
        logger.info(f"CandidateManager: {len(self._candidates)} candidatos carregados")
    
    def _load(self):
        """Carrega candidatos do arquivo."""
        if self.candidates_path.exists():
            try:
                with open(self.candidates_path, 'r', encoding='utf-8') as f:
                    data = json.load(f)
                
                for candidate in data.get("candidates", []):
                    cid = candidate.get("id")
                    if cid:
                        self._candidates[cid] = candidate
                
            except Exception as e:
                logger.error(f"Erro ao carregar candidates.json: {e}")
    
    def _save(self):
        """Salva candidatos para arquivo."""
        try:
            # Cria diretório se não existir
            self.candidates_path.parent.mkdir(parents=True, exist_ok=True)
            
            data = {
                "schema_version": "1.0",
                "last_updated": datetime.now().isoformat(),
                "candidates": list(self._candidates.values()),
                "promotion_queue": self.get_promotion_queue()
            }
            
            with open(self.candidates_path, 'w', encoding='utf-8') as f:
                json.dump(data, f, indent=2, ensure_ascii=False)
                
        except Exception as e:
            logger.error(f"Erro ao salvar candidates.json: {e}")
    
    def add_candidate(
        self,
        factory: str,
        code: str,
        context: Dict,
        source_workflow: str
    ) -> str:
        """
        Adiciona novo candidato.
        
        Args:
            factory: Classe factory do node
            code: Código Python gerado
            context: Contexto (settings, schema, etc)
            source_workflow: Nome do workflow de origem
            
        Returns:
            ID do candidato criado
        """
        # Gera ID único
        candidate_id = str(uuid.uuid4())[:8]
        
        # Gera hash do código para detectar duplicatas
        code_hash = hashlib.md5(code.encode()).hexdigest()[:12]
        
        # Verifica se já existe candidato similar
        for existing in self._candidates.values():
            if (existing.get("factory") == factory and 
                existing.get("code_hash") == code_hash):
                logger.info(f"Candidato similar já existe: {existing['id']}")
                return existing["id"]
        
        candidate = {
            "id": candidate_id,
            "factory": factory,
            "code_hash": code_hash,
            "status": "pending",
            "created_at": datetime.now().isoformat(),
            
            "generated_by": {
                "model": context.get("model", "unknown"),
                "prompt_version": "1.0"
            },
            
            "source_context": {
                "workflow": source_workflow,
                "node_id": context.get("node_id", ""),
                "node_name": context.get("node_name", "")
            },
            
            "settings_snapshot": context.get("settings", {}),
            "input_schema": context.get("input_schema", []),
            
            "generated_code": code,
            "imports_required": context.get("imports", []),
            
            "validation_results": [],
            
            "promotion_criteria": self.promotion_criteria
        }
        
        self._candidates[candidate_id] = candidate
        self._save()
        
        logger.info(f"Candidato criado: {candidate_id} para {factory}")
        
        return candidate_id
    
    def get_candidate(self, candidate_id: str) -> Optional[Dict]:
        """Obtém candidato por ID."""
        return self._candidates.get(candidate_id)
    
    def get_by_factory(self, factory: str) -> List[Dict]:
        """
        Obtém todos os candidatos para um factory.
        
        Args:
            factory: Factory class
            
        Returns:
            Lista de candidatos
        """
        return [
            c for c in self._candidates.values()
            if c.get("factory") == factory
        ]
    
    def record_validation(
        self,
        candidate_id: str,
        workflow: str,
        result: str,
        details: Optional[Dict] = None
    ):
        """
        Registra resultado de validação.
        
        Args:
            candidate_id: ID do candidato
            workflow: Nome do workflow usado para validar
            result: "PASS" ou "FAIL"
            details: Detalhes adicionais (divergências, etc)
        """
        if candidate_id not in self._candidates:
            logger.warning(f"Candidato não encontrado: {candidate_id}")
            return
        
        candidate = self._candidates[candidate_id]
        
        validation = {
            "workflow": workflow,
            "result": result,
            "validated_at": datetime.now().isoformat(),
            "details": details or {}
        }
        
        candidate["validation_results"].append(validation)
        
        # Atualiza status
        if result == "FAIL":
            candidate["status"] = "failing"
        elif self.check_promotion_ready(candidate_id):
            candidate["status"] = "ready_for_promotion"
        else:
            candidate["status"] = "validating"
        
        self._save()
        
        logger.info(
            f"Validação registrada: {candidate_id} = {result} "
            f"(total: {len(candidate['validation_results'])})"
        )
    
    def check_promotion_ready(self, candidate_id: str) -> bool:
        """
        Verifica se candidato está pronto para promoção.
        
        Critérios:
        - >= min_validations
        - pass_rate >= min_pass_rate
        - Validado em >= min_workflows diferentes
        
        Args:
            candidate_id: ID do candidato
            
        Returns:
            True se pronto para promoção
        """
        if candidate_id not in self._candidates:
            return False
        
        candidate = self._candidates[candidate_id]
        validations = candidate.get("validation_results", [])
        
        if not validations:
            return False
        
        # Conta validações
        total = len(validations)
        passed = sum(1 for v in validations if v["result"] == "PASS")
        workflows = len(set(v["workflow"] for v in validations))
        
        # Verifica critérios
        criteria = candidate.get("promotion_criteria", self.promotion_criteria)
        
        meets_min_validations = total >= criteria.get("min_validations", 3)
        meets_pass_rate = (passed / total) >= criteria.get("min_pass_rate", 1.0)
        meets_min_workflows = workflows >= criteria.get("min_workflows", 2)
        
        return meets_min_validations and meets_pass_rate and meets_min_workflows
    
    def get_promotion_queue(self) -> List[str]:
        """
        Retorna IDs de candidatos prontos para promoção.
        
        Returns:
            Lista de IDs
        """
        return [
            cid for cid in self._candidates
            if self.check_promotion_ready(cid)
        ]
    
    def remove_candidate(self, candidate_id: str):
        """
        Remove candidato (após promoção ou rejeição).
        
        Args:
            candidate_id: ID do candidato
        """
        if candidate_id in self._candidates:
            del self._candidates[candidate_id]
            self._save()
            logger.info(f"Candidato removido: {candidate_id}")
    
    def get_statistics(self) -> Dict[str, Any]:
        """
        Retorna estatísticas dos candidatos.
        
        Returns:
            Dict com estatísticas
        """
        total = len(self._candidates)
        
        by_status = {}
        by_factory = {}
        total_validations = 0
        total_passed = 0
        
        for candidate in self._candidates.values():
            # Por status
            status = candidate.get("status", "unknown")
            by_status[status] = by_status.get(status, 0) + 1
            
            # Por factory
            factory = candidate.get("factory", "unknown")
            by_factory[factory] = by_factory.get(factory, 0) + 1
            
            # Validações
            for v in candidate.get("validation_results", []):
                total_validations += 1
                if v["result"] == "PASS":
                    total_passed += 1
        
        return {
            "total_candidates": total,
            "by_status": by_status,
            "by_factory": by_factory,
            "total_validations": total_validations,
            "overall_pass_rate": (
                total_passed / total_validations 
                if total_validations > 0 else 0
            ),
            "ready_for_promotion": len(self.get_promotion_queue())
        }
```

## ARQUIVO: tests/test_response_validator.py e tests/test_candidate_manager.py

Crie testes completos para ambas as classes.

## VALIDAÇÃO

```bash
python -m py_compile src/ai/response_validator.py
python -m py_compile src/ai/candidate_manager.py
pytest tests/test_response_validator.py tests/test_candidate_manager.py -v
```
```

---

## SPRINT 5: GERAÇÃO DE CÓDIGO

<a name="prompt-51"></a>
### PROMPT 5.1 - Gerador Python Completo

```
Crie o gerador de código Python completo a partir do IR traduzido.

## ARQUIVO: src/codegen/__init__.py

```python
"""
Módulo CodeGen - Geração de Código Python
=========================================

Gera código Python executável a partir do WorkflowIR traduzido.
"""

from .python_generator import PythonGenerator
from .docstring_generator import DocstringGenerator

__all__ = ['PythonGenerator', 'DocstringGenerator']
```

## ARQUIVO: src/codegen/python_generator.py

```python
"""
Gerador Python - KNIME to Python Converter
==========================================

Gera arquivo Python completo e executável a partir do WorkflowIR.

O código gerado inclui:
- Header com metadados (origem, data, versão)
- Imports organizados (stdlib → third-party)
- Funções para nodes complexos
- main() com execução na ordem topológica
- Docstrings de rastreabilidade

Uso:
    generator = PythonGenerator(workflow_ir)
    code = generator.generate()
    generator.save(Path("output/workflow.py"))
"""

from typing import Dict, Any, List, Optional, Set
from pathlib import Path
from datetime import datetime
from collections import OrderedDict
import ast

from src.utils.logger import setup_logger

logger = setup_logger("PythonGenerator")


class PythonGenerator:
    """
    Gera arquivo Python completo a partir do WorkflowIR traduzido.
    """
    
    # =========================================================================
    # TEMPLATES
    # =========================================================================
    
    HEADER_TEMPLATE = '''"""
================================================================================
CÓDIGO GERADO AUTOMATICAMENTE - NÃO EDITAR MANUALMENTE
================================================================================
Workflow Original: {workflow_name}
Arquivo Fonte:     {source_file}
Gerado em:         {generated_at}
Gerador:           KNIME to Python Converter v2.0

Estatísticas:
- Nodes totais:    {total_nodes}
- Nodes mapeados:  {mapped_nodes}
- Nodes por IA:    {ai_nodes}
- Loops:           {total_loops}

ATENÇÃO: Este código foi gerado automaticamente a partir de um workflow KNIME.
Qualquer modificação manual pode ser sobrescrita em futuras conversões.
================================================================================
"""

'''

    IMPORTS_TEMPLATE = '''# =============================================================================
# IMPORTS
# =============================================================================
{imports}

'''

    FUNCTIONS_TEMPLATE = '''# =============================================================================
# FUNÇÕES DE TRANSFORMAÇÃO
# =============================================================================

{functions}
'''

    MAIN_TEMPLATE = '''# =============================================================================
# EXECUÇÃO PRINCIPAL
# =============================================================================

def main():
    """
    Executa o workflow completo.
    
    Ordem de execução baseada na análise topológica do grafo KNIME.
    Cada bloco corresponde a um node do workflow original.
    
    Returns:
        Dict com DataFrames de saída (nodes sem sucessores)
    """
{main_body}
    
    # Retorna outputs finais
    return {{
{return_dict}
    }}


if __name__ == "__main__":
    print("Iniciando execução do workflow...")
    print("-" * 60)
    
    results = main()
    
    print("-" * 60)
    print("Workflow executado com sucesso!")
    print(f"Outputs gerados: {{len(results)}}")
    
    for name, df in results.items():
        if hasattr(df, 'shape'):
            print(f"  - {{name}}: {{df.shape[0]}} linhas, {{df.shape[1]}} colunas")
        else:
            print(f"  - {{name}}: {{type(df).__name__}}")
'''

    def __init__(self, workflow_ir: Dict):
        """
        Inicializa o gerador.
        
        Args:
            workflow_ir: Representação intermediária do workflow
        """
        self.ir = workflow_ir
        
        # Coleções durante geração
        self._imports: Set[str] = set()
        self._functions: List[str] = []
        self._main_lines: List[str] = []
        self._output_vars: List[str] = []
        
        # Adiciona imports padrão
        self._imports.add("import pandas as pd")
        self._imports.add("import numpy as np")
    
    def generate(self) -> str:
        """
        Gera código Python completo.
        
        Returns:
            String com código Python executável
        """
        logger.info("Iniciando geração de código Python")
        
        # Limpa estado
        self._functions.clear()
        self._main_lines.clear()
        self._output_vars.clear()
        
        # 1. Coleta imports de todos os nodes
        self._collect_imports()
        
        # 2. Gera funções para nodes complexos
        self._generate_functions()
        
        # 3. Gera corpo do main()
        self._generate_main_body()
        
        # 4. Monta código final
        code = self._assemble_code()
        
        # 5. Formata código
        code = self._format_code(code)
        
        # 6. Valida sintaxe
        if not self._validate_syntax(code):
            logger.warning("Código gerado tem problemas de sintaxe")
        
        logger.info(
            f"Código gerado: {len(code)} caracteres, "
            f"{len(self._imports)} imports, {len(self._functions)} funções"
        )
        
        return code
    
    def _collect_imports(self):
        """Coleta imports de todos os nodes."""
        nodes = self.ir.get("graph", {}).get("nodes", {})
        
        for node_id, node in nodes.items():
            # Imports do mapeamento usado
            imports = node.get("imports", [])
            for imp in imports:
                if imp.startswith("import ") or imp.startswith("from "):
                    self._imports.add(imp)
                else:
                    self._imports.add(f"import {imp}")
    
    def _generate_functions(self):
        """Gera funções para nodes complexos (metanodes, loops)."""
        nodes = self.ir.get("graph", {}).get("nodes", {})
        loops = self.ir.get("graph", {}).get("loops", [])
        
        # Gera funções para metanodes
        for node_id, node in nodes.items():
            if node.get("is_metanode") and node.get("internal_workflow"):
                func = self._generate_metanode_function(node_id, node)
                if func:
                    self._functions.append(func)
        
        # Gera funções para loops (se houver lógica complexa)
        for loop in loops:
            func = self._generate_loop_function(loop)
            if func:
                self._functions.append(func)
    
    def _generate_metanode_function(
        self, 
        node_id: str, 
        node: Dict
    ) -> Optional[str]:
        """Gera função para um metanode."""
        name = node.get("name", f"metanode_{node_id}").replace(" ", "_").replace("#", "")
        internal = node.get("internal_workflow", {})
        
        # TODO: Implementar geração recursiva de metanodes
        return f'''
def process_{name}(df_input):
    """
    Metanode: {node.get("display_name", name)}
    ID: #{node_id}
    
    Nodes internos: {len(internal.get("nodes", {}))}
    """
    # TODO: Implementar lógica do metanode
    return df_input
'''
    
    def _generate_loop_function(self, loop: Dict) -> Optional[str]:
        """Gera função para um loop."""
        loop_id = loop.get("id", "unknown")
        loop_type = loop.get("loop_type", "GenericLoop")
        
        return f'''
def execute_loop_{loop_id}(df_input, group_columns=None):
    """
    Loop: {loop_type}
    Start Node: #{loop.get("start_node")}
    End Node: #{loop.get("end_node")}
    """
    results = []
    
    if group_columns:
        for group_key, group_df in df_input.groupby(group_columns):
            # Processa grupo
            result = group_df.copy()
            # TODO: Lógica interna do loop
            results.append(result)
    else:
        # Loop simples
        results.append(df_input.copy())
    
    return pd.concat(results, ignore_index=True)
'''
    
    def _generate_main_body(self):
        """Gera corpo do main() na ordem de execução."""
        execution_order = self.ir.get("graph", {}).get("execution_order", [])
        nodes = self.ir.get("graph", {}).get("nodes", {})
        
        if not execution_order:
            # Fallback: usa ordem das chaves
            execution_order = list(nodes.keys())
        
        for node_id in execution_order:
            node = nodes.get(node_id, {})
            
            if not node:
                continue
            
            # Gera código do node
            node_code = self._generate_node_code(node_id, node)
            
            if node_code:
                self._main_lines.append(node_code)
            
            # Identifica outputs finais (nodes sem sucessores)
            if not node.get("successors"):
                output_var = f"df_node_{node_id}"
                if output_var not in self._output_vars:
                    self._output_vars.append(output_var)
    
    def _generate_node_code(self, node_id: str, node: Dict) -> str:
        """
        Gera código para um node específico.
        
        Args:
            node_id: ID do node
            node: Dict com informações do node
            
        Returns:
            Código Python para o node
        """
        lines = []
        
        # Comentário com informações do node
        node_name = node.get("display_name", node.get("name", f"Node #{node_id}"))
        factory = node.get("factory", "Unknown")
        classification = node.get("classification", "UNKNOWN")
        annotation = node.get("annotation", "")
        
        lines.append(f"    # {'=' * 70}")
        lines.append(f"    # Node: {node_name}")
        lines.append(f"    # Factory: {factory}")
        lines.append(f"    # Classificação: {classification}")
        
        if annotation:
            lines.append(f"    # Descrição: {annotation}")
        
        lines.append(f"    # {'=' * 70}")
        
        # Código Python do node
        python_code = node.get("python_code", "")
        
        if python_code:
            # Indenta código
            for line in python_code.split('\n'):
                if line.strip():
                    # Garante indentação de 4 espaços
                    if not line.startswith("    "):
                        line = "    " + line
                    lines.append(line)
        else:
            # Placeholder para nodes sem código
            lines.append(f"    # TODO: Implementar {node_name}")
            lines.append(f"    df_node_{node_id} = df_node_{node.get('predecessors', ['input'])[0] if node.get('predecessors') else 'input'}.copy()")
        
        lines.append("")  # Linha em branco entre nodes
        
        return '\n'.join(lines)
    
    def _assemble_code(self) -> str:
        """Monta código final juntando todas as partes."""
        parts = []
        
        # 1. Header
        metadata = self.ir.get("metadata", {})
        stats = self.ir.get("statistics", {})
        
        header = self.HEADER_TEMPLATE.format(
            workflow_name=metadata.get("name", "Unknown"),
            source_file=metadata.get("source_file", "Unknown"),
            generated_at=datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
            total_nodes=stats.get("total_nodes", 0),
            mapped_nodes=stats.get("nodes_by_classification", {}).get("MAPPED", 0),
            ai_nodes=stats.get("nodes_by_classification", {}).get("UNKNOWN", 0),
            total_loops=stats.get("total_loops", 0)
        )
        parts.append(header)
        
        # 2. Imports
        sorted_imports = self._sort_imports()
        imports_section = self.IMPORTS_TEMPLATE.format(
            imports='\n'.join(sorted_imports)
        )
        parts.append(imports_section)
        
        # 3. Funções (se houver)
        if self._functions:
            functions_section = self.FUNCTIONS_TEMPLATE.format(
                functions='\n'.join(self._functions)
            )
            parts.append(functions_section)
        
        # 4. Main
        main_body = '\n'.join(self._main_lines) if self._main_lines else "    pass"
        
        return_dict_lines = []
        for var in self._output_vars[:10]:  # Limita a 10 outputs
            name = var.replace("df_node_", "output_")
            return_dict_lines.append(f'        "{name}": {var},')
        
        return_dict = '\n'.join(return_dict_lines) if return_dict_lines else '        "output": df_output,'
        
        main_section = self.MAIN_TEMPLATE.format(
            main_body=main_body,
            return_dict=return_dict
        )
        parts.append(main_section)
        
        return '\n'.join(parts)
    
    def _sort_imports(self) -> List[str]:
        """Ordena imports: stdlib → third-party → local."""
        stdlib = []
        third_party = []
        local = []
        
        stdlib_modules = {
            're', 'os', 'sys', 'datetime', 'collections', 'itertools',
            'functools', 'math', 'json', 'csv', 'pathlib', 'typing'
        }
        
        for imp in self._imports:
            # Extrai nome do módulo
            if imp.startswith("from "):
                module = imp.split()[1].split('.')[0]
            else:
                module = imp.split()[1].split('.')[0]
            
            if module in stdlib_modules:
                stdlib.append(imp)
            elif module in {'pandas', 'pd', 'numpy', 'np'}:
                third_party.insert(0, imp)  # Pandas/NumPy primeiro
            elif '.' in imp and 'src.' in imp:
                local.append(imp)
            else:
                third_party.append(imp)
        
        result = []
        
        if stdlib:
            result.extend(sorted(set(stdlib)))
        
        if third_party:
            if result:
                result.append("")  # Linha em branco
            result.extend(sorted(set(third_party)))
        
        if local:
            if result:
                result.append("")
            result.extend(sorted(set(local)))
        
        return result
    
    def _format_code(self, code: str) -> str:
        """Formata código seguindo PEP8 básico."""
        lines = code.split('\n')
        formatted = []
        
        prev_empty = False
        
        for line in lines:
            # Remove espaços em branco no final
            line = line.rstrip()
            
            # Evita mais de 2 linhas em branco consecutivas
            is_empty = len(line.strip()) == 0
            
            if is_empty:
                if prev_empty:
                    continue
                prev_empty = True
            else:
                prev_empty = False
            
            formatted.append(line)
        
        return '\n'.join(formatted)
    
    def _validate_syntax(self, code: str) -> bool:
        """Valida sintaxe do código gerado."""
        try:
            ast.parse(code)
            return True
        except SyntaxError as e:
            logger.error(f"Erro de sintaxe: linha {e.lineno}: {e.msg}")
            return False
    
    def save(self, output_path: Path) -> Path:
        """
        Salva código gerado em arquivo.
        
        Args:
            output_path: Caminho do arquivo de saída
            
        Returns:
            Path do arquivo criado
        """
        code = self.generate()
        
        # Cria diretório se necessário
        output_path = Path(output_path)
        output_path.parent.mkdir(parents=True, exist_ok=True)
        
        # Salva arquivo
        output_path.write_text(code, encoding='utf-8')
        
        logger.info(f"Código salvo em: {output_path}")
        
        return output_path
```

## ARQUIVO: src/codegen/docstring_generator.py

Crie gerador de docstrings de rastreabilidade.

## ARQUIVO: tests/test_python_generator.py

Crie testes completos.

## VALIDAÇÃO

```bash
python -m py_compile src/codegen/python_generator.py
pytest tests/test_python_generator.py -v
```
```

---

## SPRINTS 6 E 7

Os prompts para os Sprints 6 (Validação) e 7 (Orquestrador) seguem o mesmo padrão detalhado. Por limitação de espaço, incluo aqui a estrutura principal:

### SPRINT 6.1 - Validação Cruzada (src/validation/)
- SchemaComparator: Compara schemas gerado vs spec.xml
- WorkflowValidator: Valida workflow completo
- ValidationReportGenerator: Gera relatórios JSON/Markdown

### SPRINT 6.2 - Sistema de Aprendizado (src/learning/)
- MappingPromoter: Promove candidatos para oficiais
- RejectionAnalyzer: Analisa padrões de falhas
- MetricsTracker: Rastreia evolução do sistema

### SPRINT 7.1 - Orquestrador e CLI (main.py, cli.py)
- KnimeConverter: Classe principal que orquestra todas as fases
- CLI com argparse para uso em terminal

---

## 📋 CHECKLIST FINAL DE VALIDAÇÃO

Após implementar todos os prompts, execute:

```bash
# 1. Verifica sintaxe de todos os módulos
find src -name "*.py" -exec python -m py_compile {} \;

# 2. Executa todos os testes
pytest tests/ -v --cov=src

# 3. Verifica imports
python -c "
from src.extractors import KnimeZipExtractor, WorkflowParser, SettingsParser, SpecParser
from src.ir import GraphBuilder, TopologicalSorter, IRExporter
from src.mapping import NodeClassifier, DeterministicTranslator
from src.ai import PromptBuilder, VertexAIClient, ResponseValidator, CandidateManager
from src.codegen import PythonGenerator
from src.validation import SchemaComparator, WorkflowValidator
from src.learning import MappingPromoter, MetricsTracker
print('Todos os imports OK!')
"

# 4. Teste end-to-end (se tiver arquivo .knwf)
python cli.py exemplo.knwf --no-ai --verbose
```

---

**Documento completo gerado em:** 2025-12-15  
**Total de Prompts:** 16 (incluindo Prompt Zero)  
**Para uso com:** Windsurf + Claude Opus 4.5
