---
title: "Frontend/Backend Structure"
date: 2026-02-05 14:07:00 +0900
categories: [SI, Structure]
tags: [SI, structure]
mermaid: true
---

flowchart LR
  U[Browser] -->|HTTPS :443| N[Nginx]
  N -->|/ (Static SPA)| S[React dist]
  N -->|/api/* (REST/Auth)| B[Spring Boot]
