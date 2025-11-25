# How to Create a Credential in n8n

## Step 1: Open n8n
Log in to your n8n instance (cloud or self-hosted).

## Step 2: Go to the Credentials Page
Look at the top right corner for ‘Create Workflow’ button and click on the drop down.
Click on Credentials.

## Step 3: Add a New Credential
Click the button that says Add New Credential.

## Step 4: Select the Type of Credential
A search box and list will appear.

Type the name of the application or service you want to connect to, for example:

- HTTP Request
- PostgreSQL
- Microsoft Graph
- Google Drive
- Slack, etc.

Click on the credential type you want.

## Step 5: Fill in the Required Details
You will now see a form depending on the type of service.

Fill the fields such as:

- Name of credential (example: Prod-API, MyDB-Test, Slack-Bot)
- Authentication method (API key, OAuth2, Username+Password, etc.)
- Base URL, Host, Database name (if needed)
- API key, token, client secret, password, etc.

## Step 6: (Optional) App registration for Client ID & Secret OAuth2
You will need to request for an app registration to use OAuth2 authentication method.

Send an email to service desk at servicedesk@digitalrealty.com or create a standard request using this link - https://interxion.service-now.com/esc?id=sc_cat_item&sys_id=da060612db7bf700fd586c16ca961900

Assign the ticket to L3 Systems Administration and provide them below information:

- App Name- (Provide an application name relevant to your project or workflow)
- Purpose- (Justification for use of app registration)
- OAuth Redirect URL- (Ask them to add this url in app registration)
- Any specific API permissions mentioned in the Credential you are creating. (Usually at the bottom of the credentials window)

They will provide you with below details, you can setup credentials using them.

- Tenant ID
- Client ID
- Client Secret.

## Step 7: (Optional) Test the Credential
Most credentials have a Test button.

Click Test.

If the test succeeds, it means your credential works.

If it fails, check:

- Key or token is correct
- Base URL is correct
- System firewall/VPN/whitelist is not blocking
- For OAuth, the redirect URL matches the one shown in n8n

## Step 8: Save the Credential
Click Save.

The credential is now stored securely in n8n.


# Governance for Connectors & Integrations in n8n

We already have commonly used connectors available for **Salesforce**, **Microsoft Outlook**, **SharePoint**, **Teams**, and **ServiceNow**.

If a new connector credential is required outside these, a request must be raised with the respective **source system team**, and the **Global Admin** (e.g., John Cale) will provide the necessary permissions in **App Registration**.
   

## Refering secrets from Azure Key Vault:
Here’s the right way to reference secrets from azure key vault inside node expressions. 
users can select **Expression** and write an expression **{{$secrets.azureKeyVault[‘secret-name’]}}** to use secret value from azure key vault.

Replace your secret name in the expression in place of ‘secret-name’ - See the screenshot below referencing **CLIENT-ID,CLIENT-SECRET & TENANT-ID**.
   ![ReferencingAzureKeyVaultSecrets](https://cdostorageacct.blob.core.windows.net/n8n-chatbot/n8n_images/ReferencingAzureKeyVaultSecrets.png)

We’re using **Azure Key Vault** for credential storage. Azure Key Vault can only be used to parameterize credentials but not sensitive information you ingest in workflow nodes.
For example:
- **Api key or URL** you enter in the HTTP nodes.
- **Teams ID or Channel ID** you enter in Teams node.
This is a limitation in n8n, you can read more here - [https://docs.n8n.io/external-secrets/](https://docs.n8n.io/external-secrets/)

Please make sure not to include any **PII** or sensitive secrets unless there is no risk involved.

- Users can reach out to [CdoPlatformSupport@digitalrealty.com](mailto:CdoPlatformSupport@digitalrealty.com) for assistance in adding any new credentials to the Key Vault.