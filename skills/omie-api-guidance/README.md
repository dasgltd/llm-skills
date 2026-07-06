# Omie API Guidance

This skill acts as the definitive architectural guide for integrating any AI Agent or custom script with the Omie ERP API.

## What it does
It enforces strict development patterns for Omie, including:
1. **Rate Limiting & Blocking Prevention**: Teaches the AI about Omie's specific 4-concurrent request limit and 30-minute HTTP 425 blocks.
2. **Pagination Strategy**: Enforces caching of auxiliary tables (N+1 query prevention).
3. **Token Optimization**: Teaches the AI to translate numerical IDs into human-readable strings before LLM processing.
4. **Context-Specific Architectures**: Rules for Node.js backends, queues, and MCP Server instantiation (SSE limits).

## Module References
It includes detailed `.md` reference files for specific Omie modules (Finanças, Serviços, Produtos, CRM, Geral) so the AI reads the specific quirk of a module before generating code.

## License & Copyright
Copyright (c) 2026 Daniel A. Silva de la Garza / DASG Consulting Ltda. (CNPJ: 61.628.969/0001-04). All rights reserved.

Licensed for personal and educational (non-commercial) use only. See [LICENSE](./LICENSE) for the full terms.
