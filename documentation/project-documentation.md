# Real Estate AI Appointment Booking Automation

## 1. Project Overview

The Real Estate AI Appointment Booking Automation is an AI-powered appointment scheduling workflow built with **Vapi, n8n, Airtable, Google Calendar, and Gmail**.

The system allows a voice AI agent to receive appointment requests from prospective clients, verify the requested property and appointment details, check availability, create or update the client's booking information, schedule the appointment, and send confirmation notifications.

The workflow supports both:

* Physical property viewings
* Virtual property viewings

The automation is designed to prevent double-booking and provide the AI voice agent with a clear response that can be communicated immediately to the caller.

---

## 2. Project Objective

The objective of the project is to automate the appointment-booking process for a real estate company.

Instead of requiring an employee to manually:

1. Receive the client's appointment request
2. Check the property
3. Check the client's existing records
4. Check appointment availability
5. Create the appointment
6. Add the appointment to a calendar
7. Notify the client
8. Notify the assigned representative

the automation performs these tasks through a connected workflow.

---

## 3. Main Technologies Used

### Vapi

Vapi provides the AI voice agent interface.

The agent collects information from the caller and sends the booking request to the n8n webhook.

The booking function sends information such as:

* Client name
* Client phone
* Client email
* Property name
* Appointment type
* Appointment date
* Appointment time

Example appointment types:

* Physical
* Virtual

---

### n8n

n8n acts as the central automation and orchestration platform.

It receives the request from Vapi, processes the information, communicates with Airtable and Google Calendar, and sends the appropriate response back to Vapi.

---

### Airtable

Airtable is used as the central database for:

* Client information
* Property information
* Appointment information
* Representative information
* Appointment status
* Calendar/meeting information

---

### Google Calendar

Google Calendar is used to create confirmed appointments.

For virtual appointments, the workflow also generates a Google Meet link.

---

### Gmail

Gmail is used to send appointment-related notifications.

The workflow can send:

* Client confirmation emails
* Representative notification emails
* Additional appointment-related notifications

---

## 4. Overall Workflow Architecture

The workflow can be divided into several major stages:

```text
Vapi AI Voice Agent
        ↓
Webhook
        ↓
Clean Data from Webhook
        ↓
Property Search
        ↓
Property Validation
        ↓
Client Search / Client Validation
        ↓
Create or Update Client Record
        ↓
Appointment Availability Check
        ↓
Availability Decision
        ↓
 ┌───────────────┴───────────────┐
 ↓                               ↓
Unavailable                    Available
 ↓                               ↓
Failure Response               Create Record
                                 ↓
                         Immediate Success Response
                                 ↓
                         Physical / Virtual Logic
                                 ↓
                         Prepare Calendar Event
                                 ↓
                         Google Calendar
                                 ↓
                         Update Airtable Record
                                 ↓
                         Email Notifications
```

---

# 5. Webhook Configuration

The workflow starts with an n8n **Webhook** node.

### HTTP Method

```text
POST
```

### Path

```text
real-estate-booking
```

### Webhook URL

```text
https://billionairehighpriest.app.n8n.cloud/webhook/real-estate-booking
```

### Authentication

```text
None
```

### Response

The webhook is configured to:

```text
Using 'Respond to Webhook' Node
```

This allows different parts of the workflow to return different responses to the Vapi AI agent.

The webhook also uses a response header:

```text
Content-Type: application/json
```

This ensures that the response sent back to the calling system is interpreted as JSON.

---

# 6. Data Received From Vapi

The Vapi function call sends the appointment information to the webhook.

An example request contains a function called:

```text
book_appointment
```

with arguments such as:

```json
{
  "appointment_date": "2026-08-15",
  "appointment_time": "10:00",
  "appointment_type": "physical",
  "client_email": "africanblackangel@gmail.com",
  "client_name": "James Justin",
  "client_phone": "05129674817",
  "property_name": "Maple Crest"
}
```

The workflow extracts these values and uses them throughout the booking process.

---

# 7. Clean Data From Webhook

The first processing stage cleans and structures the information received from the webhook.

This creates a simpler data structure that can be used by the remaining nodes.

The cleaned information includes:

* `client_name`
* `client_phone`
* `client_email`
* `property_name`
* `appointment_type`
* `appointment_date`
* `appointment_time`

This makes the information easier to map into Airtable and use in expressions throughout the workflow.

---

# 8. Property Search

The workflow searches Airtable for the requested property.

The property name supplied by the AI agent is used to locate the corresponding property record.

The property record provides information required for the appointment process, including property details and other relevant fields.

The property search is important because the workflow should only proceed when the requested property can be identified.

### Important implementation consideration

Property names must match the records used by the workflow.

For example:

```text
Maple Crest
```

and

```text
Maplecrest
```

may not be treated as the same value depending on the Airtable search configuration.

Therefore, the AI agent should be instructed to use the correct property name.

---

# 9. Property Validation

An IF node is used after the property search to determine whether the requested property can be processed.

If the required property information is available, the workflow continues.

If the property cannot be found or the required information is missing, the booking should not continue into the appointment creation process.

This prevents the workflow from creating appointments for invalid properties.

---

# 10. Client Search

The workflow also searches Airtable for an existing client.

The client's information is used to determine whether the person already exists in the database.

The workflow checks existing client information before deciding whether to create a new client record or update an existing one.

---

# 11. Existing Client Logic

The workflow contains a separate branch for checking whether the client already exists.

The `Check Existing Client` stage evaluates the search result.

An IF node then determines which path should be followed.

### Existing client

If the client already exists, the existing Airtable record can be updated.

### New client

If the client does not already exist, a new Airtable record is created.

This prevents unnecessary duplicate client records.

---

# 12. Create or Update Client Record

The workflow uses Airtable nodes to manage client information.

Depending on the result of the client search, it can:

* Update an existing client record
* Create a new client record

The client information includes:

```text
Client Name
Client Phone
Client Email
Property Name
Appointment Type
Appointment Date
Appointment Time
```

The resulting record becomes part of the booking process.

---

# 13. Appointment Availability Check

After the required property and client information have been processed, the workflow checks whether the requested appointment slot is available.

The availability check considers the requested:

```text
Appointment Date
Appointment Time
Appointment Type
Property
```

The purpose of this stage is to prevent the system from confirming an appointment that cannot be scheduled.

---

# 14. Availability Decision

An IF node evaluates the result of the availability check.

There are two main outcomes.

## Available

If the requested slot is available, the workflow continues with the booking process.

## Unavailable

If the requested slot is unavailable, the workflow does not create the appointment.

Instead, it sends a response back to Vapi informing the AI voice agent that the requested time is unavailable.

Example response:

```json
{
  "success": false,
  "appointment_status": "unavailable",
  "message": "The requested appointment time is unavailable. Would you like help finding another available time?"
}
```

This allows the AI agent to continue the conversation naturally with the caller.

---

# 15. Successful Booking Response

For a successful booking, the workflow uses a **Respond to Webhook** node.

An important implementation decision was made here:

> The successful Respond to Webhook node is placed immediately after the appointment record is successfully created.

This means the AI voice agent does not have to wait for the remaining downstream automation processes before receiving confirmation.

The response confirms that the booking request has been accepted successfully.

Example response:

```json
{
  "success": true,
  "appointment_status": "confirmed",
  "message": "Appointment successfully booked."
}
```

This response is returned to Vapi, allowing the AI voice agent to tell the caller that the appointment has been confirmed.

---

# 16. Why the Successful Response Comes Immediately After Create Record

The workflow continues performing additional background tasks after the booking record has been created.

These include:

* Preparing the calendar event
* Creating the Google Calendar event
* Generating the Google Meet link where applicable
* Updating the Airtable appointment record
* Sending emails

If the response to Vapi waited until all of these operations were completed, the caller could experience unnecessary delay.

Placing the successful Respond to Webhook immediately after the booking record is created allows the voice agent to receive the confirmation quickly while the remaining automation continues.

---

# 17. Physical and Virtual Appointment Logic

The workflow supports two appointment types:

```text
Physical
Virtual
```

The appointment type is passed from the AI agent and stored in the booking record.

The workflow then uses conditional logic to determine how the appointment should be handled.

### Physical Viewing

For a physical viewing, the appointment is scheduled without requiring a Google Meet meeting link.

### Virtual Viewing

For a virtual viewing, the workflow creates the appropriate calendar event and obtains the Google Meet information.

The Google Meet link is then stored in the Airtable appointment record and used in the client notification.

---

# 18. Calendar Event Preparation

After the booking has been confirmed, the workflow prepares the information required to create the calendar event.

The `Prepare Calendar Event` stage organizes the booking information into the format required by Google Calendar.

The information includes:

* Property
* Client
* Appointment type
* Date
* Time
* Representative
* Meeting information where applicable

---

# 19. Google Calendar Event

The workflow then creates an event in Google Calendar.

The event represents the confirmed property viewing.

For virtual appointments, the Google Calendar event can contain the Google Meet information.

The calendar event provides a centralized schedule for the appointment and the assigned representative.

---

# 20. Airtable Appointment Update

After the calendar event has been created, the workflow updates the Airtable appointment record.

The updated record can contain information generated during the calendar stage.

For virtual appointments, the Google Meet link is stored in the record.

This creates a connection between:

```text
Airtable Appointment Record
        ↓
Google Calendar Event
        ↓
Google Meet
```

---

# 21. Client Email Notification

After the appointment has been confirmed, the workflow sends a confirmation email to the client.

The email contains information such as:

* Client name
* Property
* Appointment type
* Viewing date
* Viewing time
* Google Meet link for virtual appointments
* Assigned representative

Example subject:

```text
Your {{ $('Create a record').item.json.fields['Property Name'] }} Property Viewing is Confirmed
```

Example message:

```text
Hello {{ $('Create a record').item.json.fields['Client Name'] }},

Your property viewing has been successfully confirmed.

Property: {{ $('Create a record').item.json.fields['Property Name'] }}

Appointment Type: {{ $('Create a record').item.json.fields['Appointment Type'] }}

Viewing Date: {{ $('Create a record').item.json.fields['Appointment Date'] }}

Time: {{ $('Create a record').item.json.fields['Appointment Time'] }}

Your Google Meet link:
{{ $('Update record').item.json.fields['Meeting Link'] }}

Representative: {{ $('Create a record').item.json.fields.Representative.name }}

Please use the Google Meet link at the scheduled time.

Thank you.
```

The Google Meet link is taken from the **Update record** node because that value is generated after the calendar event is created.

The remaining appointment information is taken from the **Create a record** node.

---

# 22. Representative Notification

The assigned representative also receives an email notification.

The notification contains:

```text
Client Name
Client Phone
Client Email
Property
Appointment Type
Appointment Date
Appointment Time
Google Meet Link
```

The representative can therefore prepare for the appointment before the scheduled time.

The Google Meet link is retrieved from the updated appointment record because it becomes available after the calendar event is created.

---

# 23. Response Handling

The workflow uses JSON responses to communicate the result of the booking process back to the Vapi AI voice agent.

### Successful booking

```json
{
  "success": true,
  "appointment_status": "confirmed",
  "message": "Appointment successfully booked."
}
```

### Unavailable appointment

```json
{
  "success": false,
  "appointment_status": "unavailable",
  "message": "The requested appointment time is unavailable. Would you like help finding another available time?"
}
```

The response body must contain valid JSON.

Invalid JSON in the Respond to Webhook node can cause the response to fail even when the rest of the workflow executes successfully.

---

# 24. Response Headers

The webhook uses:

```text
Content-Type: application/json
```

This identifies the returned data as JSON.

The workflow does not require additional response headers for the current implementation.

---

# 25. Response Code

The webhook response uses the standard successful HTTP response for successful processing.

The JSON body communicates the actual booking status to the AI agent.

The important distinction is:

```text
HTTP response
```

indicates that the webhook request was processed, while:

```text
success
appointment_status
message
```

communicate the actual business result of the booking request.

---

# 26. Streaming

Streaming is not required for the current appointment-booking workflow.

The workflow returns a completed JSON response to the Vapi agent rather than streaming the response progressively.

---

# 27. Testing and Debugging

The workflow was tested using actual Vapi calls and n8n executions.

During testing, several issues were identified and resolved.

### Issue 1: Successful booking was not being interpreted correctly

The workflow executed successfully, but the AI voice agent reported:

```text
I couldn't confirm the appointment because the booking system did not return a valid confirmation.
```

The issue was traced to the structure of the JSON response being returned by the Respond to Webhook node.

The response body was corrected to valid JSON with explicit booking status information.

---

### Issue 2: Invalid JSON

n8n reported:

```text
Invalid JSON in 'Response Body' field
```

The response body was corrected so that it contained valid JSON syntax.

This demonstrated the importance of validating the response body before relying on it as the interface between n8n and Vapi.

---

### Issue 3: Unavailable appointments

The unavailable appointment branch was successfully tested.

The AI agent received:

```text
The requested appointment time is unavailable. Would you like help finding another available time?
```

The workflow therefore correctly prevents unavailable appointments from being confirmed.

---

### Issue 4: Property name mismatch

Testing also showed that property names need to correspond correctly with the Airtable property records.

For example:

```text
Maple Crest
```

is different from:

```text
Maplecrest
```

when the search is based on exact matching.

The property search was confirmed to work when the correct property name was supplied.

---

### Issue 5: IF node field reference

An additional issue was identified in the IF node.

The IF condition was initially referencing the wrong JSON path.

The correct field had to correspond to the actual structure of the incoming data.

This was corrected after inspecting the real execution data.

The workflow subsequently executed successfully.

---

# 28. Final Workflow Journey

The complete workflow journey is:

```text
1. Vapi AI Voice Agent
        ↓
2. Webhook
        ↓
3. Clean Data from Webhook
        ↓
4. Search Property
        ↓
5. Property Validation
        ↓
6. Search Existing Client
        ↓
7. Check Existing Client
        ↓
8. Update Existing Client OR Create New Client
        ↓
9. Merge Booking Data
        ↓
10. Check Appointment Availability
        ↓
11. Availability IF
        ↓
      ┌───────────────┐
      │               │
 UNAVAILABLE       AVAILABLE
      │               │
      ↓               ↓
Respond to        Create Appointment
Webhook           Record
                      ↓
              Respond to Webhook
              (Success)
                      ↓
              Physical / Virtual
                   Logic
                      ↓
              Prepare Calendar
                   Event
                      ↓
              Google Calendar
                Event
                      ↓
              Update Airtable
                   Record
                      ↓
              Email Client
                      ↓
              Email Representative
```

The successful response is intentionally returned immediately after the appointment record is created, while the remaining calendar, Airtable, and email operations continue as part of the workflow.

---

# 29. Key Design Decisions

### Immediate webhook response

The successful Respond to Webhook node is placed immediately after Create a Record so that the AI voice agent receives confirmation without waiting for all downstream operations.

### Availability before booking

The system checks availability before creating the appointment to prevent unavailable slots from being confirmed.

### Client record verification

The workflow searches for existing clients before creating new records to reduce duplicate client records.

### Separate physical and virtual handling

The appointment type determines whether the booking requires virtual meeting information.

### Airtable as the operational database

Airtable stores the client, property, appointment, representative, and meeting information required by the workflow.

### Calendar integration

Google Calendar provides the scheduling layer for confirmed appointments.

### JSON-based communication

The Respond to Webhook nodes provide structured JSON responses that allow Vapi to understand whether a booking succeeded or failed.

---

# 30. Project Status

The core appointment-booking workflow has been successfully implemented and tested.

The current workflow supports:

* AI voice appointment requests
* Property validation
* Client search
* Existing-client handling
* New-client creation
* Appointment availability checking
* Successful booking
* Unavailable-slot handling
* Physical appointments
* Virtual appointments
* Google Calendar event creation
* Google Meet information
* Airtable record updates
* Client email notifications
* Representative email notifications
* JSON webhook responses
* Vapi response handling

The workflow has also been tested through actual published executions rather than only individual n8n node tests.

---

# 31. Project Repository Structure

The GitHub repository is organized into the following main sections:

```text
real-estate-ai-appointment-booking/
│
├── blueprint/
│
├── documentation/
│   └── project-documentation.md
│
├── screenshots/
│   ├── README.md
│   ├── availability-check.png
│   ├── create-booking-record.png
│   ├── n8n-main-workflow.png
│   ├── physical-and-virtual-appointments.png
│   ├── property-search.png
│   ├── success-response.png
│   └── successful-execution.png
│
└── README.md
```

The `blueprint` folder contains the project blueprint.

The `screenshots` folder contains screenshots documenting the workflow and its implementation.

The `documentation` folder contains detailed technical documentation describing the workflow architecture, logic, configuration, testing, and implementation decisions.

The root `README.md` provides an overview of the project for anyone visiting the public GitHub repository.

---

# 32. Conclusion

The Real Estate AI Appointment Booking Automation combines voice AI, workflow automation, database management, calendar scheduling, and email communication into a single appointment-booking system.

The workflow is designed so that the AI agent can handle the initial conversation and booking request while n8n manages the underlying business logic.

The final system provides a structured process for validating properties, identifying clients, checking appointment availability, confirming bookings, scheduling calendar events, generating virtual meeting information, updating records, and notifying the relevant people.

The project demonstrates how AI voice interfaces can be connected to business automation systems to create a practical end-to-end real estate appointment management solution.
