### Detailed Information About the Script
#### (I decided not to use script. I do not need AU)
#### **Overview**

This Python script is designed to manage Administrative Units (AUs) in Azure Active Directory using the Microsoft Graph API. It enables the following functionalities:

1. **Authenticate** to Azure using a service principal.
2. **Create Administrative Units (AUs)**:
    - Predefined AUs: "HR," "Sales," and "Logistics."
    - Includes optional restrictive management settings.
3. **Save AU Metadata**: Writes AU details (e.g., ID, displayName) to a local file, `au_setup.json`.
4. **Delete Administrative Units (AUs)**:
    - Reads AU metadata from `au_setup.json`.
    - Deletes the listed AUs from Azure.

---

#### **How the Script Works**

1. **Authentication**:
    
    - Uses the `get_service_principal_auth` function to authenticate with Azure AD.
    - The script retrieves an access token using `client_credentials` grant type.
    - Required credentials are loaded from environment variables (`APP_CLIENT_ID`, `APP_CLIENT_SECRET`, `TENANT_ID`).
2. **Creating AUs**:
    
    - The `create_administrative_unit` function sends a POST request to the Microsoft Graph API `/administrativeUnits` endpoint.
    - Metadata for each AU is included in the request payload:
        - `displayName`: Name of the AU.
        - `description`: Optional description.
        - `isMemberManagementRestricted`: Boolean flag for restrictive management.
3. **Saving Metadata**:
    
    - The `save_au_metadata` function writes created AUs' metadata (e.g., IDs, names) into a file called `au_setup.json`.
4. **Deleting AUs**:
    
    - The `delete_administrative_units` function reads AU details from `au_setup.json`.
    - Sends DELETE requests to the `/administrativeUnits/{id}` endpoint for each AU listed.
5. **Command-Line Interaction**:
    
    - `create`: Creates predefined AUs ("HR," "Sales," and "Logistics").
    - `destroy`: Deletes AUs listed in `au_setup.json`.

---

#### **Setup Steps**

1. **Install Dependencies**:
    
    - Install required Python packages:
        
        ```bash
        pip install requests python-dotenv
        ```
        
2. **Environment Configuration**:
    
    - Create a `.env` file in the project directory:
        
        ```
        APP_CLIENT_ID=your_client_id
        APP_CLIENT_SECRET=your_client_secret
        TENANT_ID=your_tenant_id
        ```
        
3. **Prepare the Script**:
    
    - Ensure the script file is saved with an appropriate name, such as `link.py`.
4. **Usage**:
    
    - To create AUs:
        
        ```bash
        python3 link.py create
        ```
        
    - To delete AUs:
        
        ```bash
        python3 link.py destroy
        ```
        

---

#### **Dependencies**

1. **Files**:
    
    - `.env`: Stores sensitive credentials (`APP_CLIENT_ID`, `APP_CLIENT_SECRET`, `TENANT_ID`).
    - `au_setup.json`: Stores metadata for created AUs.
2. **Python Libraries**:
    
    - `os`: Manages environment variables.
    - `logging`: Provides logging functionality.
    - `json`: Handles JSON data.
    - `requests`: Facilitates HTTP requests to Microsoft Graph API.
    - `sys`: Manages command-line arguments.
    - `dotenv`: Loads environment variables from the `.env` file.
3. **External Tools**:
    
    - Azure App Registration: The script relies on an Azure AD application registration to generate credentials.

---

This script provides a comprehensive solution for managing Azure AUs programmatically. It requires minimal setup and ensures efficient handling of both creation and deletion tasks.


```
import os
import logging
import json
import requests
import sys
from dotenv import load_dotenv

# Configure logging to display information and error messages
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger("AzureADAdministrativeUnitManager")

# Load environment variables from the .env file
load_dotenv()

# Retrieve Azure AD App credentials from environment variables
APP_CLIENT_ID = os.getenv('APP_CLIENT_ID')
APP_CLIENT_SECRET = os.getenv('APP_CLIENT_SECRET')
TENANT_ID = os.getenv('TENANT_ID')

# Ensure all required environment variables are set
if not all([APP_CLIENT_ID, APP_CLIENT_SECRET, TENANT_ID]):
    raise EnvironmentError(
        "One or more environment variables are missing. "
        "Ensure APP_CLIENT_ID, APP_CLIENT_SECRET, and TENANT_ID are set."
    )

def get_service_principal_auth():
    """
    Authenticates with Azure AD using a service principal and retrieves an access token.

    Returns:
        str: Access token for Microsoft Graph API.

    Exits:
        The script exits if authentication fails.
    """
    try:
        # URL for obtaining the OAuth 2.0 token
        url = f"https://login.microsoftonline.com/{TENANT_ID}/oauth2/v2.0/token"
        headers = {"Content-Type": "application/x-www-form-urlencoded"}
        data = {
            "grant_type": "client_credentials",
            "client_id": APP_CLIENT_ID,
            "client_secret": APP_CLIENT_SECRET,
            "scope": "https://graph.microsoft.com/.default"
        }

        # Send POST request to obtain the access token
        response = requests.post(url, headers=headers, data=data)

        if response.status_code == 200:
            token_response = response.json()
            access_token = token_response.get("access_token")
            if access_token:
                logger.info("Access token acquired successfully using service principal.")
                return access_token
            else:
                logger.error("Access token not found in the response.")
                sys.exit(1)
        else:
            logger.error(
                f"Failed to acquire access token: {response.status_code}, {response.text}"
            )
            sys.exit(1)
    except Exception as e:
        logger.error(f"Error acquiring token using service principal: {str(e)}")
        sys.exit(1)

def create_administrative_unit(access_token, au_name, description=None, restrict_management=False):
    """
    Creates an Administrative Unit (AU) in Azure AD.

    Args:
        access_token (str): Access token for authentication.
        au_name (str): Display name of the Administrative Unit.
        description (str, optional): Description of the Administrative Unit.
        restrict_management (bool, optional): Flag to restrict member management.

    Returns:
        dict: Details of the created Administrative Unit.

    Exits:
        The script exits if AU creation fails.
    """
    try:
        # Validate inputs
        if not au_name or not isinstance(au_name, str):
            raise ValueError("Invalid Administrative Unit name.")
        if description and not isinstance(description, str):
            raise ValueError("Description must be a string.")

        # Microsoft Graph API endpoint for creating Administrative Units
        url = "https://graph.microsoft.com/beta/administrativeUnits"
        headers = {
            "Authorization": f"Bearer {access_token}",
            "Content-Type": "application/json"
        }
        payload = {
            "displayName": au_name,
            "description": description,
            "isMemberManagementRestricted": restrict_management
        }

        # Send POST request to create the AU
        response = requests.post(url, headers=headers, json=payload)

        if response.status_code == 201:
            au_details = response.json()
            logger.info(
                f"Administrative Unit '{au_name}' created successfully with ID: {au_details.get('id')}."
            )
            return au_details
        else:
            logger.error(
                f"Failed to create Administrative Unit: {response.status_code}, {response.text}"
            )
            sys.exit(1)
    except ValueError as ve:
        logger.error(f"Input validation error: {str(ve)}")
        sys.exit(1)
    except Exception as e:
        logger.error(f"Error creating Administrative Unit: {str(e)}")
        sys.exit(1)

def save_au_metadata(au_list):
    """
    Saves the metadata of created Administrative Units to a JSON file.

    Args:
        au_list (list): List of dictionaries containing AU details.

    Exits:
        The script exits if saving metadata fails.
    """
    try:
        with open("au_setup.json", "w") as file:
            json.dump({"administrative_units": au_list}, file, indent=4)
        logger.info("Administrative Unit metadata saved successfully to 'au_setup.json'.")
    except Exception as e:
        logger.error(f"Error saving Administrative Unit metadata: {str(e)}")
        sys.exit(1)

def create_predefined_administrative_units(access_token):
    """
    Creates a set of predefined Administrative Units and saves their metadata.

    Args:
        access_token (str): Access token for authentication.
    """
    # List of predefined Administrative Units to create
    predefined_aus = [
        {
            "displayName": "HR",
            "description": "Human Resources Department",
            "restrictManagement": True
        },
        {
            "displayName": "Sales",
            "description": "Sales Department",
            "restrictManagement": True
        },
        {
            "displayName": "Logistics",
            "description": "Logistics Department",
            "restrictManagement": True
        }
    ]

    created_aus = []

    # Iterate over each predefined AU and create it
    for au in predefined_aus:
        try:
            au_details = create_administrative_unit(
                access_token,
                au_name=au["displayName"],
                description=au["description"],
                restrict_management=au["restrictManagement"]
            )
            # Append the AU's ID and display name to the list for metadata saving
            created_aus.append({
                "id": au_details.get("id"),
                "displayName": au_details.get("displayName")
            })
        except Exception as e:
            logger.error(f"Failed to create AU {au['displayName']}: {str(e)}")

    # Save the metadata of all created AUs to a JSON file
    save_au_metadata(created_aus)

def delete_administrative_units(access_token):
    """
    Deletes Administrative Units based on metadata stored in 'au_setup.json'.

    Args:
        access_token (str): Access token for authentication.

    Exits:
        The script exits if deletion fails or if the metadata file is not found.
    """
    try:
        # Load the AU metadata from the JSON file
        with open("au_setup.json", "r") as file:
            au_data = json.load(file)
            administrative_units = au_data.get("administrative_units", [])

        # Check if there are any AUs to delete
        if not administrative_units:
            logger.info("No Administrative Units found in 'au_setup.json'. Nothing to delete.")
            return

        # Iterate over each AU and send a DELETE request
        for au in administrative_units:
            au_id = au.get("id")
            au_name = au.get("displayName")

            if au_id:
                url = f"https://graph.microsoft.com/v1.0/directory/administrativeUnits/{au_id}"
                headers = {"Authorization": f"Bearer {access_token}"}

                # Send DELETE request to remove the AU
                response = requests.delete(url, headers=headers)

                if response.status_code == 204:
                    logger.info(
                        f"Administrative Unit '{au_name}' with ID '{au_id}' deleted successfully."
                    )
                else:
                    logger.error(
                        f"Failed to delete Administrative Unit '{au_name}' with ID '{au_id}': {response.status_code}, {response.text}"
                    )
    except FileNotFoundError:
        logger.error("'au_setup.json' not found. Unable to delete Administrative Units.")
    except Exception as e:
        logger.error(f"Error deleting Administrative Units: {str(e)}")
        sys.exit(1)

def main():
    """
    Main function to handle command-line arguments and perform actions accordingly.
    """
    # Check if at least one command-line argument is provided
    if len(sys.argv) < 2:
        logger.error("No action specified. Use 'create' or 'destroy'.")
        sys.exit(1)

    # Retrieve the action from command-line arguments
    action = sys.argv[1].lower()

    # Validate the action
    if action not in ["create", "destroy"]:
        logger.error("Invalid action specified. Use 'create' or 'destroy'.")
        sys.exit(1)

    # Authenticate and obtain the access token
    access_token = get_service_principal_auth()

    # Perform the specified action
    if action == "create":
        # Create predefined Administrative Units
        create_predefined_administrative_units(access_token)
    elif action == "destroy":
        # Delete Administrative Units based on metadata
        delete_administrative_units(access_token)

# Entry point of the script
if __name__ == "__main__":
    main()

```



config.json
```
{
  "users": [
    { "name": "Alice Brown", "email": "alice.brown@cyberspacecorp.onmicrosoft.com", "team": "Supervisor - Shipping", "jobTitle": "supervisor", "department": "Logistics" },
    { "name": "Bob Green", "email": "bob.green@cyberspacecorp.onmicrosoft.com", "team": "Supervisor - Shipping", "jobTitle": "Representative", "department": "Call Center" },
    { "name": "Charlie White", "email": "charlie.white@cyberspacecorp.onmicrosoft.com", "team": "Call Center", "jobTitle": "Representative", "department": "Call Center" },
    { "name": "Daisy Black", "email": "daisy.black@cyberspacecorp.onmicrosoft.com", "team": "IT Support Team", "jobTitle": "IT Support", "department": "Technology" },
    { "name": "Edward Yellow", "email": "edward.yellow@cyberspacecorp.onmicrosoft.com", "team": "Cancellation", "jobTitle": "Cancellation Specialist", "department": "Cancellation" },
    { "name": "Fiona Red", "email": "fiona.red@cyberspacecorp.onmicrosoft.com", "team": "Sales", "jobTitle": "Salesperson", "department": "Sales" },
    { "name": "George Blue", "email": "george.blue@cyberspacecorp.onmicrosoft.com", "team": "Sales", "jobTitle": "Salesperson", "department": "Sales" },
    { "name": "Hannah Violet", "email": "hannah.violet@cyberspacecorp.onmicrosoft.com", "team": "Sales", "jobTitle": "Salesperson", "department": "Sales" },
    { "name": "Ian Orange", "email": "ian.orange@cyberspacecorp.onmicrosoft.com", "team": "Supervisor - Sales Team", "jobTitle": "Supervisor", "department": "Sales" },
    { "name": "Jane Gray", "email": "jane.gray@cyberspacecorp.onmicrosoft.com", "team": "Supervisor - Call Center", "jobTitle": "Supervisor", "department": "Call Center" },
    { "name": "Kyle Pink", "email": "kyle.pink@cyberspacecorp.onmicrosoft.com", "team": "Manager - Sales", "jobTitle": "Manager", "department": "Sales" },
    { "name": "Lily Gold", "email": "lily.gold@cyberspacecorp.onmicrosoft.com", "team": "Manager - Technology", "jobTitle": "Manager", "department": "Technology" },
    { "name": "Mike Silver", "email": "mike.silver@cyberspacecorp.onmicrosoft.com", "team": "Manager - Logistics", "jobTitle": "Manager", "department": "Logistics" },
    { "name": "Nina Bronze", "email": "nina.bronze@cyberspacecorp.onmicrosoft.com", "team": "Manager - HR", "jobTitle": "Manager", "department": "HR" },
    { "name": "Rachel Platinum", "email": "rachel.platinum@cyberspacecorp.onmicrosoft.com", "team": "Chief of Technology", "jobTitle": "Chief", "department": "Technology" },
    { "name": "Steve Diamond", "email": "steve.diamond@cyberspacecorp.onmicrosoft.com", "team": "Chief of Sales", "jobTitle": "Chief", "department": "Sales" },
    { "name": "Tina Sapphire", "email": "tina.sapphire@cyberspacecorp.onmicrosoft.com", "team": "Chief of Logistics", "jobTitle": "Chief", "department": "Logistics" },
    { "name": "Ursula Emerald", "email": "ursula.emerald@cyberspacecorp.onmicrosoft.com", "team": "Chief of HR", "jobTitle": "Chief", "department": "HR" },
    { "name": "Victor Onyx", "email": "victor.onyx@cyberspacecorp.onmicrosoft.com", "team": "Supervisor - Recruitment", "jobTitle": "Supervisor", "department": "Recruitment" },
    { "name": "Wendy Quartz", "email": "wendy.quartz@cyberspacecorp.onmicrosoft.com", "team": "Application Support", "jobTitle": "Application Specialist", "department": "Application Support" },
    { "name": "Xander Marble", "email": "xander.marble@cyberspacecorp.onmicrosoft.com", "team": "Networking", "jobTitle": "Network Specialist", "department": "Networking" },
    { "name": "Yasmine Coral", "email": "yasmine.coral@cyberspacecorp.onmicrosoft.com", "team": "Shipping", "jobTitle": "Shipping Specialist", "department": "Shipping" },
    { "name": "Zack Granite", "email": "zack.granite@cyberspacecorp.onmicrosoft.com", "team": "Inventory Management", "jobTitle": "Inventory Specialist", "department": "Inventory Management" },
    { "name": "Abby Amber", "email": "abby.amber@cyberspacecorp.onmicrosoft.com", "team": "Fleet Management", "jobTitle": "Fleet Specialist", "department": "Fleet Management" },
    { "name": "Jordan Lake", "email": "jordan.lake@cyberspacecorp.onmicrosoft.com", "team": "Application Support", "jobTitle": "Application Specialist", "department": "Application Support" },
    { "name": "Terry Moss", "email": "terry.moss@cyberspacecorp.onmicrosoft.com", "team": "Networking", "jobTitle": "Network Specialist", "department": "Networking" },
    { "name": "Sophia Brook", "email": "sophia.brook@cyberspacecorp.onmicrosoft.com", "team": "Fleet Management", "jobTitle": "Fleet Specialist", "department": "Fleet Management" },
    { "name": "Chris Blue", "email": "chris.blue@cyberspacecorp.onmicrosoft.com", "team": "supervisor_it_support", "jobTitle": "Support Specialist", "department": "IT Support" },
    { "name": "Emily Green", "email": "emily.green@cyberspacecorp.onmicrosoft.com", "team": "Shipping", "jobTitle": "Shipping Specialist", "department": "Shipping" },
    { "name": "Nathan Gold", "email": "nathan.gold@cyberspacecorp.onmicrosoft.com", "team": "Sales", "jobTitle": "Salesperson", "department": "Sales" },
    { "name": "Olivia Black", "email": "olivia.black@cyberspacecorp.onmicrosoft.com", "team": "Call Center", "jobTitle": "Representative", "department": "Call Center" },
    { "name": "Liam Red", "email": "liam.red@cyberspacecorp.onmicrosoft.com", "team": "Sales", "jobTitle": "Salesperson", "department": "Sales" },
    { "name": "Amelia White", "email": "amelia.white@cyberspacecorp.onmicrosoft.com", "team": "Call Center", "jobTitle": "Representative", "department": "Call Center" },
    { "name": "Mason Yellow", "email": "mason.yellow@cyberspacecorp.onmicrosoft.com", "team": "Cancellation", "jobTitle": "Cancellation Specialist", "department": "Cancellation" },
    { "name": "Lucas Violet", "email": "lucas.violet@cyberspacecorp.onmicrosoft.com", "team": "recruitment_team", "jobTitle": "HR", "department": "recruitment_team" },
    { "name": "Emma Brown", "email": "emma.brown@cyberspacecorp.onmicrosoft.com", "team": "General HR", "jobTitle": "HR Specialist", "department": "HR" },
    { "name": "Ethan Gray", "email": "ethan.gray@cyberspacecorp.onmicrosoft.com", "team": "Networking", "jobTitle": "Network Specialist", "department": "Networking" },
    { "name": "Ava Pink", "email": "ava.pink@cyberspacecorp.onmicrosoft.com", "team": "Shipping", "jobTitle": "Shipping Specialist", "department": "Shipping" }
  ],
  "groups": {
    "call_center": { "displayName": "Call Center Team" },
    "cancellation": { "displayName": "Cancellation Team" },
    "sales": { "displayName": "Sales Team" },
    "manager_sales": { "displayName": "Manager - Sales" },
    "supervisor_call_center": { "displayName": "Supervisor - Call Center" },
    "it_support": { "displayName": "IT Support Team" },
    "application_support": { "displayName": "Application Support Team" },
    "networking": { "displayName": "Networking Team" },
    "shipping": { "displayName": "Shipping Team" },
    "inventory_management": { "displayName": "Inventory Management Team" },
    "fleet_management": { "displayName": "Fleet Management Team" },
    "chief_technology": { "displayName": "Chief of Technology" },
    "chief_sales": { "displayName": "Chief of Sales" },
    "chief_logistics": { "displayName": "Chief of Logistics" },
    "chief_hr": { "displayName": "Chief of HR" },
    "supervisor_shipping": { "displayName": "Supervisor - Shipping" },
    "supervisor_inventory_management": { "displayName": "Supervisor - Inventory Management" },
    "supervisor_fleet_management": { "displayName": "Supervisor - Fleet Management" },
    "general_hr": { "displayName": "General HR Team" },
    "supervisor_general_hr": { "displayName": "Supervisor - General HR" },
    "supervisor_recruitment": { "displayName": "Supervisor - Recruitment" },
    "recruitment_team": { "displayName": "Recruitment Team" },
    "manager_hr": { "displayName": "Manager - HR" },
    "manager_logistics": { "displayName": "Manager - Logistics" },
    "manager_technology": { "displayName": "Manager - Technology" },
    "supervisor_it_support": { "displayName": "Supervisor - IT Support" },
    "supervisor_application_support": { "displayName": "Supervisor - Application Support" },
    "supervisor_networking": { "displayName": "Supervisor - Networking" },
    "supervisor_cancellation": { "displayName": "Supervisor - Cancellation" },
    "supervisor_sales_team": { "displayName": "Supervisor - Sales Team" }
  },
  "memberships": {
    "call_center": [
      "alice.brown@cyberspacecorp.onmicrosoft.com",
      "bob.green@cyberspacecorp.onmicrosoft.com",
      "charlie.white@cyberspacecorp.onmicrosoft.com",
      "olivia.black@cyberspacecorp.onmicrosoft.com",
      "amelia.white@cyberspacecorp.onmicrosoft.com"
    ],
    "cancellation": [
      "daisy.black@cyberspacecorp.onmicrosoft.com",
      "edward.yellow@cyberspacecorp.onmicrosoft.com",
      "mason.yellow@cyberspacecorp.onmicrosoft.com"
    ],
    "sales": [
      "fiona.red@cyberspacecorp.onmicrosoft.com",
      "george.blue@cyberspacecorp.onmicrosoft.com",
      "hannah.violet@cyberspacecorp.onmicrosoft.com",
      "nathan.gold@cyberspacecorp.onmicrosoft.com",
      "liam.red@cyberspacecorp.onmicrosoft.com"
    ],
    "manager_sales": [
      "kyle.pink@cyberspacecorp.onmicrosoft.com"
    ],
    "supervisor_call_center": [
      "jane.gray@cyberspacecorp.onmicrosoft.com"
    ],
    "it_support": [
      "victor.onyx@cyberspacecorp.onmicrosoft.com",
      "chris.blue@cyberspacecorp.onmicrosoft.com"
    ],
    "application_support": [
      "wendy.quartz@cyberspacecorp.onmicrosoft.com",
      "jordan.lake@cyberspacecorp.onmicrosoft.com"
    ],
    "networking": [
      "xander.marble@cyberspacecorp.onmicrosoft.com",
      "terry.moss@cyberspacecorp.onmicrosoft.com",
      "ethan.gray@cyberspacecorp.onmicrosoft.com"
    ],
    "shipping": [
      "yasmine.coral@cyberspacecorp.onmicrosoft.com",
      "emily.green@cyberspacecorp.onmicrosoft.com",
      "ava.pink@cyberspacecorp.onmicrosoft.com"
    ],
    "inventory_management": [
      "zack.granite@cyberspacecorp.onmicrosoft.com"
    ],
    "fleet_management": [
      "abby.amber@cyberspacecorp.onmicrosoft.com",
      "sophia.brook@cyberspacecorp.onmicrosoft.com"
    ],
    "chief_technology": [
      "rachel.platinum@cyberspacecorp.onmicrosoft.com"
    ],
    "chief_sales": [
      "steve.diamond@cyberspacecorp.onmicrosoft.com"
    ],
    "chief_logistics": [
      "tina.sapphire@cyberspacecorp.onmicrosoft.com"
    ],
    "chief_hr": [
      "ursula.emerald@cyberspacecorp.onmicrosoft.com"
    ],
    "supervisor_shipping": [
      "alice.brown@cyberspacecorp.onmicrosoft.com"
    ],
    "supervisor_inventory_management": [
      "ursula.emerald@cyberspacecorp.onmicrosoft.com"
    ],
    "supervisor_fleet_management": [
      "mike.silver@cyberspacecorp.onmicrosoft.com"
    ],
    "general_hr": [
      "emma.brown@cyberspacecorp.onmicrosoft.com"
    ],
    "supervisor_general_hr": [
      "ursula.emerald@cyberspacecorp.onmicrosoft.com"
    ],
    "supervisor_recruitment": [
      "victor.onyx@cyberspacecorp.onmicrosoft.com"
    ],
    "recruitment_team": [
      "lucas.violet@cyberspacecorp.onmicrosoft.com"
    ],
    "manager_hr": [
      "nina.bronze@cyberspacecorp.onmicrosoft.com"
    ],
    "manager_logistics": [
      "mike.silver@cyberspacecorp.onmicrosoft.com"
    ],
    "manager_technology": [
      "lily.gold@cyberspacecorp.onmicrosoft.com"
    ],
    "supervisor_it_support": [
      "chris.blue@cyberspacecorp.onmicrosoft.com"
    ],
    "supervisor_application_support": [
      "rachel.platinum@cyberspacecorp.onmicrosoft.com"
    ],
    "supervisor_networking": [
      "nina.bronze@cyberspacecorp.onmicrosoft.com"
    ],
    "supervisor_cancellation": [
      "jane.gray@cyberspacecorp.onmicrosoft.com"
    ],
    "supervisor_sales_team": [
      "ian.orange@cyberspacecorp.onmicrosoft.com"
    ]
  },
  "role_assignments": [
    { "role_id": "fdd7a751-b60b-444a-984c-02652fe8fa1c", "group": "chief_hr", "scope": "resource_group" },
    { "role_id": "729827e3-9c14-49f7-bb1b-9608f156bbb8", "group": "it_support", "scope": "resource_group" },
    { "role_id": "d37c8bed-0711-4417-ba38-b4abe66ce4c2", "group": "networking", "scope": "resource_group" },
    { "role_id": "729827e3-9c14-49f7-bb1b-9608f156bbb8", "group": "supervisor_it_support", "scope": "resource_group" },
    { "role_id": "9b895d92-2cd3-44c7-9d02-a6ac2d5ea5c3", "group": "supervisor_application_support", "scope": "resource_group" }
  ]
},
"departments": {
    "Logistics": [
        "Shipping Team",
        "Inventory Management Team",
        "Fleet Management Team",
        "Supervisor - Shipping",
        "Supervisor - Inventory Management",
        "Supervisor - Fleet Management",
        "Manager - Logistics",
        "Chief of Logistics"
    ],
    "Sales": [
        "Sales Team",
        "Manager - Sales",
        "Supervisor - Sales Team",
        "Chief of Sales"
		"Cancellation Team"
		"Call Center Team"
		"Supervisor - Cancellation"
		"Supervisor - Call Center"
		
    ],
    "Technology": [
        "IT Support Team",
        "Networking Team",
        "Application Support Team",
        "Manager - Technology",
        "Chief of Technology",
        "Supervisor - IT Support",
        "Supervisor - Networking",
        "Supervisor - Application Support"
    ],
    "HR": [
        "General HR Team",
        "Recruitment Team",
        "Manager - HR",
        "Supervisor - General HR",
        "Supervisor - Recruitment",
        "Chief of HR"
    ]
}


```


##  Terraform code

This terraform code was intend  to work with the python script. It has function to call the python but this was been remove as I decide not to use AU. 
```
terraform {
  required_providers {
    azuread = {
      source  = "hashicorp/azuread"
      version = "~> 2.0" # Specifies the Azure AD provider and its version
    }
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0" # Specifies the Azure Resource Manager provider and its version
    }
    random = {
      source  = "hashicorp/random"
      version = "~> 3.0" # Specifies the Random provider for generating secrets
    }
  }

  backend "local" {
    path = "terraform.tfstate" # Stores the Terraform state locally
  }

  required_version = ">= 1.3.0" # Ensures Terraform version compatibility
}

# Configures the Azure AD provider, which is used to manage Azure Active Directory resources
provider "azuread" {
  # Configuration options if needed
}

provider "azurerm" {
  features {} # Enables all necessary features for the Azure Resource Manager
  subscription_id = var.subscription_id # Uses the provided subscription ID
}

# Data Source to get current tenant ID
data "azurerm_client_config" "current" {}

# Declares variables for resource configurations
variable "resource_group_name" {
  description = "Name of the Azure Resource Group" # Description of the variable
  type        = string
  default     = "Cyber-Corp-Test-ResourceGroup" # Default value for the resource group name
}

variable "location" {
  description = "Azure region" # Location of the resource group
  type        = string
  default     = "East US" # Default Azure region
}

variable "environment" {
  description = "Deployment environment" # Environment tag for the deployment
  type        = string
  default     = "Test" # Default value indicating this is a test environment
}

variable "project_name" {
  description = "Project name" # Description of the project
  type        = string
  default     = "Cyber Corp Lab" # Default project name
}

variable "config_file" {
  description = "Path to the configuration JSON file" # Path to the configuration file
  type        = string
  default     = "config.json"

  validation {
    condition     = can(jsondecode(file(var.config_file))) # Validates that the config file is valid JSON
    error_message = "Configuration file missing or invalid. Please check config.json." # Error message for invalid config
  }
}

# Reads and processes configuration data from a JSON file
locals {
  config = try(jsondecode(file(var.config_file)), {})

  users           = lookup(local.config, "users", [])
  groups          = lookup(local.config, "groups", {})
  memberships     = flatten([
    for group, members in lookup(local.config, "memberships", {}) : [
      for member in members : {
        group  = group,
        member = member
      }
    ]
  ])
  role_assignments = lookup(local.config, "role_assignments", [])
}

# Creates an Azure Resource Group
resource "azurerm_resource_group" "rg" {
  name     = var.resource_group_name
  location = var.location

  tags = {
    Environment = var.environment
    Project     = var.project_name
  }
}

# Generates a secure client secret for the Azure AD Application
resource "random_password" "client_secret" {
  length  = 32
  special = true
}

# Creates an Azure AD Application for Microsoft Graph API interaction
resource "azuread_application" "graph_client" {
  display_name = "GraphClientApp"

  # Optional: Redirect URIs, etc.
}

# Creates a Service Principal for the Azure AD Application
resource "azuread_service_principal" "graph_client_sp" {
  application_id = azuread_application.graph_client.application_id
}

# Assigns the generated client secret to the Service Principal
resource "azuread_service_principal_password" "graph_client_sp_password" {
  service_principal_id = azuread_service_principal.graph_client_sp.id
  value                = random_password.client_secret.result
  end_date             = "2099-12-31T00:00:00Z"
}

# Assigns Microsoft Graph API permissions to the Azure AD Application
resource "azuread_application_required_resource_access" "graph_permissions" {
  application_object_id = azuread_application.graph_client.object_id

  required_resource_access {
    resource_app_id = "00000003-0000-0000-c000-000000000000" # Microsoft Graph

    resource_access {
      id   = "c19f3c0a-b47e-4b60-8d9b-ff91ce8bfb18" # AdministrativeUnit.ReadWrite.All
      type = "Role"
    }
  }
}

# Existing Resources (Users, Groups, Memberships, etc.)
# ... [Your existing Terraform resources for users, groups, memberships, etc.] ...

# Creates Administrative Units for HR, Sales, and Logistics
resource "azuread_administrative_unit" "au_hr" {
  display_name = "HR Administrative Unit" # Name of the HR Administrative Unit
  description  = "Administrative Unit for Human Resources" # Description of the HR Administrative Unit
  # Note: This AU is created without Restricted Management. The Python script will handle creating AUs with Restricted Management.
}

resource "azuread_administrative_unit" "au_sales" {
  display_name = "Sales Administrative Unit" # Name of the Sales Administrative Unit
  description  = "Administrative Unit for Sales Department" # Description of the Sales Administrative Unit
  # Note: This AU is created without Restricted Management. The Python script will handle creating AUs with Restricted Management.
}

resource "azuread_administrative_unit" "au_logistics" {
  display_name = "Logistics Administrative Unit" # Name of the Logistics Administrative Unit
  description  = "Administrative Unit for Logistics Department" # Description of the Logistics Administrative Unit
  # Note: This AU is created without Restricted Management. The Python script will handle creating AUs with Restricted Management.
}

# Generates secure passwords for users
resource "random_password" "user_password" {
  for_each = { for user in local.users : user.email => user } # Creates a password for each user
  length   = 16 # Specifies the password length
  special  = true # Includes special characters in the password
}

# Creates Azure AD users from the configuration
resource "azuread_user" "users" {
  for_each = { for user in local.users : user.email => user } # Creates a resource for each user

  user_principal_name   = each.value.email # Sets the user's email as the principal name
  display_name          = each.value.name # User's display name
  mail_nickname         = lower(replace(each.value.name, " ", "")) # Generates a mail nickname
  mail                  = each.value.email # User's email address
  password              = random_password.user_password[each.key].result # Assigns the generated password
  force_password_change = true # Requires the user to change their password on first login
  department            = lookup(each.value, "department", null) # Optional department field
  job_title             = lookup(each.value, "jobTitle", null) # Optional job title field
}

# Creates Azure AD groups from the configuration
resource "azuread_group" "groups" {
  for_each = local.groups # Creates a group resource for each group in the config

  display_name       = each.value.displayName # Group's display name
  mail_nickname      = lower(replace(each.value.displayName, " ", "")) # Generates a mail nickname
  security_enabled   = lookup(each.value, "security_enabled", true) # Enables security features
  assignable_to_role = lookup(each.value, "assignable_to_role", true) # Allows role assignments
}

# Maps users to groups based on the configuration
resource "azuread_group_member" "group_members" {
  for_each = { for pair in local.memberships : "${pair.group}-${pair.member}" => pair } # Maps users to groups

  group_object_id  = azuread_group.groups[each.value.group].id # Group's object ID
  member_object_id = azuread_user.users[each.value.member].id # User's object ID
}

# Assigns roles to groups
resource "azuread_directory_role_assignment" "role_assignments" {
  for_each = { for assignment in local.role_assignments : "${assignment.group}-${assignment.role_id}" => assignment } # Assigns roles to groups

  role_id             = each.value.role_id # Role ID being assigned
  principal_object_id = azuread_group.groups[each.value.group].id # Object ID of the group receiving the role
}

# Python Script Invocation to Create Administrative Unit with Restricted Management
resource "null_resource" "create_administrative_unit" {
  depends_on = [
    azuread_service_principal_password.graph_client_sp_password,
    azuread_application_required_resource_access.graph_permissions,
    azuread_administrative_unit.au_hr,
    azuread_administrative_unit.au_sales,
    azuread_administrative_unit.au_logistics
    # Add other dependencies if necessary
  ]

  provisioner "local-exec" {
    command = <<EOT
      source ~/IAM_Labs/Initial_Stage/venv/bin/activate && \
      python3 ~/IAM_Labs/Initial_Stage/create_admin_unit.py \
        --client-id ${azuread_application.graph_client.application_id} \
        --client-secret "${random_password.client_secret.result}" \
        --tenant-id ${data.azurerm_client_config.current.tenant_id} \
        --display-name "Restricted Management AU" \
        --description "Administrative Unit with Restricted Management"
    EOT

    environment = {
      AZURE_CLIENT_ID     = azuread_application.graph_client.application_id
      AZURE_CLIENT_SECRET = random_password.client_secret.result
      AZURE_TENANT_ID     = data.azurerm_client_config.current.tenant_id
    }
  }
}

# Optional: Grant Admin Consent Using Azure CLI
# Uncomment the following block if you have a script to automate admin consent
/*
resource "null_resource" "grant_admin_consent" {
  depends_on = [null_resource.create_administrative_unit]

  provisioner "local-exec" {
    command = "./grant_admin_consent.sh ${azuread_application.graph_client.application_id} ${data.azurerm_client_config.current.tenant_id}"
  }
}
*/

# Outputs the resource group name
output "resource_group_name" {
  description = "The name of the Azure Resource Group." # Outputs the resource group name
  value       = azurerm_resource_group.rg.name # Value of the resource group name
}

# Outputs a list of user principal names
output "user_principal_names" {
  description = "List of user principal names." # Outputs a list of user principal names
  value       = [for user in azuread_user.users : user.user_principal_name] # Generates the list from user resources
}

# Outputs group names mapped to their object IDs
output "group_object_ids" {
  description = "Mapping of group names to their object IDs." # Outputs group names mapped to their IDs
  value       = { for group_name, group in azuread_group.groups : group_name => group.id } # Creates the mapping
}

# Outputs group memberships
output "group_memberships" {
  description = "Mapping of groups to their members." # Outputs group memberships
  value       = {
    for group, members in local.memberships : group => [for member in members : member] # Maps group memberships
  }
}

# Outputs role assignments
output "role_assignments" {
  description = "Mapping of role IDs to principal object IDs." # Outputs role assignments
  value       = {
    for role_id in distinct([for ra in azuread_directory_role_assignment.role_assignments : ra.role_id]) :
    role_id => [
      for ra in azuread_directory_role_assignment.role_assignments :
      ra.principal_object_id if ra.role_id == role_id # Maps roles to object IDs
    ]
  }
}

# Outputs the subscription ID
output "subscription_id" {
  description = "Azure subscription ID." # Outputs the subscription ID
  value       = var.subscription_id # Value of the subscription ID
}

```
