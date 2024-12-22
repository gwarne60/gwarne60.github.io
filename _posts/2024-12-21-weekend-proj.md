---
title: "Weekend Project - SLC500 with Gemini 2.0 Flash Experimental w/ Thought"
subtitle: "A vertical SLC emulation with Oracle XE MES that I would ideally never have to check on"
date: 2024-12-21
categories: [AI, Industrial Control Systems, PLC, Automation]
---

# Over the past 12 days of OpenAI Christmas and Google releases...

...there have been a couple huge models released lately that make end-user ease of higher level concepts 
easier than they were even last week. A good opportunity to test this was using a few other projects I've been wanting to pull together. Here's the high level as summarized by Gemini 2.0 Flash:

---
# Gemini 2.0 Flash:
Overall Functionality:

The system allows users to define bread recipes and schedule production batches. As batches run, the PLC integration component simulates reading real-time data (like mixer speed and oven temperature) from the PLC and sends this data to the backend. The frontend allows users to monitor the production process and view information about recipes and batches.


Prerequisites:

Before you begin, ensure you have the following installed on your system:

    Python 3.8 or higher: The backend, PLC emulator, and PLC integration are written in Python.

    Node.js and npm (or yarn): The frontend is built with React.

    Docker: Required for running the frontend application.

    Oracle Database: An Oracle database instance is needed for the backend to store data. You'll need the connection details (host, SID, user, password). If you don't have a dedicated instance, you can use Oracle XE in a Docker container for testing purposes.

    A suitable terminal or command prompt: For executing commands.

    (Optional, for real PLC integration): A configured Rockwell Automation PLC and knowledge of its IP address and tag names. This project currently uses an emulator, so a real PLC is not strictly required for basic functionality.

---

So long story short I had a front end packaged in docker, a backend running locally on the host (this is to simplify
the plc simulated Ethernet commmunications) and a two way DBE for historical and runtime data. 
this is an architecture I've been building as a side project, and overall AI has been helpful
in building a state driven and s88 style architecture. but with a bit more processing power now I'm 
interested in building a reference for actual ladder logic. 

this is something I've had issues with in the past. with non-standard edge configuration for real world constraints 
(think edge node configuration)

