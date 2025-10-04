Front Loan Routing System
Automated loan routing and task management system for Front using Airtable integration and sidebar plugins.
Overview
This system automatically:

Extracts loan numbers from email subjects (10-digit numbers starting with 152)
Queries Airtable to retrieve loan-specific URLs
Displays task forms and dashboards directly in Front's sidebar
Enables team-based routing using loan metadata

Architecture

Application Object: Extracts loan numbers and queries Airtable
Sidebar Plugins: Display task forms and dashboards in iframes
Airtable Database: Stores loan numbers with associated URLs and team assignments

Prerequisites

Front account with company admin access
Airtable account with Personal Access Token
GitHub account (for hosting plugins)
Loan database in Airtable with:

Field: "Loan Number" (text)
Field: "Tasks Form" (URL)
Field: "Dashboard View" (URL)
Field: "Team" (text, optional for routing)



Airtable Setup
1. Create Personal Access Token

Go to https://airtable.com/create/tokens
Create new token with:

Scope: data.records:read
Base access: Your funding database


Copy token (starts with pat...)

2. Get Your Base and Table IDs

Base ID: Found in Airtable API docs (starts with app...)
Table ID: Found in table URL or API docs (starts with tbl...)

Front Configuration
App 1: Loan Task Management
Create the App

Settings → Company → Developers → Create app
Name: "Loan Task Management"

Configure Server

Add feature → Servers
Origin: https://api.airtable.com
Authentication: API Key
Property name: Authorization
Local credentials: Bearer YOUR_AIRTABLE_TOKEN

Configure Application Object

Add feature → Connector → Application object
Collect reference:

Text pattern: 152 + 7 digits
Object type: "Get Loan Number"


Send request:

Method: GET
URL: Select Airtable server
Path: /v0/YOUR_BASE_ID/YOUR_TABLE_ID?filterByFormula={Loan Number}='{Text referenced from previous step}'


Return data:

Extract Tasks Form field → Set as Target URL
Extract Dashboard View field → Add as output
Extract Team field (optional)



Add Tasks Sidebar Plugin

Add feature → Plugin → Sidebar plugin
Name: "Loan Tasks"
Plugin URL: https://yourusername.github.io/front-loan-plugin/
Context: Single conversation

App 2: Loan Dashboard Viewer

Create app → "Loan Dashboard Viewer"
Add feature → Plugin → Sidebar plugin
Name: "Loan Dashboard"
Plugin URL: https://yourusername.github.io/front-dashboard-plugin/
Context: Single conversation

Plugin Code
Tasks Plugin (index.html)
html<!DOCTYPE html>
<html>
<head>
    <script type="text/javascript" src="https://dl.frontapp.com/libs/plugin-sdk-1.8.1.min.js"></script>
    <style>
        body { margin: 0; padding: 0; overflow: hidden; }
        iframe { border: none; width: 100%; height: 100vh; }
        #content { height: 100vh; display: flex; align-items: center; justify-content: center; }
        .message { padding: 20px; text-align: center; color: #666; }
    </style>
</head>
<body>
    <div id="content"><div class="message">Loading...</div></div>
    
    <script>
        Front.contextUpdates.subscribe(context => {
            if (context.type === 'singleConversation') {
                const conversation = context.conversation;
                const tasksLink = conversation.links?.find(link => 
                    link.name === 'Tasks Form'
                );
                
                if (tasksLink && tasksLink.externalUrl) {
                    document.getElementById('content').innerHTML = 
                        `<iframe src="${tasksLink.externalUrl}"></iframe>`;
                } else {
                    document.getElementById('content').innerHTML = 
                        `<div class="message"><p>No task form found for this loan</p></div>`;
                }
            } else {
                document.getElementById('content').innerHTML = 
                    `<div class="message"><p>Select a conversation to view the task form</p></div>`;
            }
        });
    </script>
</body>
</html>
Dashboard Plugin (index.html)
html<!DOCTYPE html>
<html>
<head>
    <script type="text/javascript" src="https://dl.frontapp.com/libs/plugin-sdk-1.8.1.min.js"></script>
    <style>
        body { margin: 0; padding: 0; overflow: hidden; }
        iframe { border: none; width: 100%; height: 100vh; }
        #content { height: 100vh; display: flex; align-items: center; justify-content: center; }
        .message { padding: 20px; text-align: center; color: #666; }
    </style>
</head>
<body>
    <div id="content"><div class="message">Loading...</div></div>
    
    <script>
        Front.contextUpdates.subscribe(context => {
            if (context.type === 'singleConversation') {
                const conversation = context.conversation;
                const dashboardLink = conversation.links?.find(link => 
                    link.name === 'Dashboard View'
                );
                
                if (dashboardLink && dashboardLink.externalUrl) {
                    document.getElementById('content').innerHTML = 
                        `<iframe src="${dashboardLink.externalUrl}"></iframe>`;
                } else {
                    document.getElementById('content').innerHTML = 
                        `<div class="message"><p>No dashboard available</p></div>`;
                }
            } else {
                document.getElementById('content').innerHTML = 
                    `<div class="message"><p>Select a conversation</p></div>`;
            }
        });
    </script>
</body>
</html>
GitHub Pages Deployment
For Each Plugin:

Create repository on GitHub (public)
Add index.html with plugin code
Settings → Pages → Deploy from main branch
Copy GitHub Pages URL: https://yourusername.github.io/repo-name/
Use this URL in Front plugin configuration

Testing

Send test email with subject containing loan number (e.g., "Loan 1525269719")
Open conversation in Front
Check for attached link objects in conversation
Click sidebar plugins to view forms

Troubleshooting
Application Object Not Attaching

Verify app is published/enabled
Check loan number is in email subject (not body)
Confirm loan number exists in Airtable
Test the Airtable query in the app configuration

Plugin Shows "Loading..." Forever

Check browser console (F12) for errors
Verify GitHub Pages URL is correct (.github.io, not .github.com)
Ensure repository is public
Confirm plugin context is set to "Single conversation"

Iframe Refuses to Connect

Using GitHub Pages URL (not repository URL)
Check if target site allows iframe embedding
Review browser console for CSP errors

401 Unauthorized from Airtable

Verify Airtable token includes "Bearer " prefix
Check token has correct scopes and base access
Confirm Base ID and Table ID are correct

Team Routing (Optional)
To route based on team assignment:

In Application Object, extract "Team" field as dynamic variable
Create Front rules:

When: Application object attached
If: Dynamic variable team equals "Team One"
Then: Route to Team One inbox



Regex Pattern
The loan number extraction uses:
152\d{7}
This matches 10-digit numbers starting with 152 (e.g., 1525269719).
License
MIT
