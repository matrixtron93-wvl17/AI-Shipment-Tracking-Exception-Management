# AI Shipment Tracking & Exception Management Automation

## Overview

An AI-powered logistics automation workflow built with n8n, Google Gemini, and Neon PostgreSQL.

The system is designed to receive shipment events, normalize shipment information, classify shipment status and potential exceptions, determine exception priority, store operational records, and support notification and human-review workflows.

## Project Status

🚧 In Development

The workflow is being built and tested incrementally.

## Core Technologies

- n8n
- Google Gemini
- Neon PostgreSQL
- REST APIs
- Webhooks
- JavaScript
- PowerShell
- Git / GitHub

## Core Workflow

```text
Shipment Event
      ↓
Data Normalization
      ↓
AI Shipment Classification
      ↓
Validation
      ↓
Exception Detection
      ↓
Priority / Escalation
      ↓
Neon PostgreSQL
      ↓
Notification / Response