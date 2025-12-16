# Knowledge Graph Project with Ontology Generation Demo

This guide demonstrates how to create a knowledge graph project using Weaver and leverage its LLM-powered ontology generation capabilities to design a knowledge graph from an existing database schema.

## Overview

Weaver's knowledge graph template includes:
- Neo4j graph database integration
- Web scraping and API data extraction
- Prefect orchestration for data pipelines
- FastAPI for serving the knowledge graph
- LLM-powered ontology design from database schemas
- Schema inspection and analysis tools

The ontology generator can analyze a PostgreSQL database schema and use Claude (Anthropic) or GPT (OpenAI) to automatically design entity classes, properties, and relationships for your knowledge graph.

## Prerequisites

Before starting, ensure you have:

1. **Python 3.8+** installed
2. **Weaver CLI** installed:
   ```bash
   pip install weaver
   # OR if developing locally
   cd /path/to/weaver
   uv sync
   ```

3. **PostgreSQL database** with existing schema (for ontology generation)
   - Can be a sample database or your own data
   - Must be accessible via connection string

4. **LLM API Key** - Choose one:
   - **Anthropic API Key** (recommended): https://console.anthropic.com/
   - **OpenAI API Key**: https://platform.openai.com/

## Step 1: Create Knowledge Graph Project

Create a new knowledge graph project with ontology generation enabled:

```bash
# Using weaver CLI (installed package)
weaver create \
  --template knowledge-graph \
  --name my-knowledge-graph \
  --ontology \
  --llm anthropic

# OR if developing locally with uv
uv run weaver create \
  --template knowledge-graph \
  --name my-knowledge-graph \
  --ontology \
  --llm anthropic
```

### Command Options

- `--template knowledge-graph`: Use the knowledge graph template
- `--name my-knowledge-graph`: Name your project
- `--ontology` / `--no-ontology`: Enable/disable ontology generation (default: enabled)
- `--llm anthropic|openai`: Choose LLM provider (default: anthropic)
- `--db-host localhost`: Database host for schema inspection (default: localhost)
- `--db-port 5432`: Database port (default: 5432)

The CLI will display a summary of what will be created:

```
Project Type     Knowledge Graph
Data Sources     web_scraping, api
Storage          neo4j
Pipeline         prefect
API Framework    fastapi
Vector Search    ❌
NLP Processing   ✅
Docker Support   ✅
Ontology Generation  ✅
LLM Provider     ANTHROPIC
```

## Step 2: Navigate to Project

```bash
cd my-knowledge-graph
```

Your project structure will look like:

```
my-knowledge-graph/
├── src/
│   ├── extractors/         # Data extraction (web scraping, APIs)
│   ├── storage/            # Neo4j graph database integration
│   ├── pipeline/           # Prefect workflows
│   ├── api/                # FastAPI endpoints
│   ├── ontology/           # Ontology generation tools ⭐
│   │   ├── schema_inspector.py   # Database schema introspection
│   │   └── llm_designer.py       # LLM-powered ontology design
│   └── config/
│       └── settings.py     # Configuration
├── tests/
├── pyproject.toml
├── README.md
└── .env.example
```

## Step 3: Install Dependencies

```bash
# Install the project with dependencies
pip install -e .

# Or install with development dependencies
pip install -e ".[dev]"
```

The ontology generation module requires:
- `psycopg2-binary` - PostgreSQL database adapter
- `anthropic` - Anthropic API client (if using Claude)
- `openai` - OpenAI API client (if using GPT)

## Step 4: Configure Environment

Set up your environment variables:

```bash
# Copy the example environment file
cp .env.example .env

# Edit .env and add your API key
nano .env
```

Add your LLM API key to `.env`:

```bash
# For Anthropic (Claude)
ANTHROPIC_API_KEY=sk-ant-your-key-here

# OR for OpenAI (GPT)
OPENAI_API_KEY=sk-your-key-here

# Optional: Neo4j configuration for your knowledge graph
NEO4J_URI=bolt://localhost:7687
NEO4J_USER=neo4j
NEO4J_PASSWORD=your-password
```

## Step 5: Prepare a Sample Database (Optional)

If you don't have an existing database, you can create a sample one for testing:

```bash
# Create a test PostgreSQL database
createdb demo_company_db

# Connect and create sample schema
psql demo_company_db
```

Sample schema for a company database:

```sql
-- Companies table
CREATE TABLE companies (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL UNIQUE,
    website VARCHAR(255),
    industry VARCHAR(100),
    founded_year INTEGER,
    employee_count INTEGER,
    description TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- People table
CREATE TABLE people (
    id SERIAL PRIMARY KEY,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE,
    linkedin_url VARCHAR(255),
    title VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Employment relationships
CREATE TABLE employments (
    id SERIAL PRIMARY KEY,
    person_id INTEGER REFERENCES people(id),
    company_id INTEGER REFERENCES companies(id),
    title VARCHAR(100),
    start_date DATE,
    end_date DATE,
    is_current BOOLEAN DEFAULT true,
    UNIQUE(person_id, company_id, start_date)
);

-- Funding rounds
CREATE TABLE funding_rounds (
    id SERIAL PRIMARY KEY,
    company_id INTEGER REFERENCES companies(id),
    round_type VARCHAR(50), -- seed, series_a, series_b, etc.
    amount_usd BIGINT,
    announced_date DATE,
    lead_investor VARCHAR(255)
);

-- Products
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    company_id INTEGER REFERENCES companies(id),
    name VARCHAR(255) NOT NULL,
    description TEXT,
    launch_date DATE,
    category VARCHAR(100)
);

-- Add some sample data
INSERT INTO companies (name, website, industry, founded_year, employee_count, description) VALUES
    ('Anthropic', 'https://anthropic.com', 'AI Safety', 2021, 150, 'AI safety and research company'),
    ('TechCorp', 'https://techcorp.example', 'Software', 2015, 500, 'Enterprise software solutions');

INSERT INTO people (first_name, last_name, email, title) VALUES
    ('Jane', 'Smith', 'jane@example.com', 'CEO'),
    ('John', 'Doe', 'john@example.com', 'CTO');

INSERT INTO employments (person_id, company_id, title, start_date, is_current) VALUES
    (1, 1, 'CEO', '2021-01-01', true),
    (2, 1, 'CTO', '2021-06-01', true);
```

## Step 6: Inspect Database Schema

Use the schema inspector to analyze your database structure:

```bash
# Inspect the database and view summary
python -m src.ontology.schema_inspector 'postgresql://username:password@localhost/demo_company_db'
```

Replace `username` and `password` with your PostgreSQL credentials.

The inspector will output:

1. **Schema Summary** - Tables, columns, row counts
2. **Entity Candidates** - Tables likely to be entities (scored by heuristics)
3. **Relationships** - Foreign key relationships with cardinality
4. **JSON Export** - Full schema saved to `schema_output.json`

Example output:

```
===============================================================================
SCHEMA SUMMARY
===============================================================================
Database: demo_company_db
Schemas: public
Total Tables: 5

Table: public.companies
  Rows: 2
  Columns: 8
  Primary Keys: id
  Foreign Keys: 0
  Indexes: 2

===============================================================================
ENTITY CANDIDATES
===============================================================================

Table: companies
  Entity Score: 5
  Likely Entity: True
  Reasons: Has multiple attributes, Contains significant data, Not a junction table

Table: people
  Entity Score: 5
  Likely Entity: True
  Reasons: Has multiple attributes, Contains significant data, Not a junction table

===============================================================================
RELATIONSHIPS
===============================================================================

employments.person_id -> people.id
  Cardinality: many-to-one

employments.company_id -> companies.id
  Cardinality: many-to-one
```

## Step 7: Generate Ontology with LLM

Now use the LLM-powered designer to automatically create a knowledge graph ontology:

```bash
# Generate ontology using Claude (Anthropic)
python -m src.ontology.llm_designer \
  'postgresql://username:password@localhost/demo_company_db' \
  CompanyKnowledgeGraph
```

The tool will:

1. **Inspect the schema** - Extract all tables, columns, and relationships
2. **Call the LLM** - Send schema to Claude/GPT with ontology design instructions
3. **Generate ontology** - Create entity classes, properties, and relationships
4. **Explain decisions** - Provide rationale for design choices
5. **Suggest improvements** - Offer recommendations to enhance the ontology

### Output

```
Step 1: Inspecting database schema...
Found 5 tables in database 'demo_company_db'

Step 2: Designing ontology with LLM...

Ontology designed: CompanyKnowledgeGraph
  Classes: 4
  Properties: 15
  Relationships: 5

Ontology saved to: companyknowledgegraph_ontology.json

Step 3: Getting design explanation...
===============================================================================
DESIGN EXPLANATION
===============================================================================
[LLM provides detailed explanation of design decisions]

1. Entity Class Choices: I identified Company, Person, Employment, and
   FundingRound as the core entity classes. I excluded junction tables...

2. Property Selection: I selected properties that represent meaningful
   business attributes...

3. Relationship Modeling: The key relationships include WORKS_AT
   (Person to Company via Employment)...

Step 4: Getting improvement suggestions...
===============================================================================
IMPROVEMENT SUGGESTIONS
===============================================================================
1. Add temporal properties to track entity changes over time
2. Include provenance metadata to track data sources
3. Consider adding similarity properties for entity resolution
4. Model investor relationships more explicitly
5. Add confidence scores for fuzzy matching scenarios
```

## Step 8: Review Generated Ontology

The ontology is saved as JSON. Let's examine it:

```bash
cat companyknowledgegraph_ontology.json
```

Example structure:

```json
{
  "name": "CompanyKnowledgeGraph",
  "description": "Ontology for company, people, and funding relationships",
  "metadata": {
    "source_database": "demo_company_db",
    "source_schemas": ["public"],
    "total_source_tables": 5
  },
  "classes": [
    {
      "name": "Company",
      "description": "Business organization entity",
      "properties": ["name", "website", "industry", "foundedYear", "employeeCount"],
      "parent_class": null,
      "source_tables": ["companies"]
    },
    {
      "name": "Person",
      "description": "Individual person entity",
      "properties": ["firstName", "lastName", "email", "linkedinUrl", "currentTitle"],
      "parent_class": null,
      "source_tables": ["people"]
    }
  ],
  "properties": [
    {
      "name": "name",
      "description": "Company name - natural key for entity resolution",
      "data_type": "string",
      "domain": ["Company"],
      "range": null,
      "is_required": true,
      "is_unique": true
    },
    {
      "name": "email",
      "description": "Email address - natural key for person matching",
      "data_type": "string",
      "domain": ["Person"],
      "range": null,
      "is_required": false,
      "is_unique": true
    }
  ],
  "relationships": [
    {
      "name": "worksAt",
      "description": "Person's current employment at a company",
      "source_class": "Person",
      "target_class": "Company",
      "cardinality": "many-to-many",
      "inverse_name": "employs",
      "source_foreign_key": "employments.person_id"
    },
    {
      "name": "receivedFunding",
      "description": "Company received funding round",
      "source_class": "Company",
      "target_class": "FundingRound",
      "cardinality": "one-to-many",
      "inverse_name": "fundedCompany",
      "source_foreign_key": "funding_rounds.company_id"
    }
  ]
}
```

## Step 9: Implement in Neo4j (Optional)

Use the generated ontology to create your Neo4j graph schema:

```cypher
// Create constraints based on ontology
CREATE CONSTRAINT company_name IF NOT EXISTS FOR (c:Company) REQUIRE c.name IS UNIQUE;
CREATE CONSTRAINT person_email IF NOT EXISTS FOR (p:Person) REQUIRE p.email IS UNIQUE;

// Create nodes from your data
CREATE (c:Company {
  name: 'Anthropic',
  website: 'https://anthropic.com',
  industry: 'AI Safety',
  foundedYear: 2021,
  employeeCount: 150
});

CREATE (p:Person {
  firstName: 'Jane',
  lastName: 'Smith',
  email: 'jane@example.com',
  currentTitle: 'CEO'
});

// Create relationships
MATCH (p:Person {email: 'jane@example.com'})
MATCH (c:Company {name: 'Anthropic'})
CREATE (p)-[:WORKS_AT {title: 'CEO', startDate: date('2021-01-01'), isCurrent: true}]->(c);
```

## Step 10: Refine and Iterate (Optional)

The ontology designer supports iterative refinement:

### Refine Based on Feedback

```python
from src.ontology.llm_designer import OntologyDesigner, LLMProvider
from src.ontology.schema_inspector import SchemaInspector
import json

# Load existing ontology
with open('companyknowledgegraph_ontology.json', 'r') as f:
    ontology_dict = json.load(f)

# Get schema
with SchemaInspector('postgresql://user:pass@localhost/demo_company_db') as inspector:
    schema = inspector.inspect()
    schema_dict = schema.to_dict()

# Refine with feedback
designer = OntologyDesigner(provider=LLMProvider.ANTHROPIC)

from src.ontology.llm_designer import Ontology, OntologyClass, OntologyProperty, OntologyRelationship

# Reconstruct ontology object
ontology = Ontology(
    name=ontology_dict['name'],
    description=ontology_dict['description'],
    classes=[OntologyClass(**c) for c in ontology_dict['classes']],
    properties=[OntologyProperty(**p) for p in ontology_dict['properties']],
    relationships=[OntologyRelationship(**r) for r in ontology_dict['relationships']],
    metadata=ontology_dict.get('metadata', {})
)

refined_ontology = designer.refine_ontology(
    ontology=ontology,
    schema_dict=schema_dict,
    feedback="Add more properties for entity resolution, including phone numbers and addresses"
)

# Save refined version
with open('refined_ontology.json', 'w') as f:
    f.write(refined_ontology.to_json())
```

### Get Improvement Suggestions

```python
suggestions = designer.suggest_improvements(ontology, schema_dict)

for i, suggestion in enumerate(suggestions, 1):
    print(f"{i}. {suggestion}")
```

## Use Cases

The ontology generator is particularly useful for:

1. **Entity Resolution Projects**
   - Identifies natural keys (email, name) for matching
   - Suggests properties useful for deduplication
   - Models canonical entity patterns

2. **Data Migration**
   - Maps relational schemas to graph models
   - Identifies entity vs. relationship tables
   - Preserves cardinality constraints

3. **Knowledge Graph Design**
   - Automates initial ontology creation
   - Provides design rationale
   - Suggests improvements based on best practices

4. **Schema Documentation**
   - Generates human-readable entity descriptions
   - Explains relationships and their semantics
   - Identifies junction tables and lookup tables

## Advanced Configuration

### Using OpenAI Instead of Anthropic

```bash
# Set OpenAI API key in .env
export OPENAI_API_KEY=sk-your-key-here

# Create project with OpenAI
weaver create \
  --template knowledge-graph \
  --name my-kg \
  --llm openai

# Generate ontology with GPT
python -m src.ontology.llm_designer \
  'postgresql://user:pass@localhost/db' \
  MyOntology
```

The designer will automatically detect which API key is available.

### Custom LLM Models

Edit `src/ontology/llm_designer.py` to use specific models:

```python
designer = OntologyDesigner(
    provider=LLMProvider.ANTHROPIC,
    model="claude-opus-4-20250514"  # Or claude-sonnet-4-20250514
)
```

### Excluding System Schemas

```python
with SchemaInspector(connection_string) as inspector:
    schema = inspector.inspect(
        exclude_schemas=['audit', 'logs', 'temp'],
        include_row_counts=True
    )
```

## Troubleshooting

### API Key Not Found

```
Error: ANTHROPIC_API_KEY environment variable not set
```

**Solution**: Ensure your `.env` file contains the API key and is loaded:

```bash
export ANTHROPIC_API_KEY=sk-ant-your-key-here
# OR add to .env and source it
source .env
```

### Database Connection Failed

```
psycopg2.OperationalError: could not connect to server
```

**Solution**: Check your connection string format:

```
postgresql://username:password@hostname:port/database_name
```

### JSON Parsing Error

```
ValueError: Failed to parse LLM response as JSON
```

**Solution**: The LLM occasionally returns malformed JSON. Try:
1. Run again (LLMs can have variability)
2. Use a different model
3. Simplify your schema (fewer tables)

## Next Steps

After generating your ontology:

1. **Implement in Neo4j**: Create nodes, relationships, and constraints
2. **Build Data Pipeline**: Use the extractors to populate your graph
3. **Set Up API**: Expose graph queries via FastAPI endpoints
4. **Run Entity Resolution**: Use the ontology's natural keys for matching
5. **Iterate**: Refine the ontology based on real-world usage

## Resources

- **Neo4j Documentation**: https://neo4j.com/docs/
- **Anthropic API**: https://docs.anthropic.com/
- **Prefect Docs**: https://docs.prefect.io/
- **FastAPI**: https://fastapi.tiangolo.com/

## Summary

You've learned how to:
- Create a knowledge graph project with Weaver
- Inspect PostgreSQL database schemas
- Generate ontologies using LLMs (Claude/GPT)
- Review and refine ontology designs
- Implement knowledge graphs in Neo4j

The ontology generator accelerates knowledge graph development by automatically analyzing your data and proposing entity models, saving hours of manual design work.
