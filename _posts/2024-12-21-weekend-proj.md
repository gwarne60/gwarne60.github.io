---
title: "Weekend Project - SLC500 with Gemini 2.0 Flash Experimental w/ Thought"
subtitle: "A vertical SLC emulation with Oracle XE MES that I would ideally never have to check on"
date: 2024-12-15
categories: [AI, Industrial Control Systems, PLC, Automation]
---

# Over the past 12 days of OpenAI Christmas and Google releases...

Over the past 12 days of OpenAI Christmas and Google releases there have been a couple huge models released lately that make end-user ease of higher level concepts 
easier than they were even last week. A good opportunity to test this was using a few other projects I've been wanting to pull together. Here's the high level as summarized by Gemini 2.0 Flash:


Key Components:

    **Backend (Python FastAPI):***

        Provides API endpoints for managing recipes and production batches.

        Connects to an Oracle database to store application data.

        Includes routes for creating recipes, scheduling and starting batches, and a basic health check.

        Uses environment variables for database configuration.

    Frontend (React):

        Provides a user interface to interact with the backend API.

        Includes pages for viewing a dashboard with Key Performance Indicators (KPIs), managing recipes, and potentially a login page (though the login page functionality is incomplete).

        Built with React, React Router, and likely utilizes fetch for API calls.

    PLC Emulator (Python FastAPI):

        Simulates a PLC by providing an API to read tag values.

        This allows testing the PLC integration without a physical PLC.

    PLC Integration (Python):

        Connects to the PLC emulator (or a real PLC) to read tag values.

        Periodically reads data from specified PLC tags and sends this data to the backend API to update batch parameters.

        Uses requests to communicate with the MES API and pycomm3 (though currently using the emulator) for potential real PLC interaction.

Overall Functionality:

The system allows users to define bread recipes and schedule production batches. As batches run, the PLC integration component simulates reading real-time data (like mixer speed and oven temperature) from the PLC and sends this data to the backend. The frontend allows users to monitor the production process and view information about recipes and batches.
User Startup Guide: Bread Simulation MES

This guide provides detailed instructions on how to set up and run the Bread Simulation MES project.

Prerequisites:

Before you begin, ensure you have the following installed on your system:

    Python 3.8 or higher: The backend, PLC emulator, and PLC integration are written in Python.

    Node.js and npm (or yarn): The frontend is built with React.

    Docker: Required for running the frontend application.

    Oracle Database: An Oracle database instance is needed for the backend to store data. You'll need the connection details (host, SID, user, password). If you don't have a dedicated instance, you can use Oracle XE in a Docker container for testing purposes.

    A suitable terminal or command prompt: For executing commands.

    (Optional, for real PLC integration): A configured Rockwell Automation PLC and knowledge of its IP address and tag names. This project currently uses an emulator, so a real PLC is not strictly required for basic functionality.
