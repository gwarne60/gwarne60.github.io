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
# Gemini 2.0 Flash w/ Thought:

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
in building a state driven and s88 style architecture. But with a bit more processing power now I'm 
interested in building a reference for actual ladder logic. 

This is something I've had issues with in the past. With non-standard edge configuration for real world constraints 
(think edge node configuration) and managing maintenance of a database for low level abstractions in PLC code. After writing a simple SLC500 program for free from Rockwell, here's what Gemini 2.0 Flash w/ Thought returned:

---
# Gemini 2.0 Flash w/ Thought:
Rung-by-Rung Analysis:

Rung 0: SOR BST XIC I:0.0/0 NXB XIC O:0.0/0 BND LEQ F8:0 3.0 XIC I:0.0/1 OTE O:0.0/0 EOR

    Syntax: SOR (Start of Rung), BST (Branch Start), XIC (Examine If Closed), NXB (Next Branch), BND (Branch End), LEQ (Less Than or Equal To), OTE (Output Energize), EOR (End of Rung).

    Logic: This rung controls the pump run output (O:0/0). It has two parallel branches (OR condition) and a series condition (AND).

        Branch 1: XIC I:0.0/0 - If the pump start pb (NO) is pressed (contacts closed).

        Branch 2: XIC O:0.0/0 - If the pump run output is already energized (latching logic).

        Series Condition: LEQ F8:0 3.0 - AND the T100 Pressure (F8:0) is less than or equal to 3.0.

        Series Condition: XIC I:0.0/1 - AND the pump stop pb (NC) is NOT pressed (contacts closed).

    State Impact: If the conditions are met, the pump run output (O:0/0) will be energized.

    Observation: This rung implements a start/stop circuit with a pressure interlock. The pump can be started by pressing the start button OR if it's already running (latching). It will only run if the pressure is at or below 3.0 and the stop button is not pressed.

Rung 1: SOR BST XIC I:0.0/4 NXB XIC I:0.0/5 NXB XIC I:0.0/6 BND XIC I:0.0/2 XIC I:0.0/3 BST OTL O:0.0/1 NXB MOV N7:1 T4:0.PRE BND EOR

    Syntax: OTL (Output Latch), MOV (Move).

    Logic: This rung controls the heater on output (O:0/1) and sets the preset value of the Heater RunTime At SP timer (T4:0).

        Parallel Branches (OR):

            XIC I:0.0/4 - Heater local pb 1 (start) is pressed.

            XIC I:0.0/5 - heater pb 2 start is pressed.

            XIC I:0.0/6 - heater remote start is active.

        Series Conditions (AND):

            XIC I:0.0/2 - Heater local stop 1 is NOT pressed.

            XIC I:0.0/3 - Heater local stop 2 is NOT pressed.

        Output Latch: OTL O:0.0/1 - If the conditions are met, the heater on output (O:0/1) is latched ON. It will stay ON until unlatched.

        Move Instruction: MOV N7:1 T4:0.PRE - The value from Heater Runtime SP (remote) (N7:1) is moved to the preset value (T4:0.PRE) of the Heater RunTime At SP timer.

    State Impact: If any of the start pushbuttons are pressed AND both local stop buttons are NOT pressed, the heater on output will be energized and latched. The timer T4:0 will have its preset value updated.

    Observation: This rung implements the start/stop logic for the heater with local and remote start options, and two local stop buttons for redundancy. It also sets the runtime preset for the next rung's timer.

Rung 2: SOR XIC O:0.0/1 BST GRT F8:1 F8:2 NXB XIC T4:0/EN BND TON T4:0 1.0 30 0 EOR

    Syntax: GRT (Greater Than), TON (Timer On-Delay).

    Logic: This rung controls the Heater RunTime At SP timer (T4:0).

        XIC O:0.0/1 - The heater on output is energized.

        Parallel Branches (OR):

            GRT F8:1 F8:2 - The Heater PV (F) is greater than the Heater SP (F) (meaning the heater has reached or exceeded the setpoint).

            XIC T4:0/EN - The Heater RunTime At SP timer is already enabled (this allows the timer to continue running if the temperature fluctuates below the setpoint momentarily).

        TON T4:0 1.0 30 0 - If the conditions are met, the Timer On-Delay instruction for T4:0 is activated with:

            Preset: 30 (likely seconds, given the 1.0 Time Base).

            Accumulator: 0 (starts at 0).

    State Impact: The Heater RunTime At SP timer (T4:0) will start accumulating time when the heater is on AND either the process value is above the setpoint OR the timer is already running. The timer's Enable bit (T4:0/EN) will be set when the rung becomes true. The Done bit (T4:0/DN) will be set when the accumulator reaches the preset value.

    Observation: This rung starts a timer once the heater is on and the temperature has reached or exceeded the setpoint. The timer is likely used to track how long the heater stays at the setpoint.

Rung 3: SOR BST XIC T4:0/DN NXB XIO I:0.0/2 NXB XIO I:0.0/3 NXB GRT F8:1 250.0 BND OTU O:0.0/1 EOR

    Syntax: XIO (Examine If Open), OTU (Output Unlatch).

    Logic: This rung unlatches the heater on output (O:0/1), turning the heater OFF.

        Parallel Branches (OR):

            XIC T4:0/DN - The Heater at sp time timer (T4:0) has timed out (Done bit is set).

            XIO I:0.0/2 - Heater local stop 1 is pressed (contacts open).

            XIO I:0.0/3 - Heater local stop 2 is pressed (contacts open).

            GRT F8:1 250.0 - The Heater PV (F) is greater than 250.0 (likely a high-temperature safety limit).

        OTU O:0.0/1 - If any of these conditions are true, the heater on output (O:0/1) is unlatched, turning the heater OFF.

    State Impact: The heater on output will be turned OFF if the timer has reached its preset, or either local stop button is pressed, or if the temperature exceeds the high limit.

    Observation: This rung provides multiple ways to stop the heater: automatically after a set time at the setpoint, manually via local stop buttons, or via a high-temperature safety interlock.

Rung 4: SOR BST XIC I:0.0/7 NXB XIC O:0.0/2 BND XIC I:0.0/9 OTL O:0.0/2 EOR

    Logic: This rung controls the Agit run coil output (O:0/2).

        Parallel Branches (OR):

            XIC I:0.0/7 - Agit start pb is pressed.

            XIC O:0.0/2 - The Agit run coil output is already energized (latching logic).

        Series Condition (AND):

            XIC I:0.0/9 - Agit stop pb is NOT pressed.

        OTL O:0.0/2 - If the conditions are met, the Agit run coil output is latched ON.

    State Impact: The Agit run coil output will be energized and latched if the start button is pressed OR if it's already running, AND the stop button is not pressed.

    Observation: This is a standard start/stop latching circuit for the agitator.

Rung 5: SOR XIC O:0.0/2 BST MOV N7:3 O:0.1 NXB ONS B3:0/0 MOV N7:2 T4:1.PRE BND EOR

    Syntax: ONS (One-Shot Rising).

    Logic: This rung controls the agitator speed setting and sets the preset for the agitator runtime timer.

        XIC O:0.0/2 - The Agit run coil is energized.

        Branch 1: MOV N7:3 O:0.1 - Move the value from Agit % SP (remote) (N7:3) to the Agit Speed Pot (O:0.1). This likely controls the speed of the agitator via an analog output.

        Branch 2: ONS B3:0/0 MOV N7:2 T4:1.PRE - When the Agit run coil transitions from OFF to ON (rising edge detected by the ONS instruction on bit B3:0/0), the value from Agit Runtime SP (remote) (N7:2) is moved to the preset value (T4:1.PRE) of the Agitator Runtime SP timer. The B3:0/0 bit acts as a one-shot to only execute the move once per start.

    State Impact: When the agitator is started, its speed will be set according to the remote setpoint, and the Agitator Runtime SP timer's preset value will be updated.

    Observation: This rung sets the agitator speed and configures the runtime timer based on remote settings.

Rung 6: SOR XIC O:0.0/2 TON T4:1 1.0 90 29 EOR

    Logic: This rung starts the Agitator Runtime SP timer (T4:1).

        XIC O:0.0/2 - The Agit run coil is energized.

        TON T4:1 1.0 90 29 - If the agitator is running, the Timer On-Delay instruction for T4:1 is activated with:

            Preset: 90 (likely seconds).

            Accumulator: 29 (This suggests the timer might retain its accumulated value or be initialized with this value - check PLC documentation for specifics).

    State Impact: The Agitator Runtime SP timer (T4:1) will start accumulating time when the agitator is running.

    Observation: This timer tracks the runtime of the agitator.

Rung 7: SOR XIC O:0.0/2 XIO I:0.0/8 TON T4:2 1.0 10 0 EOR

    Logic: This rung starts another timer, potentially related to agitation.

        XIC O:0.0/2 - The Agit run coil is energized.

        XIO I:0.0/8 - The Agitator Running feedback is NOT active.

        TON T4:2 1.0 10 0 - If the agitator is running BUT there's no running feedback, the Timer On-Delay instruction for T4:2 is activated with:

            Preset: 10 (likely seconds).

            Accumulator: 0.

    State Impact: Timer T4:2 will start accumulating time if the agitator is commanded ON but the feedback signal indicating it's running is missing.

    Observation: This rung likely implements a fault detection mechanism. If the agitator is supposed to be running but the feedback is absent, this timer starts.

Rung 8: SOR BST XIC T4:2/DN NXB BST XIO I:0.0/9 NXB XIC T4:1/DN BND BND OTU O:0.0/2 EOR

    Logic: This rung unlatches the Agit run coil output, stopping the agitator.

        Outer Parallel Branches (OR):

            XIC T4:2/DN - The fault detection timer (T4:2) has timed out, indicating a problem with the agitator feedback.

            Inner Parallel Branches (OR):

                XIO I:0.0/9 - The Agit stop pb is pressed.

                XIC T4:1/DN - The Agitator Runtime SP timer (T4:1) has timed out, indicating the programmed runtime has been reached.

        OTU O:0.0/2 - If any of these conditions are true, the Agit run coil output is unlatched, stopping the agitator.

    State Impact: The agitator will stop if the fault detection timer times out, the stop button is pressed, or the programmed runtime is reached.

    Observation: This rung provides multiple ways to stop the agitator: due to a fault, manually, or after the set runtime.

Rung 9: SOR END EOR

    Syntax: END - Marks the end of the program.

Executive Summary of Control Logic:

This ladder logic controls three main unit operations: Pumping, Heating, and Agitation.

    Pumping: The pump is started by pressing the start button and will continue to run as long as the stop button is not pressed and the pressure is within the allowed range (<= 3.0). It includes a latching mechanism to keep it running.

    Heating: The heater can be started via multiple local or remote start pushbuttons. It has two local stop buttons for safety. Once started, it heats until the temperature reaches the setpoint. A timer then starts to track how long the temperature stays at the setpoint. The heater will turn OFF automatically after this timer expires, if a stop button is pressed, or if a high-temperature limit is reached. The runtime of the heater is determined by a remote setpoint.

    Agitation: The agitator is started and stopped using dedicated start and stop pushbuttons with a latching mechanism. Its speed is controlled by a remote analog output setpoint. A timer tracks the programmed runtime of the agitator. Another timer acts as a fault detection mechanism, checking for the presence of a running feedback signal. The agitator will stop automatically after its programmed runtime, if the stop button is pressed, or if a fault is detected (no feedback).
---

