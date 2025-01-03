import requests
import os
import urllib.parse as parse

# Tableau Server details
tableau_server = "https://<your-tableau-server>"
username = "<your-username>"
password = "<your-password>"
site_id = ""  # Leave as empty string for default site, else set your site ID
view_id = "<your-view-id>"  # The ID of the view you want to export
filter_values = ["ABC", "BSA", "AWS"]  # The list of filter values to apply

# Authenticate to Tableau Server and get the auth token
def authenticate():
    url = f"{tableau_server}/api/3.10/auth/signin"
    payload = {
        "credentials": {
            "name": username,
            "password": password,
            "site": {
                "contentUrl": site_id
            }
        }
    }
    response = requests.post(url, json=payload)
    if response.status_code == 200:
        auth_token = response.json()["credentials"]["token"]
        site_id = response.json()["credentials"]["site"]["id"]
        return auth_token, site_id
    else:
        raise Exception(f"Authentication failed: {response.status_code} - {response.text}")

# Apply filter to the Tableau view
def apply_filter(auth_token, site_id, view_id, filter_value):
    filter_name = "Local Bank"  # The actual filter name in Tableau workbook
    filter_value_encoded = parse.quote(filter_value)  # URL encode the filter value

    url = f"{tableau_server}/api/3.10/sites/{site_id}/views/{view_id}/filters"
    filter_payload = {
        "filter": {
            "field": filter_name,
            "value": filter_value_encoded
        }
    }
    
    headers = {
        'X-Tableau-Auth': auth_token
    }
    
    response = requests.post(url, headers=headers, json=filter_payload)
    
    if response.status_code == 200:
        print(f"Successfully applied filter: {filter_value}")
    else:
        print(f"Failed to apply filter {filter_value}: {response.status_code} - {response.text}")
        return False
    return True

# Export the view as a PDF
def export_pdf(auth_token, site_id, view_id, filter_value):
    url = f"{tableau_server}/api/3.10/sites/{site_id}/views/{view_id}/pdf"
    headers = {
        'X-Tableau-Auth': auth_token
    }
    
    # Export the view as a PDF
    response = requests.get(url, headers=headers)
    
    if response.status_code == 200:
        # Save the PDF to file
        file_path = os.path.join(r'C:\Users\klaajane\Desktop\Loan Concentration Print Project', f"{filter_value}.pdf")
        os.makedirs(os.path.dirname(file_path), exist_ok=True)
        with open(file_path, 'wb') as f:
            f.write(response.content)
        print(f"PDF saved for filter: {filter_value}")
    else:
        print(f"Failed to export PDF for {filter_value}: {response.status_code} - {response.text}")

# Main function to apply filters and export PDFs
def process_filters(filter_values):
    try:
        auth_token, site_id = authenticate()
        for filter_value in filter_values:
            if apply_filter(auth_token, site_id, view_id, filter_value):
                export_pdf(auth_token, site_id, view_id, filter_value)
    except Exception as e:
        print(f"Error: {e}")

# Run the process
process_filters(filter_values)
