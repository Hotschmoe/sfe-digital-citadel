# Family Infrastructure Services

This document provides an overview of all services running in our infrastructure, categorized by their criticality and purpose.

## Service Categories

- **[CRITICAL]**: Essential services that require high uptime, regular backups, and immediate attention when issues arise
- **[ADDITIONAL]**: Quality-of-life services that enhance family digital life but are not mission-critical

## Infrastructure & Security Services

### Reverse Proxy & SSL
- **Traefik** [CRITICAL]
  - Handles all web traffic routing
  - Manages SSL certificates
  - Provides security layer for all web services

### Network Security
- **Pi-hole** [CRITICAL]
  - Network-wide ad blocking
  - DNS filtering and security
  - Network traffic monitoring
  - Primary DNS server for family network

### VPN Services
- **WireGuard** [CRITICAL]
  - Secure remote access to family network
  - Encrypted connection for sensitive services
  - Allows secure access from anywhere

### Monitoring & Alerts
- **Grafana** [CRITICAL]
  - System and service monitoring dashboards
  - Performance metrics visualization
  - Alert management

- **Prometheus** [CRITICAL]
  - Metrics collection and storage
  - System performance monitoring
  - Alert trigger system

- **Uptime Kuma** [CRITICAL]
  - Service availability monitoring
  - Uptime tracking
  - Status page for family services

## Data Management & Backup

### File Storage
- **Nextcloud** [CRITICAL]
  - Primary file storage and sharing
  - Calendar and contacts sync
  - Collaborative documents
  - Mobile backup solution

### Backup Systems
- **Duplicati** [CRITICAL]
  - Automated backup system
  - Encrypts sensitive data
  - Handles incremental backups
  - Disaster recovery solution

- **GitHub Backup** [CRITICAL]
  - Automated backup of all GitHub repositories
  - Includes private repos and organizations
  - Local backup of code assets

## Communication & Documentation

### Chat Platform
- **Matrix-Synapse** [CRITICAL]
  - Secure family chat system
  - End-to-end encrypted communications
  - File sharing and collaboration

### Documentation
- **WikiJS** [CRITICAL]
  - Family knowledge base
  - Technical documentation
  - Procedures and guides
  - Important information storage

### Calendar & Contacts
- **Radicale** [ADDITIONAL]
  - CalDAV/CardDAV server
  - Shared family calendars
  - Contact management

## Security Management

### Password Management
- **Vaultwarden** [CRITICAL]
  - Family password manager
  - Secure credential sharing
  - Emergency access
  - Two-factor authentication

## Media Services

### Media Server
- **Jellyfin** [ADDITIONAL]
  - Family media streaming
  - Movie and TV show organization
  - Music streaming
  - Photo management

### Media Management
- **Transmission-VPN** [ADDITIONAL]
  - Protected download client
  - Bandwidth management
  - VPN-secured transfers

- **Sonarr** [ADDITIONAL]
  - TV show management
  - Episode tracking
  - Automated downloads

- **Radarr** [ADDITIONAL]
  - Movie management
  - Film tracking
  - Automated downloads

- **Lidarr** [ADDITIONAL]
  - Music collection management
  - Artist tracking
  - Album organization

- **Jackett/Prowlarr** [ADDITIONAL]
  - Search indexer management
  - Content source aggregation

- **Bazarr** [ADDITIONAL]
  - Subtitle management
  - Automatic subtitle downloads

- **Overseerr** [ADDITIONAL]
  - Media request system
  - User management
  - Request tracking

## Education & Learning

### Learning Management
- **Moodle** [ADDITIONAL]
  - Family learning platform
  - Course management
  - Educational resource organization

### Development Environment
- **JupyterHub** [ADDITIONAL]
  - Interactive coding environment
  - Data science platform
  - Educational notebooks

### Version Control
- **Gitea** [ADDITIONAL]
  - Family code repository
  - Project management
  - Code collaboration

## Gaming Services

### Game Servers
- **Minecraft** [ADDITIONAL]
  - Family Minecraft server
  - Creative building space
  - Collaborative gaming

### Gaming Infrastructure
- **LanCache** [ADDITIONAL]
  - Game download caching
  - Bandwidth optimization
  - Steam and other gaming platforms

## Financial Management

### Budget Tracking
- **Firefly III** [CRITICAL]
  - Family budget management
  - Expense tracking
  - Financial planning
  - Asset management

## Digital Library

### E-Book Management
- **Calibre-Web** [ADDITIONAL]
  - Family e-book library
  - Reading progress tracking
  - Book metadata management

## Home Automation

### Smart Home
- **Home Assistant** [ADDITIONAL]
  - Smart device management
  - Automation rules
  - Energy monitoring
  - Scene control

## Service Dependencies

### Databases
All database services (PostgreSQL instances) are marked as [CRITICAL] as they store essential data:
- `nextcloud-db`
- `wiki-db`
- `moodle-db`
- `gitea-db`
- `firefly-db`

## Future Services

### CI/CD Pipeline [PLANNED]
- Drone CI or Jenkins
- Automated testing
- Deployment automation
- Integration with Gitea

### LLM Assistant Stack [PLANNED]
- **Core Components**
  - Local LLM Server (e.g., llama.cpp, text-generation-webui)
  - Vector database for embeddings (e.g., Qdrant, Milvus)
  - API Gateway for multiple LLM providers

- **Integration Points**
  - Matrix chatbot for family assistance
  - Knowledge base integration with WikiJS
  - Document analysis with Nextcloud files

- **Planned Features**
  - Unified chat interface for multiple LLMs
  - Local model for private queries
  - Cloud model fallback for complex tasks
  - Family context awareness
  - Document summarization
  - Code assistance
  - Learning material generation for Moodle

- **Privacy & Security**
  - Local processing for sensitive information
  - Configurable routing between local/cloud models
  - Data retention policies
  - Access control per family member

- **Infrastructure Requirements**
  - GPU support for local models
  - High-speed storage for embeddings
  - Memory optimization for model serving 