




#### mongosh
cmd open a administrator
- net start MongoDB
- mongosh -u admin -p securepassword --authenticationDatabase admin


##### paths
##### copied for later
###### 1
I think if we actually want to store contracts between two people, we should have a bigger table.

This is the normal response when creating a new schuldschein from a template:


##### cmd
curl -X POST -H "Authorization: oauth_signature_method=\"PLAINTEXT\", oauth_consumer_key=\"817b9681529975b6_7995\", oauth_token=\"5be084912d289699_34284\", oauth_signature=\"8590cf62e89afcf6^&2420c5ec81ffceb2\"" -H "Content-Type: application/json" -v https://api-testbed.scrive.com/api/v2/documents/newfromtemplate/8222115557379792700
###### response
C:\Users\zavogl>curl -X POST -H "Authorization: oauth_signature_method=\"PLAINTEXT\", oauth_consumer_key=\"817b9681529975b6_7995\", oauth_token=\"5be084912d289699_34284\", oauth_signature=\"8590cf62e89afcf6^&2420c5ec81ffceb2\"" -H "Content-Type: application/json" -v https://api-testbed.scrive.com/api/v2/documents/newfromtemplate/8222115557379792700
* Host api-testbed.scrive.com:443 was resolved.
* IPv6: (none)
* IPv4: 79.125.66.216
*   Trying 79.125.66.216:443...
* Connected to api-testbed.scrive.com (79.125.66.216) port 443
* schannel: disabled automatic use of client certificate
* ALPN: curl offers http/1.1
* ALPN: server accepted http/1.1
* using HTTP/1.x
> POST /api/v2/documents/newfromtemplate/8222115557379792700 HTTP/1.1
> Host: api-testbed.scrive.com
> User-Agent: curl/8.9.1
> Accept: */*
> Authorization: oauth_signature_method="PLAINTEXT", oauth_consumer_key="817b9681529975b6_7995", oauth_token="5be084912d289699_34284", oauth_signature="8590cf62e89afcf6&2420c5ec81ffceb2"
> Content-Type: application/json
>
* Request completely sent off
< HTTP/1.1 201 Created
< Server: nginx
< Date: Thu, 10 Oct 2024 02:02:20 GMT
< Content-Type: application/json; charset=UTF-8
< Content-Length: 5179
< Connection: keep-alive
< Keep-Alive: timeout=10
< Set-Cookie: lang="en";Max-Age=31622400;expires=Sat, 11-Oct-2025 02:02:19 GMT;Path=/;Version="1";Secure
< Set-Cookie: lang-ssn="en";Max-Age=31622400;expires=Sat, 11-Oct-2025 02:02:19 GMT;Path=/;Version="1";Secure;SameSite=None;Partitioned
< X-Content-Type-Options: nosniff
< X-Scrive-UserGroupID: 2218310
< X-Scrive-UserID: 118513
< X-XSS-Protection: 1; mode=block
< Strict-Transport-Security: max-age=31536000
< Access-Control-Allow-Origin: *
< Content-Security-Policy: frame-ancestors 'self' https://*.scrive.com localhost:*; report-uri https://api-testbed.scrive.com/api/experimental/integrations/csp-violation-report?backend=kontrakcja
<
{"id":"8222115557379792724","title":"Schuldschein","parties":[{"id":"9705203","user_id":"118513","is_author":true,"is_signatory":true,"signatory_role":"signing_party","fields":[{"type":"full_name","placements":[],"description":null},{"type":"name","order":1,"value":"Luna","is_obligatory":true,"should_be_filled_by_sender":true,"editable_by_signatory":false,"placements":[],"description":null},{"type":"name","order":2,"value":"Vogl","is_obligatory":true,"should_be_filled_by_sender":true,"editable_by_signatory":false,"placements":[],"description":null},{"type":"email","value":"luna@owee.io","is_obligatory":true,"should_be_filled_by_sender":true,"editable_by_signatory":false,"placements":[],"description":null},{"type":"company","value":"owee","is_obligatory":false,"should_be_filled_by_sender":false,"editable_by_signatory":false,"placements":[],"description":null},{"type":"mobile","value":"+4917645530185","is_obligatory":false,"should_be_filled_by_sender":false,"editable_by_signatory":false,"placements":[],"description":null}],"consent_module":null,"sign_order":1,"sign_time":null,"seen_time":null,"deferred_time":null,"read_invitation_time":null,"rejected_time":null,"rejection_reason":null,"sign_success_redirect_url":null,"reject_redirect_url":null,"email_delivery_status":"unknown","mobile_delivery_status":"unknown","confirmation_email_delivery_status":"unknown","has_authenticated_to_view":false,"csv":null,"delivery_method":"email","authentications":{"view":{"name":"standard"},"sign":{"name":"standard"},"view_archived":{"name":"standard"}},"authentication_method_to_view":"standard","authentication_method_to_sign":"standard","authentication_method_to_view_archived":"standard","confirmation_delivery_method":"email","notification_delivery_method":"none","allows_highlighting":false,"hide_personal_number":false,"can_forward":false,"attachments":[],"highlighted_pages":[],"api_delivery_url":null,"is_visible":null,"document_roles":[],"sign_policy":"immediate","experimental":{"signatory_status":"document_draft"}},{"id":"9705204","user_id":null,"is_author":false,"is_signatory":true,"signatory_role":"signing_party","fields":[{"type":"name","order":1,"value":"","is_obligatory":true,"should_be_filled_by_sender":false,"editable_by_signatory":false,"placements":[],"description":null},{"type":"name","order":2,"value":"","is_obligatory":true,"should_be_filled_by_sender":false,"editable_by_signatory":false,"placements":[],"description":null},{"type":"email","value":"zach.vogl.35@gmail.com","is_obligatory":true,"should_be_filled_by_sender":true,"editable_by_signatory":false,"placements":[],"description":null},{"type":"full_name","placements":[],"description":null},{"type":"company","value":"","is_obligatory":false,"should_be_filled_by_sender":false,"editable_by_signatory":false,"placements":[],"description":null},{"type":"mobile","value":"","is_obligatory":false,"should_be_filled_by_sender":false,"editable_by_signatory":false,"placements":[],"description":null}],"consent_module":null,"sign_order":2,"sign_time":null,"seen_time":null,"deferred_time":null,"read_invitation_time":null,"rejected_time":null,"rejection_reason":null,"sign_success_redirect_url":null,"reject_redirect_url":null,"email_delivery_status":"unknown","mobile_delivery_status":"unknown","confirmation_email_delivery_status":"unknown","has_authenticated_to_view":false,"csv":null,"delivery_method":"email","authentications":{"view":{"name":"standard"},"sign":{"name":"standard"},"view_archived":{"name":"standard"}},"authentication_method_to_view":"standard","authentication_method_to_sign":"standard","authentication_method_to_view_archived":"standard","confirmation_delivery_method":"email","notification_delivery_method":"none","allows_highlighting":false,"hide_personal_number":false,"can_forward":false,"attachments":[],"highlighted_pages":[],"api_delivery_url":null,"is_visible":null,"document_roles":[],"sign_policy":"immediate","experimental":{"signatory_status":"document_draft"}}],"file":{"id":"11804613","name":"Schuldschein.pdf"},"sealed_file":null,"author_attachments":[],"ctime":"2024-10-10T02:02:20.29649Z","mtime":"2024-10-10T02:02:20.29649Z","timeout_time":null,"auto_remind_time":null,"status":"preparation","days_to_sign":90,"days_to_remind":null,"display_options":{"show_header":true,"show_pdf_download":true,"show_reject_option":true,"allow_reject_reason":true,"show_footer":true,"document_is_receipt":false,"show_arrow":true,"show_form":false,"show_form_arrow":true},"invitation_message":"","confirmation_message":"","sms_invitation_message":"","sms_confirmation_message":"","lang":"en","api_callback_url":null,"object_version":9,"access_token":"1d386b07a8567cdc","timezone":"Europe/Stockholm","tags":[],"is_template":false,"is_saved":true,"folder_id":"2320752","is_shared":false,"is_trashed":false,"is_deleted":false,"viewer":{"role":"signatory","signatory_id":"9705203"},"shareable_link":null,"template_id":"8222115557379792700","from_shareable_link":false,"experimental_features":{"user_group_to_impersonate_for_eid":null,"use_new_signview":true,"pades_evidence_file_type":"bundled","process_management":"esign","signatory_status_summary":"document_draft"}}* Connection #0 to host api-testbed.scrive.com left intact




#### how 
PECL download if .dll files are missing

### Scripts
#### scrive
###### app.py newfromtemplate > response json as json > change locally > resubmit updated json
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

  
  

def ChangeJSON():

    print("changejson started")

    # Load the JSON data from the file

    file_path = r"C:\xampp\htdocs\newfromtemplaterresponse.json"

    writepath = r"C:\xampp\htdocs\newfromtemplaterresponsenew.json"

    with open(file_path, 'r') as file:

        data = json.load(file)

  

    # Update the delivery_method

    data["delivery_method"] = "api"  # Change delivery method to "api"

  

    # Update the fields for party2

    party2 = data["parties"][1]  # Index 1 for the second party

    for field in party2["fields"]:

        if field["type"] == "name":

            if field["order"] == 1:

                field["value"] = "Zacharias"  # Set first name for party2

            elif field["order"] == 2:

                field["value"] = "Vogl"  # Set last name for party2

        elif field["type"] == "email":

            field["value"] = "luna@owee.io"  # Set email for party2

  

    # Update the fields for party3

    party3 = data["parties"][2]  # Index 2 for the third party

    for field in party3["fields"]:

        if field["type"] == "name":

            if field["order"] == 1:

                field["value"] = "Lukas"  # Set first name for party3

            elif field["order"] == 2:

                field["value"] = "Schuster"  # Set last name for party3

        elif field["type"] == "email":

            field["value"] = "lukas@owee.io"  # Set email for party3

  

    # Save the modified data back to the file

    with open(writepath, 'w') as file:

        json.dump(data, file, indent=2)

  

    print("Delivery method, Party2, and Party3 names and emails updated successfully.")

ChangeJSON()

  

def update_document(document_id):

    # API URL for the update

    print("x")

    url = f"https://api-testbed.scrive.com/api/v2/documents/{document_id}/update"

  

    # Load the original JSON from the newfromtemplaterresponse.json file

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

  

    return response_json  # Return JSON response

  

if __name__ == '__main__':

    app.run(debug=True)
   #### 

###### 2 ids -> Contract:
###### Input json: {"ids":[20,21]}

###### 