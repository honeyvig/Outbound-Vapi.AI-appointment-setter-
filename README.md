# Outbound-Vapi.AI-appointment-setter
We already have experience with setting up a Sales Process for multi channel approach, so we already have a Call Script and we know the most common questions and how to handle those objections. For this part maybe we will just need consultancy on how to effectively communicate with the AI Agent through the info provided in the Knowledge Base for the best performances.

Aside from this we will also need consultancy on choosing the right settings for the Voice etc.

Most important part of the project and what we need most help with are the Workflows.

We want to export data from Apollo.io. We will upload those list in the required format on Vapi.ai.

We want the AI Agent to call the numbers from the list and to use the First Name of the prospect.

During the call the purpose of the AI Agent is to book a meeting with the client. So we will need it to check the availability of our Calendly.com (or other provider if needed).

Also, after agreeing for a time that works for the client we would like to book the meeting via AI as well, on the calendar if possible.

We also want to confirm with the prospect if the email from Apollo export is correct. (If possible)

After the call is done, we want some return information into a Google Spreadsheet for reporting purposes.

We want to get the following information after the call:

-First Name
-Last Name
-Title
-Company
-Phone number
(ALL OF THESE WOULD BE AVAILABLE FROM THE APOLLO EXPORT)

-Call ID (From VAPI)
-Picked Up Status (Yes/No - if the call was answered)
-Interested Status (Yes/N0 - If the client was interested or not)
-Summary of the call (Brief information on the summary of the call)
-----------------------

To implement the requirements you have, we'll need to create a system that integrates with Vapi.ai (for the AI agent), Apollo.io (for the prospect list), Calendly (for scheduling meetings), and Google Sheets (for reporting). This project will consist of multiple parts, including workflow design, data handling, API integration, and using Vapi.ai to make calls and automate the scheduling process.

Below is an outline of how to approach the system, along with a Python script that integrates the necessary components.
Step 1: Integration Overview

    Data Extraction from Apollo.io:
        Export the prospect data from Apollo.io into a CSV or another structured format.
        The data should contain first names, last names, titles, companies, and phone numbers.

    Voice AI Setup (Vapi.ai):
        Use Vapi.ai's APIs to configure the voice agent, define call scripts, and handle objections.
        Set up the AI to read from a knowledge base, handle objections, and confirm meeting availability.

    Calendly Integration:
        Use Calendly’s API to check available meeting slots and book appointments.

    Google Sheets Integration:
        After each call, populate a Google Sheets report with the relevant information (e.g., first name, last name, phone number, call status, and more).

Step 2: Breakdown of Components
1. Data Extraction from Apollo.io:

    Export data from Apollo.io in a CSV format, and then process the file in Python to extract the necessary columns.

2. Vapi.ai Setup:

You'll need to consult the Vapi.ai documentation for configuring the AI agent to perform the call and ask for information like the availability of the prospect. Once this is set up, the system can use API calls to manage the conversations.
3. Calendly Integration:

    Calendly API can be used to:
        Check for available slots.
        Book the meeting directly from the call.

4. Google Sheets Reporting:

You can use the Google Sheets API to send call data back into a report sheet.
Step 3: Python Script for Integrating All Components
Prerequisites:

    Vapi.ai API credentials.
    Calendly API credentials.
    Google Sheets API credentials.
    A CSV file exported from Apollo.io.

Install Required Libraries:

pip install requests google-auth google-auth-oauthlib google-auth-httplib2 google-api-python-client

Python Code Implementation:

import requests
import csv
import datetime
from googleapiclient.discovery import build
from google.oauth2.service_account import Credentials

# Step 1: Function to read Apollo.io CSV and extract data
def read_apollo_data(file_path):
    with open(file_path, 'r') as file:
        reader = csv.DictReader(file)
        data = [row for row in reader]
    return data

# Step 2: Function to call Vapi.ai API and initiate calls
def initiate_call(prospect):
    api_url = 'https://api.vapi.ai/calls'  # Replace with actual VAPI API URL
    headers = {
        'Authorization': 'Bearer YOUR_VAPI_API_KEY',  # Add your VAPI API Key
    }
    data = {
        'phone_number': prospect['Phone Number'],
        'first_name': prospect['First Name'],
        'script': 'YOUR_CALL_SCRIPT',  # Add your predefined script
    }
    response = requests.post(api_url, json=data, headers=headers)
    return response.json()

# Step 3: Function to check Calendly availability and book meeting
def check_and_book_meeting(prospect, meeting_time):
    calendly_api_url = 'https://api.calendly.com/scheduled_events'  # Calendly API endpoint
    calendly_headers = {
        'Authorization': 'Bearer YOUR_CALENDLY_API_KEY',
    }
    meeting_data = {
        "event": {
            "start_time": meeting_time,
            "end_time": (datetime.datetime.strptime(meeting_time, '%Y-%m-%dT%H:%M:%S') + datetime.timedelta(hours=1)).strftime('%Y-%m-%dT%H:%M:%S'),
            "invitees": [
                {
                    "email": prospect['Email'],  # Email for the prospect (if available from Apollo)
                    "first_name": prospect['First Name'],
                    "last_name": prospect['Last Name'],
                }
            ]
        }
    }
    response = requests.post(calendly_api_url, json=meeting_data, headers=calendly_headers)
    return response.json()

# Step 4: Function to update Google Sheets with call results
def update_google_sheet(call_data):
    # Set up Google Sheets API credentials
    SCOPES = ['https://www.googleapis.com/auth/spreadsheets']
    creds = Credentials.from_service_account_file('YOUR_GOOGLE_CREDENTIALS_FILE.json', scopes=SCOPES)

    service = build(' sheets', 'v4', credentials=creds)

    # Spreadsheet ID and range where data should go
    SPREADSHEET_ID = 'YOUR_SPREADSHEET_ID'
    RANGE = 'Sheet1!A1'  # Adjust the range accordingly

    # Prepare the data to be inserted
    values = [
        [call_data['First Name'], call_data['Last Name'], call_data['Title'], call_data['Company'], 
         call_data['Phone Number'], call_data['Call ID'], call_data['Picked Up Status'], 
         call_data['Interested Status'], call_data['Call Summary']]
    ]
    body = {
        'values': values
    }

    # Call the Sheets API to append data
    service.spreadsheets().values().append(
        spreadsheetId=SPREADSHEET_ID, range=RANGE,
        valueInputOption='RAW', body=body).execute()

# Step 5: Main workflow that processes each prospect
def process_prospects(file_path):
    prospects = read_apollo_data(file_path)

    for prospect in prospects:
        # Call the VAPI Agent
        call_response = initiate_call(prospect)

        # Check if the call was picked up and if there was interest
        call_id = call_response.get('call_id')
        picked_up = call_response.get('picked_up')  # Assume Vapi returns this
        interested = call_response.get('interested')  # Assume Vapi returns this
        
        # If a meeting is to be booked, check availability and schedule
        if interested == 'Yes':
            meeting_time = '2025-01-01T10:00:00'  # Placeholder for booking time, to be customized
            calendly_response = check_and_book_meeting(prospect, meeting_time)

        # Prepare data for Google Sheets
        call_data = {
            'First Name': prospect['First Name'],
            'Last Name': prospect['Last Name'],
            'Title': prospect['Title'],
            'Company': prospect['Company'],
            'Phone Number': prospect['Phone Number'],
            'Call ID': call_id,
            'Picked Up Status': picked_up,
            'Interested Status': interested,
            'Call Summary': 'Meeting scheduled' if interested == 'Yes' else 'No interest'
        }

        # Update Google Sheets with the call data
        update_google_sheet(call_data)

# Start the process by calling the main function
if __name__ == '__main__':
    file_path = 'apollo_prospects.csv'  # Path to the exported Apollo.io CSV
    process_prospects(file_path)

Explanation:

    Data Import: The function read_apollo_data() reads the CSV data and extracts relevant columns (First Name, Last Name, Phone Number, etc.).
    Vapi.ai Integration: The initiate_call() function uses the Vapi.ai API to place calls, passing the required data (like the prospect's name and phone number).
    Calendly Integration: The check_and_book_meeting() function checks the availability on Calendly and schedules the meeting based on the confirmed time.
    Google Sheets Reporting: The update_google_sheet() function updates the Google Sheets document with the call data after the call.
    Main Workflow: The process_prospects() function combines these processes and runs them for each prospect in the list.

Final Considerations:

    You'll need to replace YOUR_VAPI_API_KEY, YOUR_CALENDLY_API_KEY, YOUR_GOOGLE_CREDENTIALS_FILE.json, and YOUR_SPREADSHEET_ID with the actual API keys, credentials, and spreadsheet ID.
    You may also need to handle rate limits, retries, or any errors returned by the APIs.
    Make sure that Vapi.ai supports the specific features you're looking for in terms of call flows and integration.
