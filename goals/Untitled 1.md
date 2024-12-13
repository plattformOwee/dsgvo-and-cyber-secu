from flask import Flask, request, jsonify, render_template

import requests

import json

  

app = Flask(__name__)

  

@app.route('/')

def index():

    return render_template('index.html')

  

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

  

    # Now trigger the update_document function

    update_response = update_document(document_id)

  

    # Return both responses

    return jsonify({"create_response": response_json, "update_response": update_response})

  
  

import json

import requests

  

def update_document(document_id):

    # API URL for the update

    url = f"https://api-testbed.scrive.com/api/v2/documents/{document_id}/update"

  

    # Load the original JSON from the newfromtemplaterresponse.json file

    json_file_path = r"C:\xampp\htdocs\newfromtemplaterresponse.json"

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

  

    return response_json  # Return JSON response

  
  

if __name__ == '__main__':

    app.run(debug=True)