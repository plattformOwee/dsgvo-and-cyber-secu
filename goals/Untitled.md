from flask import Flask, request, jsonify, render_template

import requests

import json

  

app = Flask(__name__)

  

@app.route('/')

def index():

    return render_template('index.html')

  

def ChangeJSON():

    print("changejson started")

    # Load the JSON data from the file

    file_path = r"C:\xampp\htdocs\newfromtemplaterresponse.json"

    writepath = r"C:\xampp\htdocs\newfromtemplaterresponsenew.json"

    with open(file_path, 'r') as file:

        data = json.load(file)

  

    # Update the global delivery_method

    data["delivery_method"] = "api"  # Change delivery method to "api"

  

    # Update the delivery_method for party 2 and party 3 to "api"

    party2 = data["parties"][1]  # Index 1 for the second party

    party3 = data["parties"][2]  # Index 2 for the third party

  

    party2["delivery_method"] = "api"  # Set delivery method to "api" for party2

    party3["delivery_method"] = "api"  # Set delivery method to "api" for party3

  

    # Update the fields for party2

    for field in party2["fields"]:

        if field["type"] == "name":

            if field["order"] == 1:

                field["value"] = "Zacharias"  # Set first name for party2

            elif field["order"] == 2:

                field["value"] = "Vogl"  # Set last name for party2

        elif field["type"] == "email":

            field["value"] = "luna@owee.io"  # Set email for party2

  

    # Update the fields for party3

    for field in party3["fields"]:

        if field["type"] == "name":

            if field["order"] == 1:

                field["value"] = "Lukas"  # Set first name for party3

            elif field["order"] == 2:

                field["value"] = "Schuster"  # Set last name for party3

        elif field["type"] == "email":

            field["value"] = "lukas@owee.io"  # Set email for party3

  

    # Set the same sign_success_redirect_url for each party

    for party in data["parties"]:

        party["sign_success_redirect_url"] = "https://fahrschule-peppermint.com/"

  

    # Save the modified data back to the file

    with open(writepath, 'w') as file:

        json.dump(data, file, indent=2)

  

    print("Delivery method, Party2, Party3 names and emails, and sign_success_redirect_url updated successfully.")

  
  

def startSigning(document_id):

    print("Starting the signing process...")

  

    # API URL to start the signing process

    url = f"https://api-testbed.scrive.com/api/v2/documents/{document_id}/start"

  

    # Set the required headers including authorization

    headers = {

        'Authorization': 'oauth_signature_method="PLAINTEXT", oauth_consumer_key="70905cd9f94e77da_8036", oauth_token="adc612e3b83be957_34795", oauth_signature="0aab8ec0352747e5&d28efc845d128d91"',

        'Content-Type': 'application/json',

        'Cookie': 'lang="en"; lang-ssn="en"'

    }

  

    # Make the POST request to start the signing process

    response = requests.post(url, headers=headers)

  

    # Get JSON response

    response_json = response.json()

  

    # Save the response to signstart.json

    signstart_file_path = r"C:\xampp\htdocs\signstart.json"

    with open(signstart_file_path, 'w') as json_file:

        json.dump(response_json, json_file, indent=2)  # Pretty print JSON

  
  

    print(f"Signing process started successfully. Response saved to {signstart_file_path}.")

    [chatgpt:

     "

     load the api_delivery_url from the json

     1. "with open(file_path, 'r') as file:

        data = json.load(file)"]

    2. party2 = data["parties"][1]  # Index 1 for the second party

    party3 = data["parties"][2]

    3. delivery_url1 = party2["api_delivery_url"] delivery_url2 = party3["api_delivery_url"]

    4. save the two urls in two .json files

    "

    ]

    # Return JSON response for further use

    return response_json

  

def ConnectTheUser(api_deliver_url1, api_delivery_url2)

    [chatgpt: "add code here to connect to flutter app and explain how"]

  
  
  

@app.route('/submit', methods=['POST'])

def submit():

    # Create Document from Template

    url = "https://api-testbed.scrive.com/api/v2/documents/newfromtemplate/8222115557379851728/"

    payload = {}

    headers = {

        'Authorization': 'oauth_signature_method="PLAINTEXT", oauth_consumer_key="70905cd9f94e77da_8036", oauth_token="adc612e3b83be957_34795", oauth_signature="0aab8ec0352747e5&d28efc845d128d91"',

        'Cookie': 'lang="en"; lang-ssn="en"'

    }

    response = requests.post(url, headers=headers, data=payload)

    response_json = response.json()  # Get JSON response

  

    # Save the value of "id" to id.txt

    document_id = response_json.get("id")  # Extract "id" from the response

    with open(r"C:\xampp\htdocs\id.txt", 'w') as id_file:

        id_file.write(document_id)  # Save the document_id to a text file

  

    # Print the document_id

    print("Document ID:", document_id)

  

    # Save response to a file

    with open(r"C:\xampp\htdocs\newfromtemplaterresponse.txt", 'w') as txt_file:

        txt_file.write(json.dumps(response_json, indent=2))  # Pretty print JSON

  

    # Save to newfromtemplaterresponse.json

    with open(r"C:\xampp\htdocs\newfromtemplaterresponse.json", 'w') as json_file:

        json.dump(response_json, json_file)

  
  

    ChangeJSON()

    # Now trigger the update_document function

    update_response = update_document(document_id)

  

    # Return both responses

    return jsonify({"create_response": response_json, "update_response": update_response})

  
  

def update_document(document_id):

    # API URL for the update

    print("Updating document with ID:", document_id)

    url = f"https://api-testbed.scrive.com/api/v2/documents/{document_id}/update"

  

    # Load the modified JSON from the newfromtemplaterresponsenew.json file

    json_file_path = r"C:\xampp\htdocs\newfromtemplaterresponsenew.json"

    with open(json_file_path, 'r') as json_file:

        json_data = json.load(json_file)  # Load the JSON as a dictionary

  

    # Convert JSON to a string and properly escape it

    json_string = json.dumps(json_data)  # Convert JSON to string

  

    # Create the payload in the correct format

    payload = f'document={json_string}'

  

    # Set the required headers including authorization

    headers = {

        'Authorization': 'oauth_signature_method="PLAINTEXT", oauth_consumer_key="70905cd9f94e77da_8036", oauth_token="adc612e3b83be957_34795", oauth_signature="0aab8ec0352747e5&d28efc845d128d91"',

        'Content-Type': 'application/x-www-form-urlencoded',  # URL-encoded form data format

        'Cookie': 'lang="en"; lang-ssn="en"'

    }

  

    # Make the POST request with the URL-encoded payload

    response = requests.post(url, headers=headers, data=payload)  # Use 'data' for form-encoded data

    # Get JSON response

    response_json = response.json()

  

    # Save response to update_response.txt

    with open(r"C:\xampp\htdocs\update_response.txt", 'w') as update_file:

        update_file.write(json.dumps(response_json, indent=2))  # Pretty print JSON

  

    # Call startSigning and save the result as signstart.json

    signing_response = startSigning(document_id)

  

    # Return both update response and signing response

    return {"update_response": response_json, "signing_response": signing_response}

  
  
  
  

if __name__ == '__main__':

    app.run(debug=True)