# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is an intelligent metadata generation repository focused on proteomics data from the PRIDE (PRoteomics IDEntifications Database) Archive. The repository enables automated retrieval of proteomics project metadata from PRIDE and generates structured JSON files following standardized schemas.

## Core Architecture

### PRIDE MCP Integration

The repository uses a Model Context Protocol (MCP) server for PRIDE API integration, configured in `.claude/mcp.json`:

- **PRIDE MCP Server**: Node-based MCP server providing direct access to PRIDE REST API
- **Available Operations**:
  - `mcp__pride__pride_search_projects`: Search for proteomics projects by keyword
  - `mcp__pride__pride_get_project`: Retrieve detailed project metadata by accession
  - `mcp__pride__pride_list_files`: List files associated with a project
  - `mcp__pride__pride_search_proteins`: Search for proteins within projects

### Specialized Agent

The repository includes a custom `pride-protein-metadata-specialist` agent (`.claude/agents/pride-protein-metadata-specialist.md`) designed for:
- Validating proteomics metadata against PSI (Proteomics Standards Initiative) specifications
- Structuring protein ontology annotations
- Mapping between protein databases (UniProt, RefSeq, Ensembl)
- Annotating post-translational modifications using PSI-MOD and UniMod ontologies
- Ensuring compliance with PRIDE submission standards

**When to use this agent**: Invoke when working with PRIDE submissions, validating proteomics data formats (mzIdentML, mzML, PRIDE XML), or annotating protein modifications.

## Metadata Schema

### Template Structure

The `pride-metadata-template.json` defines the JSON Schema (draft-07) for PRIDE project metadata with these key sections:

**Required Fields**:
- `accession`: PRIDE project accession (pattern: `^PXD\d+$`)
- `title`: Project title
- `submissionType`: One of "COMPLETE", "PARTIAL", "PRIDE"
- `submissionDate`: ISO 8601 datetime

**Core Metadata**:
- Project descriptions and protocols (sample processing, data processing)
- DOI and publication information
- Keywords and project classification tags
- Submission and publication dates

**Entities and Annotations**:
- `submitters` and `labPIs`: Contact information with ORCID support
- `organisms`: Species studied with NEWT taxonomy annotations
- `instruments`: Mass spectrometry instruments (CvParam format with MS ontology)
- `experimentTypes`: Experimental approaches (e.g., DDA, DIA)
- `identifiedPTMStrings`: Post-translational modifications with MOD ontology terms
- `softwares`: Analysis tools used
- `quantificationMethods`: Quantitative proteomics approaches

**Controlled Vocabulary Format**: Many fields use CvParam objects with:
```json
{
  "@type": "CvParam",
  "cvLabel": "MS",
  "accession": "MS:1000449",
  "name": "LTQ Orbitrap"
}
```

## Typical Workflows

### Searching and Retrieving PRIDE Projects

1. **Search by keyword**: Use `mcp__pride__pride_search_projects` with organism names, experimental approaches, or biological terms
2. **Retrieve full details**: Use `mcp__pride__pride_get_project` with project accession (e.g., "PXD012345")
3. **Generate metadata**: Map API response to template schema, extracting:
   - Basic project information
   - Sample and data processing protocols (often contain technical details)
   - Organism information (convert to CvParam format)
   - Submitter details
   - Keywords derived from project content

### Generating Metadata Files

**Output location**: Store generated metadata in `outputs/` directory with naming convention `{ACCESSION}.json` (e.g., `outputs/PXD069370.json`)

**Data transformation considerations**:
- API responses may return "N/A" for missing controlled vocabulary terms
- Protocol text often contains HTML formatting that should be preserved in JSON strings
- Dates should be converted to ISO 8601 format (`YYYY-MM-DDTHH:MM:SSZ`)
- Organism data needs conversion from API format to CvParam structure with taxonomy accessions

### Batch Processing

When processing multiple projects:
1. Fetch detailed metadata in parallel using multiple `mcp__pride__pride_get_project` calls
2. Create output directory if it doesn't exist: `mkdir -p outputs`
3. Generate individual JSON files for each project
4. Include relevant keywords in each file based on project description and organism

## Important Data Standards

### PRIDE Accession Format
- Pattern: `PXD` followed by 6+ digits (e.g., PXD000001, PXD069370)
- Used as primary identifier and filename basis

### Ontology Term Usage
- **MS Ontology**: Instruments, software, quantification methods (cvLabel: "MS")
- **NEWT/Taxonomy**: Organism classification (cvLabel: "NEWT")
- **PSI-MOD**: Post-translational modifications (cvLabel: "MOD")
- **BTO**: Organism parts/tissues (cvLabel: "BTO")
- **DOID**: Disease terms (cvLabel: "DOID")

### Protocol Documentation
Sample and data processing protocols often contain critical reproducibility information including:
- Lysis buffers and reagent concentrations
- Digestion enzymes and conditions
- Mass spectrometry parameters (resolution, scan ranges, collision energies)
- Database search parameters (mass tolerances, modifications, FDR thresholds)
- Quantification normalization strategies

## Working with the Repository

### File Organization
```
.
├── .claude/
│   ├── agents/
│   │   └── pride-protein-metadata-specialist.md
│   ├── mcp.json
│   └── settings.local.json
├── outputs/              # Generated metadata files
│   └── PXD*.json
└── pride-metadata-template.json
```

### Quality Assurance for Generated Metadata
- Verify required fields are populated
- Ensure accession numbers match PRIDE format
- Check that dates are valid ISO 8601 timestamps
- Validate organism information includes proper taxonomy terms
- Confirm CvParam structures have all required fields (@type, cvLabel, accession, name)

## MCP Server Configuration

The PRIDE MCP server environment variables (`.claude/mcp.json`):
- `PRIDE_API_TIMEOUT`: API request timeout (default: 30000ms)
- `PRIDE_PAGE_SIZE`: Results per page for search operations (default: 10)

**Note**: The MCP server path is absolute and may need adjustment for different development environments.
