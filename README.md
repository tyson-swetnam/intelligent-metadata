# Intelligent Metadata Generator for PRIDE Archive

Automated retrieval and structuring of proteomics project metadata from the PRIDE (PRoteomics IDEntifications Database) Archive. This repository uses Claude Code with a specialized MCP (Model Context Protocol) server to search, retrieve, and generate standardized JSON metadata files for proteomics projects.

## Overview

This tool enables researchers to:
- Search the PRIDE Archive for proteomics projects by keyword (organism, technique, topic)
- Retrieve detailed project metadata including protocols, instruments, and annotations
- Generate structured JSON files following a standardized schema
- Ensure metadata compliance with proteomics community standards (PSI formats, ontologies)

## Features

- **PRIDE API Integration**: Direct access to PRIDE Archive through MCP server
- **Standardized Schema**: JSON Schema (draft-07) template for consistent metadata structure
- **Controlled Vocabularies**: Automatic formatting of ontology terms (MS, PSI-MOD, UniMod, NEWT taxonomy)
- **Batch Processing**: Generate metadata files for multiple projects in parallel
- **Quality Validation**: Built-in specialist agent for proteomics metadata validation

## Prerequisites

- [Claude Code](https://claude.ai/code) - AI-powered development environment
- [PRIDE MCP Server](https://github.com/tyson-swetnam/pride-mcp-server) - MCP server for PRIDE API access
- Node.js (required for MCP server)

## Installation

1. **Clone this repository**:
   ```bash
   git clone https://github.com/yourusername/intelligent-metadata.git
   cd intelligent-metadata
   ```

2. **Install and configure the PRIDE MCP Server**:
   ```bash
   # Clone and build the PRIDE MCP server
   git clone https://github.com/tyson-swetnam/pride-mcp-server.git
   cd pride-mcp-server
   npm install
   npm run build
   ```

3. **Configure MCP connection**:
   Copy the example configuration and update with your local path:
   ```bash
   cp .claude/mcp.json.example .claude/mcp.json
   ```

   Then edit `.claude/mcp.json` to point to your MCP server installation:
   ```json
   {
     "mcpServers": {
       "pride": {
         "command": "node",
         "args": [
           "/absolute/path/to/pride-mcp-server/dist/index.js"
         ],
         "env": {
           "PRIDE_API_TIMEOUT": "30000",
           "PRIDE_PAGE_SIZE": "10"
         }
       }
     }
   }
   ```

   **Note**: `.claude/mcp.json` is gitignored to prevent exposing local paths.

## Usage

### Using Claude Code

Open the repository in Claude Code and use natural language commands:

```
Search PRIDE for E. coli projects and generate metadata files
```

```
Find proteomics projects about phosphorylation and create JSON files
```

```
Get detailed information for project PXD012345 and save to outputs/
```

### Example Workflow

1. **Search for projects**:
   ```
   Search PRIDE for "breast cancer" proteomics projects
   ```

2. **Generate metadata files**:
   ```
   Generate metadata files for the first 10 results and save to outputs/
   ```

3. **Validate metadata** (using the specialized agent):
   ```
   Validate the generated metadata files for PSI compliance
   ```

## Repository Structure

```
.
├── .claude/
│   ├── agents/
│   │   └── pride-protein-metadata-specialist.md  # Specialized validation agent
│   ├── mcp.json.example                           # MCP server configuration template
│   ├── mcp.json                                   # MCP server configuration (local, gitignored)
│   └── settings.local.json                        # Claude Code settings (local, gitignored)
├── outputs/                                        # Generated metadata files
│   └── PXD*.json                                  # Project metadata (accession-named)
├── pride-metadata-template.json                   # JSON Schema template
├── CLAUDE.md                                      # Development guide for Claude Code
├── README.md                                      # This file
└── .gitignore                                     # Git ignore rules
```

## Metadata Schema

The `pride-metadata-template.json` defines a comprehensive schema including:

### Required Fields
- `accession`: PRIDE project accession (e.g., PXD000001)
- `title`: Project title
- `submissionType`: COMPLETE, PARTIAL, or PRIDE
- `submissionDate`: ISO 8601 datetime

### Key Metadata Sections
- **Project Information**: Description, keywords, tags, DOI
- **Protocols**: Sample processing and data processing workflows
- **People**: Submitters and lab PIs with ORCID support
- **Biological Context**: Organisms, tissues, diseases (with ontology terms)
- **Technical Details**: Instruments, software, quantification methods
- **Modifications**: Post-translational modifications (PTMs)
- **Publications**: References and citations

### Controlled Vocabulary Format

Ontology terms use the CvParam structure:
```json
{
  "@type": "CvParam",
  "cvLabel": "MS",
  "accession": "MS:1000449",
  "name": "LTQ Orbitrap"
}
```

**Supported Ontologies**:
- **MS**: Mass Spectrometry Ontology (instruments, methods)
- **NEWT**: NCBI Taxonomy (organisms)
- **PSI-MOD**: Protein Modifications
- **UniMod**: Unimod Modifications Database
- **BTO**: BRENDA Tissue Ontology
- **DOID**: Disease Ontology

## Output Files

Generated files are saved in `outputs/` with naming convention:
```
outputs/PXD{ACCESSION}.json
```

Example: `outputs/PXD069370.json`

Each file contains structured metadata validated against the schema template.

## PRIDE Archive

The [PRIDE Archive](https://www.ebi.ac.uk/pride/) is a centralized, standards-compliant repository for mass spectrometry-based proteomics data. It is a core member of the ProteomeXchange (PX) consortium.

**Key Resources**:
- PRIDE Archive: https://www.ebi.ac.uk/pride/
- PRIDE API Documentation: https://www.ebi.ac.uk/pride/ws/archive/v2/
- ProteomeXchange: http://www.proteomexchange.org/

## Available MCP Operations

The PRIDE MCP server provides these tools:

- **`pride_search_projects`**: Search projects by keyword with pagination
- **`pride_get_project`**: Get detailed metadata for a specific project accession
- **`pride_list_files`**: List all files associated with a project
- **`pride_search_proteins`**: Search for specific proteins within projects

## Contributing

Contributions are welcome! Areas for enhancement:
- Additional metadata validation rules
- Support for other proteomics repositories (MASSIVE, PeptideAtlas)
- Automated ontology term mapping improvements
- Bulk export capabilities
- Integration with other proteomics tools

## License

[Specify your license here]

## Citation

If you use this tool in your research, please cite:
- PRIDE Database: Perez-Riverol et al. (2022) Nucleic Acids Research
- ProteomeXchange: Deutsch et al. (2020) Nucleic Acids Research

## Contact

For questions or issues, please open an issue on GitHub or contact the maintainers.

## Acknowledgments

- PRIDE Team at EMBL-EBI for maintaining the PRIDE Archive and API
- ProteomeXchange Consortium for proteomics data standards
- Claude Code team for the MCP integration framework
