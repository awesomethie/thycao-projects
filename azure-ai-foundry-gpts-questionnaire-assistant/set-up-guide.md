# Setup Guide

When working with security questionnaires, one recurring problem was preparing the knowledge needed to answer them accurately.

Most questionnaires were provided as Excel files. The custom GPT could work with these files, but the supporting knowledge came from around 30 internal documents. Since the GPT could only accept a limited number of knowledge files, uploading every source document separately was not practical.

Before this setup was in place, the documents had to be reviewed and combined manually. That worked, but it took a lot of time and made it easy to miss important information or duplicate the same content.

To make this process easier, I decided to use Claude Code to scan the documents, extract the relevant information and convert it into a smaller set of structured Markdown files.

There was no separate company Claude account available, but Claude models were available through Microsoft Foundry. I therefore deployed a Claude model in Microsoft Foundry and connected it to Claude Code in Visual Studio Code.

The idea is straightforward:

* deploy a Claude model in Microsoft Foundry
* install Claude Code in Visual Studio Code
* connect Claude Code to the Foundry resource
* authenticate using the Foundry API key
* open a local folder containing the approved documents
* ask Claude Code to review the files and build the knowledge base

This guide explains how the setup was done.

## Create a Microsoft Foundry resource

The first thing needed was a Microsoft Foundry resource.

To create one:

* go to `https://ai.azure.com/`
* sign in using your Microsoft work account
* create a new Microsoft Foundry resource
* create a project under the resource

You need the correct Azure permissions to create the resource.

If the option is not available, ask your Azure administrator to give you access or create the resource for you.

After the resource has been created, make a note of the resource name.

The resource name will later be used in the Claude Code configuration.

For example:

```text
<foundry-resource-name>
```

Do not use a real resource name in public documentation if it contains a personal name, company name or internal project name.

## Deploy the Claude model

After the Foundry resource has been created, the next step is to deploy the Claude model.

To do that:

* open the Microsoft Foundry portal
* open your project
* go to the model catalogue
* search for the Claude model you want to use
* select the model
* select **Deploy**
* choose the project where the model should be deployed
* enter a deployment name
* complete the deployment

After the deployment has finished, make a note of the deployment name.

For example:

```text
<claude-deployment-name>
```

The deployment name is important because Claude Code needs to use the exact same value.

The deployment name does not always have to be the same as the official model name.

For example, the model may be Claude Opus, but the deployment can have a custom name.

## Copy the Foundry API key

Claude Code needs permission to connect to the model.

For this setup, I used the API key from the Microsoft Foundry project.

To find the key:

* open the Foundry project
* go to the project overview page
* find the project API key
* copy the key
* store it somewhere secure

For public examples, use a placeholder:

```text
<foundry-api-key>
```

Never include the real key in:

* GitHub
* screenshots
* documentation
* source code
* chat messages
* issue descriptions
* pull requests

If an API key is accidentally exposed, it should be replaced.

## Install Visual Studio Code

The next step is to install Visual Studio Code.

If it is not already installed:

* go to `https://code.visualstudio.com/`
* download the Windows installer
* complete the installation
* open Visual Studio Code

Visual Studio Code will be used to open the local document folder and interact with Claude Code.

## Install Claude Code

Claude Code can be installed as an extension in Visual Studio Code.

To install it:

* open Visual Studio Code
* select **Extensions** from the left-hand menu
* or press `Ctrl+Shift+X`
* search for `Claude Code`
* select the official extension published by Anthropic
* select **Install**

After the extension has been installed, a Claude Code icon should appear in the Visual Studio Code sidebar.

If the icon does not appear:

* press `Ctrl+Shift+P`
* search for `Developer: Reload Window`
* select the command

## Create the Claude Code settings file

Claude Code now needs to be told to use Microsoft Foundry instead of connecting directly to Anthropic.

On Windows, Claude Code uses the following settings file:

```text
C:\Users\<windows-username>\.claude\settings.json
```

Replace `<windows-username>` with your Windows username.

You can also open the same location using:

```text
%USERPROFILE%\.claude
```

If the `.claude` folder does not exist:

* open File Explorer
* go to your Windows user folder
* create a folder called `.claude`
* open the folder
* create a file called `settings.json`

The final file should be:

```text
%USERPROFILE%\.claude\settings.json
```

## Add the Microsoft Foundry configuration

Open `settings.json` and add the following configuration:

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "env": {
    "CLAUDE_CODE_USE_FOUNDRY": "1",
    "ANTHROPIC_FOUNDRY_RESOURCE": "<foundry-resource-name>",
    "ANTHROPIC_FOUNDRY_API_KEY": "<foundry-api-key>",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "<claude-deployment-name>"
  }
}
```

Replace the placeholders with the correct values from Microsoft Foundry.

```text
<foundry-resource-name>
```

Replace this with the name of the Microsoft Foundry resource.

```text
<foundry-api-key>
```

Replace this with the API key copied from the Foundry project.

```text
<claude-deployment-name>
```

Replace this with the exact deployment name of the Claude model.

## Understand the settings

The configuration contains four main settings.

### Use Microsoft Foundry

```json
"CLAUDE_CODE_USE_FOUNDRY": "1"
```

This tells Claude Code to use Microsoft Foundry.

Without this setting, Claude Code may try to connect directly to Anthropic.

### Select the Foundry resource

```json
"ANTHROPIC_FOUNDRY_RESOURCE": "<foundry-resource-name>"
```

This tells Claude Code which Foundry resource should be used.

The value must match the resource created earlier.

### Provide the API key

```json
"ANTHROPIC_FOUNDRY_API_KEY": "<foundry-api-key>"
```

This provides the API key that Claude Code uses to authenticate.

The real value should never be added to a public GitHub repository.

### Select the model deployment

```json
"ANTHROPIC_DEFAULT_OPUS_MODEL": "<claude-deployment-name>"
```

This tells Claude Code which Claude deployment should be used.

The value must match the deployment name in Microsoft Found
