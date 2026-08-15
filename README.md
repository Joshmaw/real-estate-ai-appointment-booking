# Real Estate Appointment Booking Automation

## Project Overview

This project is an AI-powered real estate appointment booking automation designed to handle property viewing requests from start to finish.

The system connects a **Vapi Voice AI Agent** to an **n8n automation workflow**, allowing prospective clients to request property viewings through a voice conversation.

The workflow processes the client's information, verifies the requested property, checks appointment availability, creates or updates client records, creates the appointment, returns the booking status to the AI agent, and continues with the required calendar, meeting, and notification processes.

The automation supports both **physical and virtual property viewings**.

---

## Project Objective

The goal of this project is to automate the appointment booking process for a real estate company and reduce the amount of manual work required from real estate representatives.

Instead of a representative manually checking property availability, recording client information, creating appointments, generating meeting links, updating records, and sending notifications, the workflow handles these processes automatically.

The AI Voice Agent acts as the first point of contact, while n8n coordinates the different systems involved in the booking process.

---

## Tools and Technologies

| Technology          | Purpose                                                              |
| ------------------- | -------------------------------------------------------------------- |
| **Vapi**            | AI voice receptionist and appointment booking interface              |
| **n8n**             | Main workflow automation and orchestration platform                  |
| **Airtable**        | Stores property, client, representative, and appointment information |
| **Google Calendar** | Creates calendar appointments                                        |
| **Google Meet**     | Provides meeting links for virtual property viewings                 |
| **Gmail**           | Sends appointment notifications and confirmations                    |
| **Webhook**         | Connects the Vapi AI Agent with the n8n workflow                     |

---

# Workflow Journey

The workflow begins when a prospective client requests a property viewing through the Vapi Voice AI Agent.

The complete workflow can be summarized as:

```text
Vapi Voice AI Agent
        ↓
     Webhook
        ↓
Clean Data from Webhook
        ↓
Client / Property Searches
        ↓
Client Verification
        ↓
Availability Check
        ↓
      IF
   ↙       ↘
Available   Unavailable
   ↓            ↓
Create       Respond to
Record        Webhook
   ↓            ↓
Respond to    Vapi AI
Webhook          ↓
   ↓          Caller
Vapi AI
   ↓
Caller receives
confirmation
   ↓
Workflow continues
   ↓
Physical / Virtual Logic
   ↓
Prepare Calendar Event
   ↓
Create Google Calendar Event
   ↓
Update Airtable Record
   ↓
Send Notifications
```

The important design principle is that the **AI agent receives the immediate booking result after the booking record has been created**, while the workflow can continue with downstream processing.

---

## 1. Receive the Booking Request

The workflow starts with the **Webhook** node.

Vapi sends a `POST` request containing the appointment information collected during the conversation.

The information includes:

* Client name
* Client phone number
* Client email
* Property name
* Appointment type
* Appointment date
* Appointment time

The webhook passes this information into the n8n workflow.

---

## 2. Clean and Prepare Incoming Data

The **Clean Data from Webhook** node extracts and organizes the information received from Vapi.

This converts the incoming webhook structure into a cleaner format that can be used throughout the workflow.

The cleaned information includes fields such as:

```text
client_name
client_phone
client_email
property_name
appointment_type
appointment_date
appointment_time
```

This makes the information easier to search, compare, store, and pass between nodes.

---

## 3. Search for Existing Client Information

The workflow searches Airtable to determine whether the client already exists in the database.

The workflow contains logic for handling both existing and new clients.

The **Search Records**, **Check Existing Client**, and **IF** nodes determine which path the client information should follow.

---

## 4. Update or Create the Client Record

If the client already exists, the workflow can update the existing Airtable record.

If the client does not exist, the workflow creates a new record.

This prevents unnecessary duplicate client records while ensuring that new clients are stored in the system.

---

# Appointment Availability Process

## 5. Verify the Property and Appointment Availability

After processing the client information, the workflow checks the requested appointment.

The availability process verifies information such as:

* Property
* Appointment date
* Appointment time
* Appointment type

The workflow uses the property search, merge, availability-checking logic, and conditional nodes to determine whether the requested slot can be booked.

---

## 6. Handle an Unavailable Appointment

If the requested appointment slot is unavailable, the workflow does **not** create an appointment record.

Instead, the workflow follows the unavailable branch and reaches the **Respond to Webhook** node.

The response returned to Vapi contains a clear unsuccessful result indicating that the requested time is unavailable.

For example:

```json
{
  "success": false,
  "appointment_status": "unavailable",
  "message": "The requested appointment time is unavailable."
}
```

The Vapi AI Agent then communicates this result to the caller and can offer to help find another available time.

This prevents the AI from incorrectly telling the caller that an unavailable appointment was booked.

---

# Successful Booking Process

## 7. Create the Appointment Record

If the requested appointment slot is available, the workflow proceeds through the available branch.

The **Create a Record** node creates the appointment record in Airtable.

The record contains the relevant booking information, including:

* Client name
* Client phone
* Client email
* Property
* Appointment type
* Appointment date
* Appointment time

This creates the booking record before the workflow returns the successful result to Vapi.

---

## 8. Immediately Return the Successful Webhook Response

Immediately after **Create a Record**, the workflow reaches the **Respond to Webhook** node for the successful booking branch.

This is an important part of the architecture.

The Vapi AI Agent does **not** wait for the entire downstream workflow to finish before receiving the booking result.

Instead, once the appointment record has been successfully created, n8n immediately returns a structured response to Vapi.

A successful response follows this structure:

```json
{
  "success": true,
  "appointment_status": "confirmed",
  "message": "Appointment successfully booked."
}
```

The AI Agent can then tell the caller that the appointment has been successfully booked.

The AI agent is instructed never to assume that an appointment was booked simply because the tool was called.

It must rely on the actual response returned by the booking workflow.

---

# Downstream Appointment Processing

## 9. Continue Processing After the Webhook Response

After the successful webhook response has been returned to Vapi, the n8n workflow continues with the remaining appointment-processing steps.

The workflow determines whether the appointment is:

* **Physical**
* **Virtual**

This allows the same booking system to support both types of property viewings.

---

## 10. Physical Property Viewing

For a physical viewing, the workflow continues with the appointment-processing logic without requiring a virtual meeting link.

The appointment information can still be processed through the calendar and notification stages so that the representative and client have the confirmed viewing details.

The physical appointment therefore follows the booking process while omitting the Google Meet requirement.

---

## 11. Virtual Property Viewing

For a virtual viewing, the workflow continues into the calendar and meeting creation process.

The system prepares the required calendar information and creates the appointment in Google Calendar.

The calendar event can generate the Google Meet information required for the virtual property viewing.

---

## 12. Prepare the Calendar Event

The **Prepare Calendar Event** node formats the appointment information into the structure required by Google Calendar.

The event information can include:

* Client
* Property
* Appointment type
* Appointment date
* Appointment time
* Representative
* Other relevant appointment details

---

## 13. Create the Google Calendar Event

The **Create an Event** node creates the appointment in Google Calendar.

For virtual appointments, the calendar event provides the Google Meet information needed for the online property viewing.

---

## 14. Update the Airtable Appointment Record

After the calendar event is created, the **Update Record** node updates the existing Airtable appointment record.

For virtual appointments, the Google Meet link generated during the calendar process is added to the appointment record.

This keeps the Airtable booking information synchronized with the calendar event.

---

# Notification Process

## 15. Send Client Confirmation

After the appointment has been processed, the workflow sends an email confirmation to the client.

The confirmation contains relevant information such as:

* Client name
* Property
* Appointment type
* Viewing date
* Viewing time
* Google Meet link for virtual appointments
* Assigned representative

For virtual appointments, the client receives the Google Meet link so they can join the viewing at the scheduled time.

---

## 16. Notify the Representative

The workflow also sends the assigned representative a notification containing the appointment details.

The representative notification includes:

* Client name
* Client phone
* Client email
* Property
* Appointment type
* Appointment date
* Appointment time
* Google Meet link where applicable

This ensures that the representative has the necessary information before the viewing.

---

# Booking Response Logic

The workflow is designed around explicit booking outcomes.

### Successful Appointment

When the requested slot is available:

```text
Availability Check
        ↓
      IF
        ↓
Create a Record
        ↓
Respond to Webhook
        ↓
Vapi AI Agent
        ↓
Caller receives confirmation

        ↓
Workflow continues
        ↓
Physical / Virtual processing
        ↓
Calendar
        ↓
Airtable Update
        ↓
Notifications
```

### Unavailable Appointment

When the requested slot is unavailable:

```text
Availability Check
        ↓
      IF
        ↓
Respond to Webhook
        ↓
Vapi AI Agent
        ↓
Caller is informed
```

No appointment record is created for an unavailable slot.

---

# AI Agent Booking Rules

The Vapi AI Agent follows several rules to prevent incorrect bookings.

Before calling the booking tool, it collects:

* Full name
* Phone number
* Email address
* Property name
* Appointment type
* Appointment date
* Appointment time

The AI asks for missing information one item at a time.

Before the booking tool is called, the AI reads the complete booking details back to the caller and asks for confirmation.

The booking tool is only called after the caller clearly confirms that the details are correct.

The AI then relies on the response returned by the booking workflow.

It does not assume that a booking was successful simply because the tool call completed.

---

# Error and Failure Handling

The workflow also accounts for unsuccessful responses.

If the requested time is unavailable, the AI informs the caller that the requested time cannot be booked.

If the booking system returns an error, timeout, empty response, or unclear response, the AI does not claim that the appointment was booked.

Instead, it informs the caller that the appointment could not be confirmed and can offer to try again.

This provides a safer and more reliable booking experience.

---

# Key Features

* AI-powered voice appointment booking
* Natural-language interaction with prospective clients
* Property verification
* Client information collection
* Existing client detection
* New client record creation
* Appointment availability checking
* Physical property viewing support
* Virtual property viewing support
* Airtable appointment management
* Google Calendar integration
* Google Meet integration
* Automated client confirmation emails
* Automated representative notifications
* Structured webhook responses
* Successful booking handling
* Unavailable-slot handling
* Error and timeout handling
* Reduced manual administrative work

---

# Project Architecture

The project uses n8n as the central orchestration layer.

```text
                ┌──────────────────┐
                │   Vapi AI Agent  │
                └────────┬─────────┘
                         │
                         ▼
                ┌──────────────────┐
                │      Webhook      │
                └────────┬─────────┘
                         │
                         ▼
                ┌──────────────────┐
                │   Data Cleaning   │
                └────────┬─────────┘
                         │
                         ▼
                ┌──────────────────┐
                │ Client / Property│
                │     Searches     │
                └────────┬─────────┘
                         │
                         ▼
                ┌──────────────────┐
                │ Availability Check│
                └────────┬─────────┘
                         │
                  ┌──────┴──────┐
                  │             │
             Available      Unavailable
                  │             │
                  ▼             ▼
           Create Record   Respond to
                  │         Webhook
                  ▼             │
           Respond to           ▼
            Webhook          Vapi AI
                  │
                  ▼
             Vapi AI
                  │
                  ▼
        Physical / Virtual Logic
                  │
                  ▼
          Calendar Processing
                  │
                  ▼
           Airtable Update
                  │
                  ▼
             Notifications
```

---

# Project Blueprint

The complete n8n workflow blueprint is available in the repository:

**`blueprint/workflow-blueprint.png`**

The blueprint provides a visual representation of the complete automation and its different decision paths.

---

# Screenshots

The `screenshots` directory contains screenshots demonstrating the major components of the implementation.

These include:

* Vapi booking tool configuration
* n8n workflow
* Webhook configuration
* Property search
* Availability checking
* Airtable record creation
* Physical appointment processing
* Virtual appointment processing
* Successful webhook response
* Unavailable appointment response
* Successful workflow execution

---

# Documentation

Detailed implementation documentation is available in:

**`documentation/project-documentation.md`**

---

# Project Outcome

The completed system transforms a traditional manual property-viewing booking process into an automated AI-powered workflow.

A prospective client can communicate with the Vapi Voice AI Agent, provide their property viewing details, and receive an appropriate response based on the actual availability and booking status.

The system does not simply collect appointment information. It verifies the property and requested slot, creates the booking record, returns the appropriate result to the AI agent, and continues with calendar, meeting, database, and notification processing.

The architecture also separates **booking confirmation** from **downstream appointment processing**. This allows the AI agent to receive the booking result immediately after the appointment record has been created while the automation continues completing the remaining administrative tasks.

---

# Project Status

**Completed and tested.**

The workflow has been tested with successful and unsuccessful appointment scenarios, including unavailable appointment slots and both physical and virtual property-viewing workflows.
