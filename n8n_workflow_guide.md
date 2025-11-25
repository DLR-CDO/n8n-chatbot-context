# Creating Your First Workflow

## 2.1 Creating and Running a Workflow
1. From the **Dashboard**, click **New Workflow**.  
   ![WorkflowCreation](https://cdostorageacct.blob.core.windows.net/n8n-chatbot/n8n_images/CreateWorkflow.png)
2. Give your workflow a descriptive name (e.g., `project--Demo`). See Naming Conventions and Tags below for inspiration.
   ![NamingWorkflow](https://cdostorageacct.blob.core.windows.net/n8n-chatbot/n8n_images/NameWorkflow.png)
3. Drag and drop nodes onto the canvas to build the automation sequence.  
4. Use the **Execute Workflow** button to run it end to end.  
5. Review the **Execution Log** for detailed input/output data and debugging.  
    ### Naming Conventions and Tags
    Use clear naming patterns to make workflows easy to track and maintain:
    - **Format:** `Sourcesystem-project-name` (e.g., `SFDC-LeadRouting-FileSync`)  
    - **Tags:** Use project tags for grouping (`HR`, `Finance`, etc.)  
    - **Variable naming:** Prefix environment or purpose (e.g., `$env.apiKey`, `$json.user_id`)  

These standards simplify collaboration and prevent confusion when multiple users edit workflows.

## 2.2 Example Workflow: Build a Simple AI Chatbot with Memory
This example shows how to create an AI-powered chatbot in n8n. The chatbot uses Google Gemini to generate responses and Simple Memory so it remembers the conversation context.

## Quick architecture (what you’ll create)

When chat message received (Trigger) → Agent (wired to AzureOpenAI model \+ Simple Memory) → reply to chat
   ![ExampleWorkflow](https://cdostorageacct.blob.core.windows.net/n8n-chatbot/n8n_images/ExampleWorkflow.png)
### 1. Create a new workflow

1.	In n8n, click New Workflow.
2.	Name it: AI Chatbot with Memory.
3.	Save the workflow (click Save early and often).

### 2. Add the trigger node

1. Add When chat message received (or equivalent chat trigger in your n8n edition).
2. Optional: set a welcome hint or placeholder text (e.g., *Start by saying "hi"*).
3. Save node.

### 3. Create your first credential

1.	Go to Credentials → New → choose Azure OpenAI (or the LLM provider of your choice).
2.	Paste your Endpoint, API key and name the credential.
      ![CredentialCreation](https://cdostorageacct.blob.core.windows.net/n8n-chatbot/n8n_images/CredentialCreation.png)

### 4. Add the AI Agent node

1. Click on “\+” icon after chat trigger and add AI Agent node to the canvas.
2. Configure:
	a.	Chat Model: Choose the credential created in step3 and model (choose based on availability/cost).
	b. Leave simple memory as it is for now.
3.	Double click on Agent and select “Conversation Agent” connected to “Chat trigger Node”
   ![SelectConversationalAgent](https://cdostorageacct.blob.core.windows.net/n8n-chatbot/n8n_images/SelectConversationalAgent.png)

### 5. Execute the workflow

Prompt the agent to trigger the workflow execution
   ![ExecuteWorkflow](https://cdostorageacct.blob.core.windows.net/n8n-chatbot/n8n_images/ExecuteWorkflow.png)

### 6. How to Make the Chatbot Publicly Accessible (Optional) 
1. Open the trigger node in your workflow, and edit the node settings 

2. Enable "Make Chat Publicly Available"  

3. Switch to "Hosted Chat" mode. 
   ![Make Chat Publicly Available](https://cdostorageacct.blob.core.windows.net/n8n-chatbot/n8n_images/PublicChat.png)

4. You can choose to allow anyone to access the chat (Setting Authentication to "None") or restrict access using authentication:
    
   **Basic Auth:**
   1. Create a new credential for Basic Auth with a username and password. 
   2. All users must use the same username and password to access the chat. 
      ![BasicAuth](https://cdostorageacct.blob.core.windows.net/n8n-chatbot/n8n_images/BasicAuth.png)
   3. Copy the URL displayed in the Trigger Node settings and share it with the users. 
   4. On pasting the URL in a browser, you get prompted to enter the credentials to login and access the chat interface. 
      ![BasicAuthExample](https://cdostorageacct.blob.core.windows.net/n8n-chatbot/n8n_images/BasicAuthExample.png)

   **n8n User Auth:**
   5. Only users logged in to an n8n account can access the chat. 
   6. Users must be authenticated in n8n before using the public URL 
      ![n8nUserAuth](https://cdostorageacct.blob.core.windows.net/n8n-chatbot/n8n_images/n8nUserAuth.png)
5. Save the Workflow.
