# Architecture Diagram

## System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                          MASTER VM                              │
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐        │
│  │   Jenkins    │  │   Ansible    │  │     Git      │        │
│  │   Master     │  │              │  │              │        │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘        │
│         │                  │                  │                 │
└─────────┼──────────────────┼──────────────────┼─────────────────┘
          │                  │                  │
          │ SSH              │ SSH              │ Pull Code
          │                  │                  │
          ▼                  ▼                  ▼
┌─────────────────────────────────────────────────────────────────┐
│                      TEST SERVER (Slave)                        │
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐        │
│  │   Jenkins    │  │   Docker     │  │   Puppet     │        │
│  │   Slave      │  │   Engine     │  │   Agent      │        │
│  └──────────────┘  └──────┬───────┘  └──────────────┘        │
│                            │                                    │
│                            ▼                                    │
│         ┌────────────────────────────────────┐                │
│         │     Docker Container               │                │
│         │  ┌──────────────────────────────┐  │                │
│         │  │   Apache + PHP               │  │                │
│         │  │   AppleBite Application      │  │                │
│         │  │   Port: 80                   │  │                │
│         │  └──────────────────────────────┘  │                │
│         └────────────────────────────────────┘                │
│                            │                                    │
└────────────────────────────┼────────────────────────────────────┘
                             │
                             │ HTTP :80
                             │
                             ▼
                    ┌─────────────────┐
                    │   End Users     │
                    │   (Browser)     │
                    └─────────────────┘
```

## Deployment Flow

```
┌────────────┐
│ Developer  │
└─────┬──────┘
      │ git push
      ▼
┌────────────────┐
│ Git Repository │
└─────┬──────────┘
      │ webhook
      ▼
┌────────────────────────────────────────────────────────────┐
│                    Jenkins Pipeline                        │
│                                                            │
│  Stage 1: Setup Puppet Agent                               │
│  ┌────────────────────────────────────────────────────┐   │
│  │ • Download Puppet                                  │   │
│  │ • Install on Test Server                           │   │
│  │ • Configure agent                                  │   │
│  └────────────────────────────────────────────────────┘   │
│                          │                                 │
│                          ▼                                 │
│  Stage 2: Install Docker                                   │
│  ┌────────────────────────────────────────────────────┐   │
│  │ • Run Ansible playbook                             │   │
│  │ • Install Docker dependencies                      │   │
│  │ • Setup Docker engine                              │   │
│  │ • Configure permissions                            │   │
│  └────────────────────────────────────────────────────┘   │
│                          │                                 │
│                          ▼                                 │
│  Stage 3: Build & Deploy                                   │
│  ┌────────────────────────────────────────────────────┐   │
│  │ • Pull latest code                                 │   │
│  │ • Copy files to Test Server                        │   │
│  │ • Build Docker image                               │   │
│  │ • Stop old container                               │   │
│  │ • Start new container                              │   │
│  │ • Verify deployment                                │   │
│  └────────────────────────────────────────────────────┘   │
│                          │                                 │
│                   ┌──────┴──────┐                         │
│                   │             │                          │
│                Success      Failure                        │
│                   │             │                          │
│                   │             ▼                          │
│                   │   Stage 4: Cleanup                     │
│                   │   ┌──────────────────────┐            │
│                   │   │ • Stop container     │            │
│                   │   │ • Remove container   │            │
│                   │   │ • Clean resources    │            │
│                   │   └──────────────────────┘            │
│                   │                                        │
└───────────────────┼────────────────────────────────────────┘
                    │
                    ▼
         ┌──────────────────┐
         │ Application Live │
         │  on Port 80      │
         └──────────────────┘
```

## Network Diagram

```
Internet
    │
    │
    ▼
┌──────────────────────────────────────────────┐
│              Firewall/Security               │
└────────────┬─────────────────────┬───────────┘
             │                     │
             │                     │
     Port 8080 (Jenkins)      Port 80 (App)
             │                     │
             ▼                     ▼
    ┌────────────────┐    ┌────────────────┐
    │   Master VM    │    │  Test Server   │
    │                │───▶│                │
    │  Jenkins       │SSH │  Jenkins Slave │
    │  Ansible       │    │  Docker        │
    │  Git           │    │  Puppet Agent  │
    └────────────────┘    └────────────────┘
             │                     │
             └──────── LAN ────────┘
```

## Component Interaction

```
┌──────────────┐
│     Git      │◀─────── Developer pushes code
└──────┬───────┘
       │
       │ Webhook
       ▼
┌──────────────┐
│   Jenkins    │
│   Master     │
└──────┬───────┘
       │
       ├────────────────────┐
       │                    │
       ▼                    ▼
┌──────────────┐     ┌──────────────┐
│   Ansible    │     │    Puppet    │
│  (Job 2)     │     │   (Job 1)    │
└──────┬───────┘     └──────┬───────┘
       │                    │
       │     SSH            │ SSH
       │ ┌──────────────────┘
       │ │
       ▼ ▼
┌──────────────────┐
│  Test Server     │
│  ┌────────────┐  │
│  │   Docker   │  │
│  │  (Job 3)   │  │
│  │            │  │
│  │ ┌────────┐ │  │
│  │ │  App   │ │  │
│  │ └────────┘ │  │
│  └────────────┘  │
└──────────────────┘
```

## Data Flow

```
1. Code Change
   ↓
2. Git Push
   ↓
3. Webhook Trigger
   ↓
4. Jenkins Receives Event
   ↓
5. Start Pipeline
   ↓
6. Job 1: Puppet Setup ────────┐
   ↓                            │
7. Job 2: Ansible Docker ──────┼─→ Test Server
   ↓                            │
8. Job 3: Build & Deploy ──────┘
   │
   ├── Success: App Running
   │
   └── Failure: Job 4 Cleanup
```

## Technology Stack Layers

```
┌─────────────────────────────────────────┐
│          Application Layer              │
│         (PHP Web Application)           │
└─────────────────────────────────────────┘
                  │
┌─────────────────────────────────────────┐
│         Container Layer                 │
│         (Docker Container)              │
└─────────────────────────────────────────┘
                  │
┌─────────────────────────────────────────┐
│        Container Engine                 │
│         (Docker Engine)                 │
└─────────────────────────────────────────┘
                  │
┌─────────────────────────────────────────┐
│      Configuration Management           │
│      (Ansible + Puppet)                 │
└─────────────────────────────────────────┘
                  │
┌─────────────────────────────────────────┐
│         CI/CD Platform                  │
│          (Jenkins)                      │
└─────────────────────────────────────────┘
                  │
┌─────────────────────────────────────────┐
│       Version Control                   │
│            (Git)                        │
└─────────────────────────────────────────┘
                  │
┌─────────────────────────────────────────┐
│      Operating System                   │
│      (Ubuntu Linux)                     │
└─────────────────────────────────────────┘
```

## Security Architecture

```
┌────────────────────────────────────────────┐
│            External Access                 │
│          (HTTPS/SSL - Port 443)           │
└──────────────────┬─────────────────────────┘
                   │
                   ▼
         ┌─────────────────┐
         │   Firewall      │
         │   Rules         │
         └────────┬────────┘
                  │
         ┌────────┴────────┐
         │                 │
         ▼                 ▼
    ┌────────┐        ┌────────┐
    │ Master │        │ Slave  │
    │   VM   │        │ Node   │
    └────────┘        └────────┘
         │                 │
         │    SSH Keys     │
         │◀───────────────▶│
         │   (No Password) │
         │                 │
    ┌────────┐        ┌────────┐
    │Jenkins │        │ Docker │
    │Secured │        │Secured │
    └────────┘        └────────┘
```

## Monitoring Points

```
┌──────────────────────────────────────────────┐
│             Monitoring Layer                 │
│                                              │
│  ┌────────────┐  ┌────────────┐            │
│  │  Jenkins   │  │   Docker   │            │
│  │  Console   │  │    Logs    │            │
│  └────────────┘  └────────────┘            │
│                                              │
│  ┌────────────┐  ┌────────────┐            │
│  │  Ansible   │  │   System   │            │
│  │   Logs     │  │    Logs    │            │
│  └────────────┘  └────────────┘            │
└──────────────────────────────────────────────┘
```

## Backup Strategy

```
┌─────────────────────────────────────┐
│         What to Backup              │
│                                     │
│  ├─ Jenkins Configuration           │
│  ├─ SSH Keys                        │
│  ├─ Ansible Playbooks               │
│  ├─ Git Repository                  │
│  ├─ Docker Images (Tagged)          │
│  └─ Application Data                │
└─────────────────────────────────────┘
```



**Use these diagrams to understand the project architecture and workflow.**
