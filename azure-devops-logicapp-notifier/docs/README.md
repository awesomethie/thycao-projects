# Azure DevOps Logic App Email Notifier

This project automates sending email notifications to requestors when a work item reaches certain states in Azure DevOps. It uses Azure Logic Apps and Service Hooks to remove the need for manual updates.

---

## Overview

In many workflows, requestors need to be updated when their request progresses. Doing this manually is slow and easy to forget.

This solution uses a simple event-driven setup: when a work item is updated, Azure DevOps sends a webhook to a Logic App, which checks the state and sends an email if needed.

---

## How It Works

1. A work item is updated in Azure DevOps  
2. A Service Hook sends the update to a Logic App  
3. The Logic App checks the current state  
4. If it matches a configured state, an email is sent to the requestor  

---

## Architecture

`Azure DevOps → Service Hook → Logic App → Email Notification`

---

## Features

- Automatically sends emails based on work item state  
- Uses webhook (event-driven) instead of polling  
- Pulls requestor email dynamically from the work item  
- Simple and easy to extend  

---

## Pros

- Very low cost (nearly 0 if traffic is low)  
- No infrastructure to manage (serverless)  
- Quick to build and deploy  
- Scales automatically with usage  

---

## Limitations

- Only checks the current state (does not track transitions)  
- Can send duplicate emails if the same state is updated multiple times  
- Each new state needs to be added manually in the Logic App  

---

## Tech Stack

- Azure Logic Apps  
- Azure DevOps Service Hooks  
- Office 365 Outlook (shared mailbox)  

---

## Setup Guide

Step-by-step setup is documented here:

[docs/setup-guide.md](docs/setup-guide.md)

---

## Notes

This is a simple automation example built as part of a collection of work-related projects. The focus is on keeping the solution lightweight and practical rather than over-engineered.
