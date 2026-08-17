# Real Estate AI Appointment Booking Automation

## Project REVA - Your Real Estate Virtual Assistant


## 1. Project Overview

The **Real Estate AI Appointment Booking Automation** is an AI-powered appointment scheduling system built with **Vapi, n8n, Airtable, Google Calendar, Google Meet, and Gmail**.

The system allows a voice AI agent to receive appointment requests from prospective clients, collect and validate booking information, verify the requested property, check client records, check appointment availability, create the booking record, schedule the appointment, generate virtual meeting information where required, update Airtable, and notify the client and assigned representative.

The workflow supports:

* Physical property viewings
* Virtual property viewings
* Property validation
* Existing-client detection
* New-client creation
* Appointment availability checking
* Business-hour validation
* Sunday restriction
* Google Calendar scheduling
* Google Meet generation for virtual appointments
* Airtable record management
* Client email notifications
* Representative email notifications
* Structured JSON responses to Vapi

The system is designed to prevent invalid or unavailable appointment requests from being confirmed.

---

# 2. Project Objective

The objective of the project is to automate the real estate appointment-booking process from the initial voice conversation through appointment confirmation and notification.

Instead of requiring a staff member to manually:

1. Receive the client's appointment request
2. Collect the client's information
3. Verify the requested property
4. Check whether the client already exists
5. Check appointment availability
6. Create or update the client record
7. Create the appointment
8. Schedule the calendar event
9. Generate a Google Meet link where required
10. Update the appointment record
11. Notify the client
12. Notify the assigned representative

the automation performs these tasks through an integrated workflow.

---

# 3. Technologies Used

## Vapi

Vapi provides the AI voice-agent interface.

The AI agent communicates with the caller, collects the required booking information, confirms the details with the caller, and calls the `book_appointment` function.

The function sends:

```text
client_name
client_phone
client_email
property_name
appointment_type
appointment_date
appointment_time
```

---

## n8n

n8n acts as the central workflow automation and orchestration platform.

It receives the Vapi webhook request, cleans and processes the data, communicates with Airtable and Google Calendar, determines the availability result, and returns a structured response to Vapi.

---

## Airtable

Airtable functions as the operational database.

It stores information relating to:

* Clients
* Properties
* Appointments
* Representatives
* Appointment availability
* Appointment status
* Calendar information
* Google Meet information

---

## Google Calendar

Google Calendar is used to create calendar events for confirmed appointments.

For virtual appointments, the calendar workflow generates Google Meet information where applicable.

---

## Google Meet

Google Meet is used for virtual property viewings.

The generated meeting link is subsequently stored in the Airtable appointment record.

---

## Gmail

Gmail is used for appointment-related notifications.

The workflow sends:

* Client confirmation emails
* Representative notification emails

---

# 4. Business Booking Rules

The appointment system follows the company's inspection schedule.

## Inspection Days

Property inspections are available:

```text
Monday
Tuesday
Wednesday
Thursday
Friday
Saturday
```

Sunday is not an available inspection day.

## Inspection Hours

Inspection appointments are available between:

```text
10:00 AM and 3:00 PM
```

Therefore:

```text
Monday–Saturday: 10:00 AM–3:00 PM
Sunday: Unavailable
```

If a caller requests an appointment on Sunday, the AI agent must inform the caller that inspections are not available on Sundays.

If a caller requests a time outside the inspection hours, the AI agent must inform the caller that the requested time is unavailable for inspection.

The agent should offer to help the caller choose another valid date or time.

These business-hour rules should be enforced before attempting to book the appointment.

---

# 5. Overall Workflow Architecture

The workflow consists of the following major stages:

```text
Vapi AI Voice Agent
        ↓
Webhook
        ↓
Clean Data from Webhook
        ↓
Search Property
        ↓
Property Validation
        ↓
Search Existing Client
        ↓
Check Existing Client
        ↓
Update Existing Client
        OR
Create New Client
        ↓
Merge Booking Data
        ↓
Check Appointment Availability
        ↓
Availability IF
        ↓
 ┌───────────────────────────────┐
 │                               │
UNAVAILABLE                   AVAILABLE
 │                               │
 ↓                               ↓
Respond to Webhook          Create Appointment
                                   Record
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

The successful webhook response is deliberately positioned immediately after the appointment record is created.

---

# 6. Vapi AI Booking Prompt

The AI agent uses the following booking rules.

```text
BOOKING RULES

When the caller wants to schedule an appointment:

1. Collect the following information:
   - Full name
   - Phone number
   - Email address
   - Property name
   - Appointment type: physical or virtual
   - Appointment date
   - Appointment time

2. Ask for missing information one item at a time.
   Never ask again for information the caller has already provided.

3. Inspection appointments are available Monday through Saturday only.

4. Inspection hours are from 10:00 AM to 3:00 PM.

5. If the caller selects Sunday:
   - Do not call book_appointment.
   - Tell the caller that inspections are unavailable on Sundays.
   - Offer to help find another available date.

6. If the caller selects a time outside 10:00 AM to 3:00 PM:
   - Do not call book_appointment.
   - Tell the caller that the requested time is outside the company's inspection hours.
   - Offer to help find another available time.

7. Before calling book_appointment, read the complete booking details back to the caller and ask for confirmation.

8. ONLY call book_appointment after the caller clearly confirms that all booking details are correct.

9. Send the exact confirmed values to book_appointment:
   - client_name
   - client_email
   - client_phone
   - property_name
   - appointment_date
   - appointment_time
   - appointment_type

10. After calling book_appointment, WAIT for the tool response.
    Do not assume the appointment was booked.

11. The book_appointment tool response is the ONLY authority for whether the appointment was successfully booked.

12. If the tool returns:
    success: true

    Tell the caller:
    "Your appointment has been successfully confirmed."

13. If the tool returns:
    success: false
    AND appointment_status: "unavailable"

    Tell the caller that the requested appointment time is unavailable and offer to help find another available time.

14. If the tool returns:
    success: false
    for any other reason:

    Tell the caller that the appointment could not be confirmed.
    Do not say that the appointment was booked.

15. If the tool returns an error, timeout, empty response, missing response, or unclear response:

    DO NOT claim that the appointment was booked.

    Tell the caller:

    "I couldn't confirm the appointment because the booking system did not return a valid confirmation. Would you like me to try again?"

16. NEVER say that an appointment is booked, confirmed, or successfully scheduled unless the tool explicitly returns:

    success: true

IMPORTANT:
- Do not treat the tool call itself as confirmation.
- Do not treat a successful HTTP request as confirmation.
- Do not treat appointment_status: "confirmed" by itself as confirmation.
- Do not infer success from any other field.
- Only success: true means the appointment was successfully booked.
```

---

# 7. Webhook Configuration

The workflow begins with an n8n Webhook node.

### HTTP Method

```text
POST
```

### Path

```text
real-estate-booking
```

### Authentication

```text
None
```

### Response Mode

```text
Using 'Respond to Webhook' Node
```

This allows different branches of the workflow to return different responses to Vapi.

The response is returned as JSON.

---

# 8. Data Received From Vapi

The Vapi `book_appointment` function sends the booking information to the n8n webhook.

Example function arguments:

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

The actual Vapi webhook structure contains a `toolCallList`.

The relevant path for the appointment type is:

```text
{{ $json.body.message.toolCallList[0].function.arguments.appointment_type }}
```

It is important not to use:

```text
toolCalls
```

when the actual incoming payload contains:

```text
toolCallList
```

The distinction was identified during real workflow testing.

---

# 9. Clean Data From Webhook

The `Clean data from Webhook` stage extracts the important booking information into a simpler structure.

The resulting fields include:

```text
client_name
client_phone
client_email
property_name
appointment_type
appointment_date
appointment_time
```

This simplified structure is then used throughout the workflow.

---

# 10. Property Search

The workflow searches Airtable for the requested property.

The property name supplied by the AI agent is used to locate the corresponding Airtable property record.

The property record provides information required for the appointment process.

The workflow should only continue when the requested property can be identified.

---

# 11. Property Name Matching

Property names must correspond correctly with the property records.

For example:

```text
Maple Crest
```

and:

```text
Maplecrest
```

may be treated as different values when the Airtable search uses exact matching.

During testing, this difference was confirmed as an important consideration.

The AI agent should therefore use the property name exactly as provided by the available property information whenever possible.

---

# 12. Property Validation

After the property search, an IF node validates the result.

If the property exists and the required property information is available, the workflow continues.

If the property cannot be identified, the workflow should not continue into appointment creation.

This prevents appointments from being created for invalid or unidentified properties.

---

# 13. Existing Client Search

The workflow searches Airtable to determine whether the client already exists.

The search uses client information to identify an existing record.

The result determines whether the workflow should update an existing client record or create a new one.

---

# 14. Existing Client Logic

The `Check Existing Client` stage evaluates the client search.

The workflow supports two paths.

### Existing Client

If the client already exists:

```text
Update existing client record
```

### New Client

If the client does not exist:

```text
Create new client record
```

This reduces duplicate client records.

---

# 15. Client Information

The client record contains information such as:

```text
Client Name
Client Phone
Client Email
Property Name
Appointment Type
Appointment Date
Appointment Time
```

The resulting record becomes part of the appointment-booking process.

---

# 16. Appointment Availability Check

After the property and client information have been processed, the workflow checks whether the requested appointment slot is available.

The availability process considers information such as:

```text
Appointment Date
Appointment Time
Appointment Type
Property
```

The availability check prevents the workflow from creating an appointment for a slot that has already been taken or is otherwise unavailable.

---

# 17. Availability IF Logic

The availability IF node evaluates the availability result.

A representative condition used in the workflow is:

```text
{{ $json.fields["Physical Viewing Available"] }}
```

with the comparison:

```text
is equal to
true
```

For the appointment type, the incoming Vapi value is taken from:

```text
{{ $json.body.message.toolCallList[0].function.arguments.appointment_type }}
```

and compared against:

```text
physical
```

The workflow must use the actual JSON structure returned by the live webhook execution.

---

# 18. Unavailable Appointment Response

If the requested appointment cannot be booked, the workflow returns a structured JSON response.

```json
{
  "success": false,
  "appointment_status": "unavailable",
  "message": "The requested appointment time is unavailable. Would you like help finding another available time?"
}
```

The AI agent then communicates the unavailability to the caller and can help find another time.

This response was successfully tested during the project.

---

# 19. Successful Booking Record

If the appointment is available, the workflow creates the appointment record in Airtable.

The creation of this record represents the successful booking stage of the workflow.

The successful Respond to Webhook node is positioned immediately after the `Create a record` node.

---

# 20. Successful Respond to Webhook

The successful response uses valid JSON.

```json
{
  "success": true,
  "appointment_status": "confirmed",
  "message": "Appointment successfully booked."
}
```

The most important field is:

```json
"success": true
```

The Vapi prompt explicitly states that `success: true` is the only authoritative confirmation of a successful booking.

---

# 21. Why the Successful Response Comes Immediately After Create a Record

The successful Respond to Webhook node is intentionally placed immediately after `Create a record`.

This allows Vapi to receive the booking confirmation without waiting for the remaining automation.

After the response is returned, the workflow continues with downstream processing such as:

```text
Physical / Virtual Logic
        ↓
Prepare Calendar Event
        ↓
Google Calendar
        ↓
Update Airtable
        ↓
Email Client
        ↓
Email Representative
```

This reduces perceived response time for the caller.

### Important implementation distinction

The immediate response confirms that the booking record was successfully created.

It does not mean that every downstream operation has already completed.

---

# 22. Physical and Virtual Appointment Logic

The workflow supports:

```text
physical
virtual
```

The appointment type determines the downstream handling.

### Physical Appointment

A physical viewing does not require a Google Meet link.

### Virtual Appointment

A virtual viewing requires meeting information.

The calendar workflow generates the Google Meet information, which is subsequently stored in the Airtable appointment record.

---

# 23. Calendar Event Preparation

After the successful response has been returned, the workflow prepares the calendar event.

The calendar information includes:

```text
Property
Client
Appointment Type
Appointment Date
Appointment Time
Representative
Meeting Information where applicable
```

---

# 24. Google Calendar

The workflow creates the appointment event in Google Calendar.

The event represents the confirmed property viewing.

For virtual appointments, Google Meet information is generated as part of the calendar process.

---

# 25. Airtable Appointment Update

After the Google Calendar event is created, the workflow updates the appointment record in Airtable.

For virtual appointments, the Google Meet link is stored in the updated record.

The resulting relationship is:

```text
Airtable Appointment
        ↓
Google Calendar Event
        ↓
Google Meet
```

---

# 26. Client Confirmation Email

The client receives a confirmation email after the appointment-processing stage.

### Subject

```text
Your {{ $('Create a record').item.json.fields['Property Name'] }} Property Viewing is Confirmed
```

### Message

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

The implementation intentionally uses:

```text
Create a record
```

for the original appointment information.

The Google Meet link is taken from:

```text
Update record
```

because the meeting link becomes available after the calendar event is created.

---

# 27. Representative Notification

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

The Google Meet link is retrieved from the updated appointment record.

The remaining appointment information can be retrieved from the original `Create a record` output.

---

# 28. Respond to Webhook Configuration

The Respond to Webhook node returns JSON.

### Response Header

```text
Content-Type: application/json
```

### Response Body — Successful Booking

```json
{
  "success": true,
  "appointment_status": "confirmed",
  "message": "Appointment successfully booked."
}
```

### Response Body — Unavailable

```json
{
  "success": false,
  "appointment_status": "unavailable",
  "message": "The requested appointment time is unavailable. Would you like help finding another available time?"
}
```

The JSON must be syntactically valid.

For example, the following is invalid:

```text
{ success: true }
```

because JSON requires quoted property names.

The correct format is:

```json
{
  "success": true
}
```

---

# 29. Response Code

The webhook uses the normal successful HTTP response for processing the request.

The HTTP status indicates that the webhook request was processed.

The JSON body communicates the actual booking result.

Therefore:

```text
HTTP response
```

and:

```text
success
appointment_status
message
```

serve different purposes.

The Vapi agent should rely on the JSON booking result rather than treating an HTTP success status alone as proof that an appointment was booked.

---

# 30. Streaming

Streaming is not required for the current implementation.

The workflow returns a completed JSON response to Vapi rather than progressively streaming the response.

---

# 31. Error and Invalid Response Handling

If the booking system returns an error, timeout, empty response, missing response, or unclear response, the AI agent must not claim that the appointment was booked.

The configured fallback response is:

```text
I couldn't confirm the appointment because the booking system did not return a valid confirmation. Would you like me to try again?
```

This prevents false confirmations.

---

# 32. Testing and Debugging

The workflow was tested through actual published Vapi calls and n8n executions.

Several issues were identified and resolved.

## Issue 1 — Invalid JSON Response

n8n initially displayed:

```text
Invalid JSON in 'Response Body' field
```

The Respond to Webhook body was corrected to valid JSON.

This was important because the workflow could execute successfully while Vapi still failed to interpret the response correctly.

---

## Issue 2 — Successful Booking Not Recognized

The workflow executed successfully, but Vapi returned:

```text
I couldn't confirm the appointment because the booking system did not return a valid confirmation.
```

The response body was corrected to explicitly return:

```json
{
  "success": true,
  "appointment_status": "confirmed",
  "message": "Appointment successfully booked."
}
```

The AI agent was then instructed to treat only:

```text
success: true
```

as confirmation.

---

## Issue 3 — Unavailable Appointment

The unavailable branch was successfully tested.

The AI agent returned:

```text
The requested appointment time is unavailable. Would you like help finding another available time?
```

This confirmed that the availability branch was correctly communicating with Vapi.

---

## Issue 4 — Property Name Mismatch

The workflow demonstrated that:

```text
Maple Crest
```

and:

```text
Maplecrest
```

can be treated as different property names when exact matching is used.

The correct property name must therefore be supplied to the property search.

---

## Issue 5 — Incorrect Vapi JSON Path

During testing, an IF node was referencing the wrong Vapi field path.

The incorrect structure referenced:

```text
toolCalls
```

while the actual incoming payload contained:

```text
toolCallList
```

The correct expression is:

```text
{{ $json.body.message.toolCallList[0].function.arguments.appointment_type }}
```

This was corrected after inspecting the actual published execution data.

The workflow subsequently executed successfully.

---

## Issue 6 — Airtable Active Field Reference

The availability/client logic also required the correct Airtable field path.

For example, where the incoming merged item exposes `Active` at the root level, the expression is:

```text
{{ $json["Active"] }}
```

Where the field is inside an Airtable `fields` object, the appropriate structure is:

```text
{{ $json.fields["Active"] }}
```

The correct expression must always match the actual structure shown in the live execution data.

This is why inspecting the **actual execution JSON** is preferable to assuming the structure from a node's expected schema.

---

# 33. Important JSON and Expression Rules

The following rules should be maintained when modifying the workflow.

### Vapi appointment type

```text
{{ $json.body.message.toolCallList[0].function.arguments.appointment_type }}
```

### Physical availability

```text
{{ $json.fields["Physical Viewing Available"] }}
```

### Root-level Active field

Where the execution exposes `Active` at the root:

```text
{{ $json["Active"] }}
```

### Create Record — Property

```text
{{ $('Create a record').item.json.fields['Property Name'] }}
```

### Create Record — Client

```text
{{ $('Create a record').item.json.fields['Client Name'] }}
```

### Create Record — Appointment Type

```text
{{ $('Create a record').item.json.fields['Appointment Type'] }}
```

### Create Record — Date

```text
{{ $('Create a record').item.json.fields['Appointment Date'] }}
```

### Create Record — Time

```text
{{ $('Create a record').item.json.fields['Appointment Time'] }}
```

### Update Record — Google Meet

```text
{{ $('Update record').item.json.fields['Meeting Link'] }}
```

### Create Record — Representative

```text
{{ $('Create a record').item.json.fields.Representative.name }}
```

The exact node names must remain unchanged if these expressions are used.

If a node is renamed, the corresponding expressions must also be updated.

---

# 34. Vapi Response Logic

The AI agent follows this decision structure:

```text
Book Appointment Tool Called
        ↓
Wait for Tool Response
        ↓
      ┌───────────────────────────┐
      │                           │
 success: true             success: false
      │                           │
      ↓                    ┌──────┴───────────┐
Confirmed              unavailable        other failure
                             │                   │
                             ↓                   ↓
                       Offer another       Say booking
                       available time      could not be confirmed
```

If the tool response is missing, empty, unclear, or produces an error:

```text
Do not claim success.
```

Instead:

```text
I couldn't confirm the appointment because the booking system did not return a valid confirmation. Would you like me to try again?
```

---

# 35. Final Workflow Journey

The complete implementation is:

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
8. Update Existing Client
        OR
   Create New Client
        ↓
9. Merge Booking Data
        ↓
10. Check Appointment Availability
        ↓
11. Availability IF
        ↓
 ┌───────────────────────┐
 │                       │
UNAVAILABLE           AVAILABLE
 │                       │
 ↓                       ↓
Respond to           Create a Record
Webhook                  ↓
                    Respond to Webhook
                       SUCCESS
                         ↓
                Physical / Virtual IF
                         ↓
                Prepare Calendar Event
                         ↓
                   Google Calendar
                         ↓
                  Update Airtable
                         ↓
                    Email Client
                         ↓
               Email Representative
```

The unavailable branch terminates the booking process without creating the appointment.

The successful branch returns the booking confirmation immediately after the appointment record is created and then continues with downstream calendar, Airtable, and notification operations.

---

# 36. Key Design Decisions

## Immediate Webhook Response

The successful Respond to Webhook node is positioned immediately after the appointment record is created.

This allows Vapi to receive the booking confirmation without waiting for all downstream automation.

---

## Availability Before Booking

The workflow checks appointment availability before creating the appointment record.

This prevents unavailable appointment slots from being confirmed.

---

## Business-Hour Validation

The AI agent prevents bookings outside the company's inspection schedule:

```text
Monday–Saturday
10:00 AM–3:00 PM
```

Sunday appointments are not accepted.

---

## Client Record Verification

The workflow searches for existing clients before creating new records.

This reduces duplicate client records.

---

## Property Validation

The requested property must be identified before the workflow proceeds.

---

## Physical and Virtual Separation

The appointment type determines whether Google Meet information is required.

---

## Airtable as Operational Database

Airtable stores the operational information required throughout the appointment lifecycle.

---

## Calendar Integration

Google Calendar provides the scheduling layer for confirmed appointments.

---

## JSON-Based Communication

Respond to Webhook provides structured JSON responses between n8n and Vapi.

---

## Explicit Success Confirmation

The AI agent does not infer success.

Only:

```json
"success": true
```

is treated as a successful booking confirmation.

---

# 37. Project Status

The core Real Estate AI Appointment Booking Automation has been implemented and tested through actual published workflow executions.

The current system supports:

* AI voice appointment requests
* Booking confirmation
* Business-hour validation
* Sunday restriction
* Property validation
* Client search
* Existing-client handling
* New-client creation
* Appointment availability checking
* Unavailable-slot handling
* Physical appointments
* Virtual appointments
* Google Calendar event creation
* Google Meet information
* Airtable appointment updates
* Client email notifications
* Representative email notifications
* JSON webhook responses
* Vapi response handling
* Error and invalid-response handling

The workflow has been tested through real Vapi calls and published n8n executions rather than only individual node tests.

---

# 38. GitHub Repository Structure

The public GitHub repository is organized as follows:

```text
real-estate-ai-appointment-booking/
│
├── blueprint/
│   └── project blueprint screenshot
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

The `documentation` folder contains detailed technical documentation covering architecture, workflow logic, JSON communication, testing, debugging, and implementation decisions.

The root `README.md` provides a concise project overview for visitors to the public repository.

---

# 39. Conclusion

The Real Estate AI Appointment Booking Automation combines voice AI, workflow automation, database management, calendar scheduling, virtual meeting generation, and email communication into an end-to-end appointment management system.

The AI agent handles the conversation and collection of booking information, while n8n manages the underlying business logic and integrations.

The final system validates the property, identifies the client, enforces inspection-day and inspection-hour rules, checks availability, creates the appointment record, returns a structured booking response, schedules the calendar event, generates Google Meet information for virtual appointments, updates Airtable, and sends notifications.

The project demonstrates how an AI voice interface can be integrated with business automation infrastructure to create a practical and reliable real estate appointment-booking solution.
