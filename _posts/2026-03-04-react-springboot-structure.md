---
title: "Frontend/Backend Structure"
date: 2026-02-05 14:07:00 +0900
categories: [SI, Structure]
tags: [SI, structure]
mermaid: true
---
## 1) 환경 구성
### 1.1 OS      : Ubuntu (24.04)
### 1.2 Proxy   : Nginx (1.24)
### 1.3 Frontend: React (19.2.5)
### 1.4 Backend : Spring Boot (2.7.15), JDK (1.8)

## 2) 환경별 구조도
### 2.1 개발
```mermaid
flowchart LR
  U[Browser] -->|HTTPS :443| N[Nginx]

  N -->|/ SPA, HMR| V["Vite Dev Server<br/>(React)"]
  N -->|/api/* REST/Auth| B["Spring Boot<br/>(Business + Auth)"]

  V -->|XHR/Fetch to /api/*| N
```

### 2.2 운영
```mermaid
flowchart LR
  U[Browser] -->|HTTPS :443| N[Nginx]

  N -->|"/ (Static SPA)"| S["React Build Artifacts<br/>(/dist served by Nginx)"]
  N -->|"/api/* (REST/Auth)"| B["Spring Boot<br/>(Business + Auth)"]
```
