Chapter 1
Create Your First Agent
In this chapter, we'll walk through the process of creating your very first AI agent using the Foundry Agent Service.
By the end, you'll have a simple agent running locally that you can interact with in real time.

Open your Dev Enviroment
Open VSCode, by default VSCode should open the directory c:\agent, if that is not the case follow this:

Open your project folder

Open VSCode
Click "File"
Click "Open Folder"
Select "This Computer\Local Disk (C:)\Agent"
Click on "Select Folder"
Click "Yes I trust the authors"
Login to Azure
Before you can use the Azure Foundry Agent Service, you need to sign in to your Azure subscription.

If you don't see the terminal opened at the bottom of VSCode, follow this:

In the top menu click "Terminal"
Click "New Terminal"
Run the following command and follow the on-screen instructions. Use credentials that have access to your Microsoft Foundry resource:

shell
TypeCopy
az login --use-device-code
Use the credentials below
If you need to reset your Temporary Access Password (TAP), click this button:

TAP: <tap>

Note: You will only be able to use this TAP after generating, the other will no longer work.

Username:
User1-56904705@LODSPRODMSLEARNMCA.onmicrosoft.com
Access Token:
9ps7fxx^

NOTE: Once logged in, return to the terminal and see 1 Azure Subscription listed. Press Enter on your keyboard to select the default

Create a .env File
We'll store secrets (such as your project connection string) in an environment file for security and flexibility.

Create a file named .env in the root of your project directory.

Add the following line to the file:

env
TypeCopy
PROJECT_CONNECTION_STRING="https://<your-foundry-resource>.services.ai.azure.com/projects/<your-project-name>"
Replace https://<your-foundry-resource>.services.ai.azure.com/api/projects/<your-project-name> with the actual values from your Microsoft Foundry project.



Where to find your connection string:

Open a new tab in Edge Browser.
Visit https://ai.azure.com, signing in with the given Azure credentials.
Navigate to your project
Click on Overview
The connection string will be displayed on the homepage of your project, under the tab that reads Microsoft Foundry project endpoint.
Make sure there are no spaces around the "=" sign in the ".env" file.

Create a Basic Agent
We'll now create a basic Python script that defines and runs an agent.

Start by opening the file called: agent.py in the workshop folder
Add Imports to agent.py
These imports bring in the Azure SDK, environment handling, and helper classes:

python
TypeCopy
import os
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential
from azure.ai.agents.models import MessageRole, FilePurpose, FunctionTool, FileSearchTool, ToolSet
from dotenv import load_dotenv
Load the .env File
Load environment variables into your script by adding this line to agent.py:

python
TypeCopy
load_dotenv(override=True)
Create an AIProjectClient Instance
This client connects your script to the Microsoft Foundry service using the connection string and your Azure credentials.

python
TypeCopy
project_client = AIProjectClient(
    endpoint=os.environ["PROJECT_CONNECTION_STRING"],
    credential=DefaultAzureCredential()
)
Create the Agent
Now, let's create the agent itself. In this case, it will use the GPT-4o model.

To look-up the name of your deployed model go to: ai.azure.com and Model + endpoints. Here you find the name of your deployed model.

python
TypeCopy
agent = project_client.agents.create_agent(
    model="gpt-4o",
    name="my-agent"
)
print(f"Created agent, ID: {agent.id}")
Create a Thread
Agents interact within threads. A thread is like a conversation container that stores all messages exchanged between the user and the agent.

python
TypeCopy
thread = project_client.agents.threads.create()
print(f"Created thread, ID: {thread.id}")
Add a Message
This loop lets your user send messages to the agent. A textual input is required from the user via the terminal, which is then added to the thread as a new incoming message.

python
TypeCopy
try:
    while True:

        # Get the user input
        user_input = input("You: ")

        # Break out of the loop
        if user_input.lower() in ["exit", "quit"]:
            break

        # Add a message to the thread
        message = project_client.agents.messages.create(
            thread_id=thread.id,
            role=MessageRole.USER, 
            content=user_input
    )
Create and Process an Agent Run
The agent processes the conversation thread and generates a response.

python
TypeCopy
        run = project_client.agents.runs.create_and_process(
            thread_id=thread.id, 
            agent_id=agent.id
        )
Fetch All Messages from the Thread
This retrieves all messages from the thread and prints the agent's most recent response.

python
TypeCopy
        messages = project_client.agents.messages.list(thread_id=thread.id)
        first_message = next(iter(messages), None) 
        if first_message: 
            print(next((item["text"]["value"] for item in first_message.content if item.get("type") == "text"), "")) 
Delete the Agent When Done
Once you're finished, clean up by deleting the agent:

python
TypeCopy
finally:
    # Clean up the agent when done
    project_client.agents.delete_agent(agent.id)
    print("Deleted agent")
Add this code to delete the agent outside of the while True-loop. Otherwise the agent will be deleted immediately after your first interaction.

Run the Agent
Save your file before running!

Finally, run the Python script:

shell
TypeCopy
python agent.py
You can now chat with your agent directly in the terminal. You can try some of these prompts for testing purposes:

TypeCopy
 What is the tallest mountain in the world?
TypeCopy
What do you recommend to do in San Francisco during my visit?
Type exit or quit to stop the conversation.

Recap
In this chapter, you have:

Logged in to Azure
Retrieved a connection string
Separated secrets from code using ".env"
Created a basic agent with the Foundry Agent Service
Started a conversation with a GPT-4o model
Cleaned up by deleting the agent when done
