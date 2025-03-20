# N8N-GoogleAPI

## Creat key service account

![image](https://github.com/user-attachments/assets/77fd1ba5-935b-469e-b753-77623264c545)

Access this page, then create new service account, next we navigate to key and get the new one. After that we will download a file that contain policy
![image](https://github.com/user-attachments/assets/f165283f-9c87-4e67-8034-b32cea02e408)
![image](https://github.com/user-attachments/assets/c2628dd8-bcbb-4fc0-9227-7721705abf0b)


## Create 'update_google_sheet.py':

- The Sheet ID is the string between /d/ and /edit/, copy it
- The service account is the json you download before 
~~~bash
import sys
import json
from google.oauth2.service_account import Credentials
from googleapiclient.discovery import build

# Load JSON input from stdin
input_data = json.load(sys.stdin)

# Google Service Account Key (You can load it from a file instead)
service_account_key_path = "/home/peter/service_account.json"

# Authenticate with Google Sheets API
creds = Credentials.from_service_account_file(service_account_key_path, scopes=["https://www.googleapis.com/auth/spreadsheets"])
service = build("sheets", "v4", credentials=creds)

# Google Sheets Details
spreadsheet_id = ""
sheet_name = ""

# Extract values to update
values = [[entry["Money_In"], entry["Date"], entry["Month"], entry["Total_Monthly"]] for entry in input_data]
body = {"values": values}

# Append data to Google Sheets
service.spreadsheets().values().append(
    spreadsheetId=spreadsheet_id,
    range=f"{sheet_name}!A:D",
    valueInputOption="USER_ENTERED",
    insertDataOption="INSERT_ROWS",
    body=body
).execute()

print(json.dumps({"success": True, "message": "Data updated successfully"}))
~~~~

