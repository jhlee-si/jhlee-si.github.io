---
title: "Frontend/Backend Structure"
date: 2026-02-05 14:07:00 +0900
categories: [SI, Structure]
tags: [SI, structure]
mermaid: true
---
# 개발
MariaDB Query 작업 이력 정보를 관리

```mermaid
flowchart LR
  U[Browser] -->|HTTPS :443| N[Nginx]

  N -->|/ SPA, HMR| V["Vite Dev Server<br/>(React)"]
  N -->|/api/* REST/Auth| B["Spring Boot<br/>(Business + Auth)"]

  V -->|XHR/Fetch to /api/*| N
```

# 운영
```mermaid
flowchart LR
  U[Browser] -->|HTTPS :443| N[Nginx]

  N -->|"/ (Static SPA)"| S["React Build Artifacts<br/>(/dist served by Nginx)"]
  N -->|"/api/* (REST/Auth)"| B["Spring Boot<br/>(Business + Auth)"]
```
