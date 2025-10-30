# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Core Commands
- `npm run dev` - Start development server with Mastra development mode
- `npm run build` - Build the project using Mastra build system
- `npm run start` - Start the production server

### Environment Setup
- Copy `.env.example` to `.env` and configure:
  - `LAUNCH_STUDIO_API_KEY` - Required for Launch Studio proxy service access
  - `LAUNCH_STUDIO_MODEL` - Model to use (launch-studio-boost or launch-studio-standard)
  - `E2B_API_KEY` - Required for sandbox creation and code execution

## Architecture Overview

This is a **Mastra-based AI coding agent** that provides secure code execution and file management through E2B sandboxes. The architecture follows a tool-based agent pattern with comprehensive development workflow capabilities.

### Core Components

#### **Mastra Framework Integration** (`src/mastra/index.ts`)
- Central Mastra instance with LibSQL storage backend
- Pino logger for development/production logging
- Observability enabled for agent monitoring
- Database stored at `mastra.db` (LibSQL)

#### **Coding Agent** (`src/mastra/agents/coding-agent.ts`)
- Main AI agent using Launch Studio proxy service (launch-studio-boost/launch-studio-standard models)
- Routes through Launch Studio API for customer isolation and rate limit management
- Configured with comprehensive tool access for development workflows
- **Memory System**: Semantic recall, working memory, and thread management with vector embeddings
- **Execution Limits**: 20 max steps per conversation to prevent infinite loops
- Designed for multi-language development (Python, JavaScript, TypeScript)

#### **E2B Tools** (`src/mastra/tools/e2b.ts`)
Complete E2B sandbox interaction toolkit with 14 distinct tools:

**Sandbox Management:**
- `createSandbox` - Initialize isolated environments with custom metadata/env vars

**Code Execution:**
- `runCode` - Execute Python/JS/TS with timeout and environment control

**File Operations:**
- `writeFile` / `writeFiles` - Single and batch file creation
- `readFile` - Content reading for validation/analysis
- `listFiles` - Directory exploration and structure validation
- `deleteFile` - Cleanup and file removal
- `createDirectory` - Project structure setup

**File Information:**
- `getFileInfo` - Detailed metadata (permissions, timestamps, size)
- `checkFileExists` - Conditional logic for file operations
- `getFileSize` - Monitor file sizes with human-readable formatting

**Development Workflow:**
- `watchDirectory` - Live file system monitoring with event capture
- `runCommand` - Shell command execution with working directory support

### Key Design Patterns

#### **Agent-Tool Architecture**
The agent operates through tools rather than direct API calls, enabling:
- Structured input/output validation with Zod schemas
- Error handling and recovery patterns
- Sandbox isolation and security

#### **Memory-Enabled Conversations**
- **Semantic Recall**: Vector-based search through conversation history
- **Working Memory**: Context maintenance across interactions
- **Thread Management**: Automatic conversation title generation
- **Vector Storage**: FastEmbed + LibSQL for semantic search

#### **Multi-Language Support**
The system is designed for polyglot development:
- Python environment with package management
- JavaScript/TypeScript with Node.js ecosystem
- Cross-language project coordination
- Build tool integration for each ecosystem

## Development Workflow Patterns

### **Project Initialization**
1. Create sandbox with appropriate environment variables
2. Set up project structure using `createDirectory` and `writeFiles`
3. Install dependencies via `runCommand`
4. Enable live monitoring with `watchDirectory`

### **Iterative Development**
1. Write code incrementally using `writeFile`
2. Execute and test with `runCode`
3. Monitor changes with file watching
4. Validate with `runCommand` for builds/tests

### **File Management Strategy**
- Use `writeFiles` for batch operations to reduce tool calls
- Validate with `checkFileExists` before operations
- Monitor file sizes to catch issues early
- Use proper directory structures for organization

## Technology Stack

- **Runtime**: Node.js 20+ (ES2022 modules)
- **AI Framework**: Mastra with Launch Studio proxy service
- **LLM Models**: launch-studio-boost (default) or launch-studio-standard
- **LLM Routing**: Launch Studio proxy (https://launch-studio-service-production.up.railway.app/v1)
- **Sandbox**: E2B Code Interpreter for secure execution
- **Storage**: LibSQL for agent memory and vector storage
- **Embedding**: FastEmbed for semantic search
- **Validation**: Zod for tool input/output schemas
- **Language**: TypeScript with strict configuration

## Security & Isolation

- All code execution happens in isolated E2B sandboxes
- LLM requests routed through Launch Studio proxy for customer account isolation
- Sandboxes have configurable timeouts (max 24h Pro, 1h Hobby)
- No direct file system access - all operations through E2B API
- Environment variables isolated per sandbox
- Resource limits enforced by E2B platform

## Launch Studio Integration

### Models Available
- **launch-studio-boost**: Higher performance model (default)
- **launch-studio-standard**: Standard performance model

### Custom Features
The Launch Studio proxy supports additional options via `launch_options`:
- **File Context**: Pass file contents for enhanced code understanding
- **Lazy Edits**: Enable smart edit suggestions
- **Smart Files Context**: Intelligent file context management

### Usage Monitoring
Check API usage with:
```bash
curl -X GET https://launch-studio-service-production.up.railway.app/v1/usage \
  -H "Authorization: Bearer $LAUNCH_STUDIO_API_KEY"
```