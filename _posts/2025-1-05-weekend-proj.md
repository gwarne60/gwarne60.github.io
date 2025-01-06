---
layout: post
title: "Building a Dockerized OT System with LLM Integration"
date: 2025-01-05
categories: [OT, Automation, Docker, OpenAI, OPC UA]
tags: [docker, opcua, python, nodejs, openai, llm, industrial-automation]
description: "Exploring how to integrate LLMs into industrial automation workflows using Docker, OPC UA, and Python."
---

# Building a Dockerized OT System with LLM Integration

Industrial automation is evolving rapidly, and the intersection of operational technology (OT) and machine learning offers exciting new possibilities. This project explores the integration of a **Dockerized OT system** with **Large Language Model (LLM) capabilities** for natural language-driven automation commands. Here’s how I designed and implemented a **custom tank simulation engine** integrated with OPC UA, Python, Node.js, and OpenAI.

---

## Inspiration for the Project

Modern OT systems often rely on a mix of legacy technologies and cutting-edge platforms, creating challenges when it comes to scalability, interoperability, and user-friendliness.  
1. **Challenge**: Operators often need to interact with complex HMIs to control equipment or simulations manually.  
2. **Solution**: What if you could issue natural language commands like, "Start the simulation" or "Stop the pump," and have the system interpret and execute these commands seamlessly?  
3. **Goal**: Combine modular, Dockerized OT systems with LLMs to make these interactions intuitive, while ensuring reliability and scalability for industrial workflows.

---

## Workflow Summary

### **Technologies Used**
- **Python** (Flask, Pydantic, OPC UA): For the back-end, OPC UA server, and simulation engine.
- **Node.js**: For a simple front-end to interact with the system.
- **OpenAI API**: For natural language processing and interpreting commands to control the simulation.
- **Docker & Docker Compose**: For containerizing the entire stack, ensuring modularity and ease of deployment.
- **PostgreSQL** and **InfluxDB** (optional): For storing metadata and process data.

---

## Key Features of the System

### 1. **Custom Tank Simulation Engine**
The back-end includes a tank simulation engine that models a simple fill/drain process:
- **Fill Cycle**: Opens an inlet valve and fills the tank until the high-level switch is triggered.
- **Drain Cycle**: Closes the inlet valve, opens the outlet valve, and starts a pump to drain the tank until the low-level switch is triggered.

The simulation runs on an OPC UA server, making it accessible to external clients for real-time monitoring.

---

### 2. **LLM-Driven Control**
Operators can issue natural language commands such as:
- *"Start the simulation"*  
- *"Stop the pump"*  

The system sends these commands to OpenAI’s GPT-3.5 or GPT-4 API, which interprets the intent and determines whether to start or stop the simulation.

---

### 3. **Dockerized Architecture**
The project is fully containerized with Docker:
- **Back-End**: Runs the simulation, OPC UA server, and Flask API for LLM interactions.
- **Front-End**: Provides a simple Node.js-based web interface to send commands and monitor status.
- **Databases**: Optional PostgreSQL and InfluxDB containers store metadata and time-series data for expanded functionality.

Using Docker Compose, the entire stack can be deployed and orchestrated with a single command.

---

## Step-by-Step Workflow

### **1. Create the Simulation Engine**
Using Python’s `opcua` library, I built a simulation engine that:
- Creates an OPC UA server.
- Publishes tank status (level, inlet valve state, outlet valve state, pump status) as OPC UA variables.
- Manages a state machine for the tank’s fill/drain logic based on configurable thresholds.

The state machine logic ensures realistic behavior, such as stopping the pump when the low-level threshold is reached.

---

### **2. Integrate OpenAI for Natural Language Commands**
The system leverages OpenAI’s API to interpret natural language input. Here’s how:
- A user enters a command like “Start the simulation” via the front-end.
- The front-end sends the input to the back-end’s Flask API.
- The back-end forwards the input to OpenAI’s GPT model, which determines whether the user wants to start, stop, or perform another action.
- The result is validated and used to control the simulation.

---

### **3. Build the Front-End**
The front-end, built with Node.js and Express, serves a simple web interface:
- A form to input natural language commands.
- Buttons to view the current simulation status or interact with the system.

The front-end communicates with the back-end API to process commands and retrieve real-time status.

---

### **4. Containerize the System**
Using Docker, I containerized each component:
- **Python Back-End**: Runs the OPC UA server, simulation logic, and Flask API.
- **Node.js Front-End**: Serves the user interface and communicates with the back-end.
- **Databases**: PostgreSQL and InfluxDB (optional) for metadata and process data storage.

Docker Compose ties everything together, ensuring seamless orchestration across services.

---

## Challenges and Solutions

### **Challenge 1: LLM Interpretation**
LLMs can sometimes produce ambiguous responses. For instance, "Should I start the simulation now?" might be misinterpreted as a directive.
- **Solution**: I used a system message to instruct the LLM to respond only with clear directives like “start” or “stop.”

### **Challenge 2: Modular Scalability**
The system needed to support future expansion, such as additional tanks or more complex control logic.
- **Solution**: The architecture is modular, with the simulation engine, API, and front-end operating independently in containers.

---
