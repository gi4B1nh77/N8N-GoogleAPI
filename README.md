# N8N-GoogleAPI

### 1.Creat key service account and connect to Google Service

![image](https://github.com/user-attachments/assets/77fd1ba5-935b-469e-b753-77623264c545)

- Access this page, then create new service account, next we navigate to key and get the new one. After that we will download a file that contain policy

![image](https://github.com/user-attachments/assets/c2628dd8-bcbb-4fc0-9227-7721705abf0b)

- Next, go to Google Sheet and share with the email of the service account

### 2. Prepare python env

- Access bash on n8n container:
~~~bash
docker exec -it --user root n8n /bin/sh
~~~

- Install python and dependencies:
~~~bash
apk add --no-cache python3 py3-pip
python3 --version
pip3 --version
~~~

- Create env in Python:
~~~bash
 python3 -m venv /app/venv
. /app/venv/bin/activate
~~~

- Install GoogleAPI
~~~bash
pip install google-auth google-auth-oauthlib google-auth-httplib2 google-api-python-client
python -c "import google.oauth2.service_account; print('Google module is installed')"
~~~

### 3. Create `update_google_sheet.py`:

- The Sheet ID is the string between /d/ and /edit/, copy it
- The service account is the json you download before 
~~~bash
import sys
import json
from google.oauth2.service_account import Credentials
from googleapiclient.discovery import build

# Load JSON input from stdin
try:
    input_data = json.loads(sys.stdin.read())  # Ensure JSON is correctly loaded
except json.JSONDecodeError as e:
    print(json.dumps({"success": False, "error": f"Invalid JSON input: {str(e)}"}))
    sys.exit(1)

# Google Service Account Key (You can load it from a file instead)
service_account_key_path = "/home/peter/service_account.json"

# Authenticate with Google Sheets API
creds = Credentials.from_service_account_file(service_account_key_path, scopes=["https://www.googleapis.com/auth/spreadsheets"])
service = build("sheets", "v4", credentials=creds)

# Google Sheets Details
spreadsheet_id = "1socVXNSb2K8Ik3Au3OuDDOafYMGh2GTOpNg2kMLRPSU"
sheet_name = "Management"

# Ensure input_data is a list
if not isinstance(input_data, list):
    input_data = [input_data]  # Wrap in a list if it's a single dictionary

# Extract values to update
values = [[entry["Date"], entry["Month"], entry["Money_In"]] for entry in input_data]
body = {"values": values}

# Append data to Google Sheets
service.spreadsheets().values().append(
    spreadsheetId=spreadsheet_id,
    range=f"{sheet_name}!A:C",
    valueInputOption="USER_ENTERED",
    insertDataOption="INSERT_ROWS",
    body=body
).execute()

print(json.dumps({"success": True, "message": "Data updated successfully"}))
~~~~

### Exec node
![image](https://github.com/user-attachments/assets/57dcdb85-e90f-4d41-b4ff-d23c22ac2d76)

- The command bellow executes in an env venv then uses the path/to/file.py
~~~bash
python3 -m venv /app/venv
. /app/venv/bin/activate
echo '[{"Date": "{{ $json.message.formatted_date }}", "Month": "{{ $json.message.month }}", "Money_In": {{ $json.message.text }}}]' | /app/venv/bin/python /home/node/update_google_sheet.py
~~~~

