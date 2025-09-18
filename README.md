# xOffense: An AI-driven autonomous penetration testing framework with offensive knowledge-enhanced LLMs and multi agent systems

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)

xOffense is an AI-powered penetration testing framework that automates cybersecurity assessments using Large Language Model (LLM) agents. The system combines automated vulnerability scanning, exploitation, and reporting through a sophisticated role-based architecture.

## Features

- **Multi-Agent Architecture**: Three specialized roles (Collector, Scanner, Exploiter) working together
- **LLM Integration**: Support for OpenAI, Ollama, and custom API endpoints
- **Knowledge Base**: RAG system using Milvus vector database for cybersecurity knowledge
- **Remote Execution**: SSH-based command execution on Kali Linux machines
- **Web Interface**: Streamlit-based UI for monitoring and interaction
- **Session Management**: Persistent sessions with database storage
- **Comprehensive Logging**: Detailed logging with configurable verbosity

## Table of Contents

- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [Project Structure](#project-structure)
- [Development](#development)
- [Testing Scenarios](#testing-scenarios)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)

## Architecture

### Multi-Agent System

xOffense implements a three-phase penetration testing approach:

1. **Collection / Reconnaissance Phase**: The Collector agent gathers initial reconnaissance data about targets
2. **Scanning Phase**: The Scanner agent identifies vulnerabilities and attack vectors
3. **Exploitation Phase**: The Exploiter agent executes exploitation techniques and privilege escalation

### Core Components

- **Planner System**: Manages task sequences and execution flow using LLM-based planning
- **Shell Manager**: Handles remote command execution via SSH to Kali Linux machines
- **Knowledge Base**: RAG system with Milvus vector database for cybersecurity documentation
- **Database Layer**: MySQL-backed persistence for sessions, plans, tasks, and conversations
- **Web Interface**: Streamlit-based UI for real-time monitoring and interaction

## Prerequisites

### Required Services

1. **Python 3.10+** (recommended: 3.11)
2. **MySQL Database** (for session and task persistence)
3. **Milvus Vector Database** (for RAG knowledge base)
4. **Kali Linux Machine** (for remote command execution)
5. **LLM Backend** (OpenAI API, Ollama, or compatible endpoint)

### System Requirements

- Linux/macOS (Windows with WSL)
- 8GB+ RAM (16GB recommended)
- Docker (for Milvus and victims from AutoPenBench)
- SSH client

## Installation

### 1. Clone the Repository

```bash
git clone <repository-url>
cd MyIntern
```

### 2. Install Python Dependencies

```bash
# Create virtual environment (recommended)
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### 3. Set Up Milvus Vector Database

```bash
# Start Milvus using the provided script
bash standalone_embed.sh start

# Verify Milvus is running
sudo docker ps | grep milvus-standalone
```

### 4. Set Up MySQL Database

Create a MySQL database and user for the application:

```sql
CREATE DATABASE mysql;
CREATE USER 'vuln'@'localhost' IDENTIFIED BY 'vulnbot';
GRANT ALL PRIVILEGES ON mysql.* TO 'vuln'@'localhost';
FLUSH PRIVILEGES;
```

### 5. Initialize the System

```bash
# Initialize database tables and directories
python cli.py init
```

## Configuration

xOffense uses four YAML configuration files that will be created automatically during initialization:

### `basic_config.yaml`
Core system settings including SSH connections and server ports:

```yaml
log_verbose: true
enable_rag: false
mode: auto
kali:
  hostname: "10.102.11.10"
  port: 22
  username: "root"
  password: "kali"
api_server:
  host: "0.0.0.0"
  port: 7861
webui_server:
  host: "0.0.0.0"
  port: 8501
```

### `model_config.yaml`
LLM configuration:

```yaml
api_key: "your-api-key"
llm_model: "openai"  # or "ollama"
base_url: "https://api.openai.com/v1"
llm_model_name: "gpt-4"
embedding_models: "maidalun1020/bce-embedding-base_v1"
temperature: 0.5
```

### `db_config.yaml`
Database settings:

```yaml
mysql:
  host: "localhost"
  port: 3306
  user: "vuln"
  password: "vulnbot"
  database: "mysql"
```

### `kb_config.yaml`
Knowledge base settings:

```yaml
default_vs_type: "milvus"
milvus:
  uri: "http://localhost:19530"
  user: "root"
  password: "Milvus"
```

## Usage

### Starting the Services

```bash
# Start both API server and Web UI
python cli.py start --all

# Start only API server
python cli.py start --api

# Start only Web UI
python cli.py start --webui
```

### Running Penetration Tests

#### Main VulnBot Interface (Recommended)

```bash
# Interactive role-based penetration testing
python cli.py vulnbot --max_interactions 10
```

This will:
1. Prompt you to describe the penetration testing task
2. Allow you to continue from previous sessions
3. Execute the three-phase testing approach
4. Save session data for future use

#### PentestGPT Mode (Experimental)

```bash
# Single-agent approach
python cli.py pentestgpt
```

#### Base Experiment Mode

```bash
# Basic testing mode
python cli.py base
```

### Web Interface

Access the web interface at `http://localhost:8501` after starting the services. The interface provides:
- Real-time monitoring of penetration testing sessions
- Task execution status
- Log viewing
- Configuration management

## Project Structure

```
xOffense/
├── actions/              # Core action modules
│   ├── planner.py        # Task planning logic
│   ├── shell_manager.py  # SSH command execution
│   └── write_code.py     # Code generation
├── config/               # Configuration management
├── db/                   # Database models and repositories
│   ├── models/           # SQLAlchemy models
│   └── repository/       # Data access layer
├── experiment/           # Experimental features
│   ├── pentestgpt.py     # Single-agent implementation
│   └── base.py           # Base experiment mode
├── prompts/              # LLM prompts
├── rag/                  # RAG system components
├── roles/                # Agent role implementations
│   ├── collector.py      # Collection phase agent
│   ├── scanner.py        # Scanning phase agent
│   └── exploiter.py      # Exploitation phase agent
├── server/               # API server
├── utils/                # Utility functions
├── web/                  # Web interface
├── cli.py                # Command-line interface
├── pentest.py            # Main penetration testing logic
└── startup.py            # Service startup
```

## Development

### Adding New Roles

1. Create a new role class inheriting from `roles/role.py`
2. Implement required methods: `run()`, `put_message()`
3. Define role-specific prompts in `prompts/` directory
4. Register in `pentest.py` roles dictionary

### Database Operations

The system automatically creates tables on initialization. Database models are in `db/models/` and repositories handle data access in `db/repository/`.

### LLM Integration

The framework supports multiple LLM backends:
- OpenAI API (configure `base_url` and `api_key`)
- Ollama local models
- Custom API endpoints via ngrok tunneling

### Knowledge Base

RAG functionality uses:
- Milvus vector database for embeddings
- BGE embedding models (local or remote)
- Document processing pipeline for cybersecurity knowledge

## Testing Scenarios

The system includes comprehensive penetration testing scenarios in `pentest_describe.txt` covering:

- **Network Security**: Port scanning, service enumeration
- **Web Application Security**: SQL injection, file upload vulnerabilities, RCE
- **Privilege Escalation**: SETUID exploitation, sudo vulnerabilities
- **Cryptographic Attacks**: Key recovery, cipher analysis
- **CVE Exploitation**: Metasploit integration for known vulnerabilities

Each scenario targets specific IP ranges (10.102.11.x) with detailed exploitation steps.

## Troubleshooting

### Common Issues

**Milvus Connection Error**
```bash
# Check Milvus status
sudo docker ps | grep milvus-standalone

# Restart Milvus
bash standalone_embed.sh restart
```

**MySQL Connection Error**
- Verify database credentials in `db_config.yaml`
- Ensure MySQL service is running
- Check user permissions

**SSH Connection Issues**
- Verify Kali machine IP and credentials in `basic_config.yaml`
- Test SSH connection manually
- Check network connectivity

**LLM API Errors**
- Verify API key and base URL in `model_config.yaml`
- Check API rate limits
- Test with different models

### Logging

Enable verbose logging in `basic_config.yaml`:
```yaml
log_verbose: true
```

Logs are stored in the `logs/` directory:
```bash
# View main log
tail -f logs/Auto-Pentest.log

# View service-specific logs
ls logs/
```

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable
5. Submit a pull request

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- Built using LangChain for LLM integration
- Milvus for vector database capabilities
- Streamlit for web interface
- FastAPI for API server

## Support

For issues and questions:
1. Check the [troubleshooting section](#troubleshooting)
2. Review the logs in the `logs/` directory
3. Open an issue in the repository

---

**Disclaimer**: This tool is for authorized penetration testing and educational purposes only. Users are responsible for ensuring they have proper authorization before testing any systems.
