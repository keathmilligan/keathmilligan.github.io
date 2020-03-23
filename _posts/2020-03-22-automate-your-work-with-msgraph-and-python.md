---
layout: post
title: Automate your work with the Microsoft Graph API and Python
date: 2020-03-22 10:26:55 -0500
categories: [dev]
tags: [python, rest, microsoft, graph, msal, teams, sharepoint]
featured-img: msgraph.png
---

![Microsoft Graph](/assets/images/msgraph.png)

Automate tasks with Microsoft online apps from the command-line with Python.
<!--more-->

The [Microsoft Graph API](https://docs.microsoft.com/en-us/graph/overview) gives you access to a wide variety of functionality in Office 365 - create and manipulate Office documents, access files in OneDrive and Sharepoint, interact with Teams spaces and more. Using Python, you can create command-line apps that securely interact with Graph to automate every day work tasks.

## Command-line vs. web/interactive apps

Many of the examples you may find are geared toward interactive web or desktop applications that require user input. In this case however, we will build a command-line app that can run without user input and potentially be used as part of an unattended automated process driven by a cron job or something similar.

The key to this is to leverage "device flow" authentication that allows us to authenticate the app on behalf of a user rather then using an API key or having to store and supply a username/password with each request. This can usually be accomplished by regular users without admin consent.

## Register your app

> If you don't have Office 365 available or you don't want to use your work/production instance, you can create free [developer sandbox](https://developer.microsoft.com/en-us/microsoft-365/dev-program) with full functionality for a limited time.

The first step is to register your app in the [Azure Active Directory Portal](https://aad.portal.azure.com/). 

1. Navigate to the portal and select **Azure Active Directory** > **App registrations**. Click the **Register an application** button.
2. Give the app a descriptive name. Users will see this name when they authenticate.
3. For most apps, just leave the "Supported account types" set to the default ("Accounts in this organization only").
4. In the "Redirect URI" section, select "Public client/native (mobile & desktop)" and enter `https://login.microsoftonline.com/common/oauth2/nativeclient` in the redirect URI field.
5. Click the **Register** button.
6. _Important:_ click the link next to "Redirect URIs" - it should say something like "0 web, 1 public client". Then under "Advanced Settings", click **Yes** next to "Treat as a public client".
7. Click the **Save** button near the top.

Refer to the [Microsoft docs](https://docs.microsoft.com/en-us/graph/auth-register-app-v2) for more info on app registration.

## Add permissions to your app

In order for your app to access Office 365 content and functionality, you need to grant it permission to specific resources you want to use.

1. Select **API permissions** in the portal to view/add permissions.
2. Click the **Add a permission** button and then select "Microsoft Graph".
3. Select "Delegated prmissions".

For the purposes of this example, locate and add the following permissions:

* Files.ReadWrite.All
* Sites.ReadWrite.All
* User.Read
* User.ReadBasic.All

Click the **Add Permissions** button to apply your changes. Note that you can add more permissions later, but users will need to re-authenticate the application.

Leave the portal tab open, you'll need to copy some info from it later.

## Python setup

Create a Python virtual environment (recommended) and install the following packages with `pip`:

* **requests**
* **msal**

You're probably already familiar with **requests** for working with web services. **msal** ([Microsoft authentication library](https://github.com/AzureAD/microsoft-authentication-library-for-python)) is a new Python library from Microsoft that makes authenticating with Azure Active Directory easy.

Import them in your script:

```python
import requests
import msal
```

### Key site information required

Back in the Azure AD portal, select the app you just registered (**Azure Active Directory** > **App registrations** > _your app_). Copy the following values:

* Application (client) ID
* Directory (tenant) ID

Paste these into your Python script:

```python
TENANT_ID = '<your tenant id>'
CLIENT_ID = '<your application id>'
```

You will also need your SharePoint host name. This is typically something like "yourcompany.sharepoint.com":

```python
SHAREPOINT_HOSTNAME = 'yourcompany.sharepoint.com'
```

## Minimal example

Now we have enough to put together a minimal app:

```python
import requests
import msal

TENANT_ID = '<your tenant id>'
CLIENT_ID = '<you application id>'
SHAREPOINT_HOST_NAME = 'yourcompany.sharepoint.com'

AUTHORITY = 'https://login.microsoftonline.com/' + TENANT_ID
ENDPOINT = 'https://graph.microsoft.com/v1.0'

SCOPES = [
    'Files.ReadWrite.All',
    'Sites.ReadWrite.All',
    'User.Read',
    'User.ReadBasic.All'
]

app = msal.PublicClientApplication(CLIENT_ID, authority=AUTHORITY)

flow = app.initiate_device_flow(scopes=SCOPES)
if 'user_code' not in flow:
    raise Exception('Failed to create device flow')

print(flow['message'])

result = app.acquire_token_by_device_flow(flow)

if 'access_token' in result:
    result = requests.get(f'{ENDPOINT}/me', headers={'Authorization': 'Bearer ' + result['access_token']})
    result.raise_for_status()
    print(result.json())

else:
    raise Exception('no access token in result')
```

When you run the app, you will see something like:

```
To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code JE64IUO6K to authenticate.
```

Open a browser to the URL, sign-in and enter the code. You will be asked to select an account and then to approve the application's access to the requesrted permissions. The app will block waiting for you to complete the process. When you approve the app, some basic information about your account will be displayed.

## Token caching

Each time you run the minimal example above, you will have to click on the link and log in with your web browser. We can avoid having to do this every time by adding a serializable token cache to the MSAL app when it is created:

```python
cache = msal.SerializableTokenCache()

app = msal.PublicClientApplication(CLIENT_ID, authority=AUTHORITY, token_cache=cache)
```

### Serialization

In order for the cache to useful in a command-line app, we need to serialize it to a file or some other form of storage so it can be accessed the next time you run the command. For example, to serialize to a file:

```python
atexit.register(lambda: open('token_cache.bin', 'w').write(cache.serialize()) if cache.has_state_changed else None)
```

This writes the cache out to the file when your script terminates.

And then to load the cache:

```python
if os.path.exists('token_cache.bin'):
    cache.deserialize(open('token_cache.bin', 'r').read())
```

> **The serialized token cache is sensitive.** It contains a refresh token that can be used it to access the resources the app has been granted permissions to until it expires. This simple example just writes the cache to a file in the current directory - a real app will need to place it in a secure location or use a more secure storage method.
{: .danger}

Refer to the [MSAL docs](https://msal-python.readthedocs.io/en/latest/#msal.SerializableTokenCache) for more info on the SerializableTokenCache class.

Here is the complete example again with a serialized token cache:

```python
import requests
import msal
import atexit
import os.path

TENANT_ID = '<your tenant id>'
CLIENT_ID = '<your application id>'
SHAREPOINT_HOST_NAME = 'yourcompany.sharepoint.com'

AUTHORITY = 'https://login.microsoftonline.com/' + TENANT_ID
ENDPOINT = 'https://graph.microsoft.com/v1.0'

SCOPES = [
    'Files.ReadWrite.All',
    'Sites.ReadWrite.All',
    'User.Read',
    'User.ReadBasic.All'
]

cache = msal.SerializableTokenCache()

if os.path.exists('token_cache.bin'):
    cache.deserialize(open('token_cache.bin', 'r').read())

atexit.register(lambda: open('token_cache.bin', 'w').write(cache.serialize()) if cache.has_state_changed else None)

app = msal.PublicClientApplication(CLIENT_ID, authority=AUTHORITY, token_cache=cache)

accounts = app.get_accounts()
result = None
if len(accounts) > 0:
    result = app.acquire_token_silent(SCOPES, account=accounts[0])

if result is None:
    flow = app.initiate_device_flow(scopes=SCOPES)
    if 'user_code' not in flow:
        raise Exception('Failed to create device flow')

    print(flow['message'])

    result = app.acquire_token_by_device_flow(flow)

if 'access_token' in result:
    result = requests.get(f'{ENDPOINT}/me', headers={'Authorization': 'Bearer ' + result['access_token']})
    result.raise_for_status()
    print(result.json())

else:
    raise Exception('no access token in result')
```

The first time you run the script, you will be prompted to authenticate with a code as before, but when you run the script again, it will acquire the access token from the cache. The script will continue to work without prompting you to authenticate until the refresh token expires (default is 90 days but this can be changed by your administrator) or is revoked.

## Working with Teams and SharePoint sites

OK, now we can do something useful like upload/download files from a Teams or SharePoint site.

Every team in Microsoft Teams has a corresponding SharePoint site that hosts its files and other content. You can find this site by clicking the **Open in SharePoint** button in the "Files" tab of the team. Additionally, private channels within a team will also get their own separate SharePoint site that is restricted to members of the channel.

Click the **Open in SharePoint** button and note the name of the site in the URL. It is usually the name of the team with spaces and special characters removed. E.g. "Test Team" becomes "TestTeam".

### Use graph to get the site id

With the name of the site, we can query Graph for more information about it. In particular, we will need the site ID:

```python
result = requests.get(f'{ENDPOINT}/sites/{SHAREPOINT_HOST_NAME}:/sites/{SITE_NAME}', headers={'Authorization': 'Bearer ' + access_token})
site_info = result.json()
site_id = site_info['id]
```

## Working with drives and files

Now that we have the site ID, we can get information about its "drive" which contains all of its files and folders:

```python
result = requests.get(f'{ENDPOINT}/sites/{site_id}/drive', headers={'Authorization': 'Bearer ' + access_token})
drive_info = result.json()
```

> You can also use Graph to access content in OneDrive drives as well. Refer to the [MS Graph docs](https://docs.microsoft.com/en-us/graph/onedrive-concept-overview) for more info.

### Get information about a file or folder

Use "items" resource to get information such as the ID, size, # of children, etc. about a file or folder:

```python
item_path = 'General'
item_url = urllib.parse.quote(item_path)
result = requests.get(f'{ENDPOINT}/drives/{drive_id}/root:/{item_url}', headers={'Authorization': 'Bearer ' + access_token})
item_info = result.json()
```

In this case, we querying information about the "General" files folder. `item_path` can refer to any valid path in the site.

### Listing folder contents

With the item ID of a folder, we can list its contents:

```python
result = requests.get(f'{ENDPOINT}/drives/{drive_id}/items/{folder_id}/children', headers={'Authorization': 'Bearer ' + access_token})
children = result.json()['value']
for item in children:
    print(item['name'])
```

See [this Gist](https://gist.github.com/keathmilligan/66b3cb84beb3aeb5807b8eed4f5b3d4d) for a complete example.

### Download a file

With the item ID of a file, you can download its contents:

```python
file_path = 'General/Test Doc.docx'
file_url = urllib.parse.quote(file_path)
result = requests.get(f'{ENDPOINT}/drives/{drive_id}/root:/{file_url}', headers={'Authorization': 'Bearer ' + access_token})
file_info = result.json()
file_id = file_info['id']

result = requests.get(f'{ENDPOINT}/drives/{drive_id}/items/{file_id}/content', headers={'Authorization': 'Bearer ' + access_token})
open(file_info['name'], 'wb').write(result.content)
```

See [this Gist](https://gist.github.com/keathmilligan/62a314e04880962427f031c10a184831) for a complete example.

### Upload a file

To upload a file, you have a couple of options. For small files (4MB or less), you can upload them directly using a single request, but for larger files, Graph documentation directs you to use a resumable upload session where you can upload the file in chunks.

Refer to the [Microsoft Graph API docs](https://docs.microsoft.com/en-us/graph/api/driveitem-put-content?view=graph-rest-1.0&tabs=http) for more information on uploading files.

#### Upload a small file (<4MB)

To upload a small file, you first need to check to see if the file already exists, in which case you will replace its contents:

```python
filename = 'Small File.txt'
folder_path = 'General'

path_url = urllib.parse.quote(f'{folder_path}/{filename}')
result = requests.get(f'{ENDPOINT}/drives/{drive_id}/root:/{path_url}', headers={'Authorization': 'Bearer ' + access_token})
if result.status_code == 200:
    file_info = result.json()
    file_id = file_info['id']
    result = requests.put(
        f'{ENDPOINT}/drives/{drive_id}/items/{file_id}/content',
        headers={
            'Authorization': 'Bearer ' + access_token,
            'Content-type': 'application/binary'
        },
        data=open(filename, 'rb').read()
    )
```

Otherwise, you will create a new item:

```python
elif result.status_code == 404:
    folder_url = urllib.parse.quote(folder_path)
    result = requests.get(f'{ENDPOINT}/drives/{drive_id}/root:/{folder_url}', headers={'Authorization': 'Bearer ' + access_token})
    result.raise_for_status()
    folder_info = result.json()
    folder_id = folder_info['id']

    file_url = urllib.parse.quote(filename)
    result = requests.put(
        f'{ENDPOINT}/drives/{drive_id}/items/{folder_id}:/{file_url}:/content',
        headers={
            'Authorization': 'Bearer ' + access_token,
            'Content-type': 'application/binary'
        },
        data=open(filename, 'rb').read()
    )
```

See [this Gist](https://gist.github.com/keathmilligan/8d177e524282b505bf9180f11cfbcc26) for a complete example.

#### Upload a large file (>4MB)

Uploading a large (>4MB) file is bit more involved because we need to use a resumable upload session to send the file in chunks. Further, [according to the docs](https://docs.microsoft.com/en-us/graph/api/driveitem-createuploadsession?view=graph-rest-1.0), the chunks have be a multiple of 320KB in size (yeah, that seems like a weird constraint to me too).

First get the destination folder ID as before:

```python
folder_path = 'General'
folder_url = urllib.parse.quote(folder_path)
result = requests.get(f'{ENDPOINT}/drives/{drive_id}/root:/{folder_url}', headers={'Authorization': 'Bearer ' + access_token})
folder_info = result.json()
folder_id = folder_info['id']
```

Next, we create the upload session:

```python
file_url = urllib.parse.quote(filename)
result = requests.post(
    f'{ENDPOINT}/drives/{drive_id}/items/{folder_id}:/{file_url}:/createUploadSession',
    headers={'Authorization': 'Bearer ' + access_token},
    json={
        '@microsoft.graph.conflictBehavior': 'replace',
        'description': 'A large test file',
        'fileSystemInfo': {'@odata.type': 'microsoft.graph.fileSystemInfo'},
        'name': filename
    }
)
upload_session = result.json()
upload_url = upload_session['uploadUrl']
```

Note the `@microsoft.graph.conflictBehavior` setting in the JSON request body. This determines what will happen if the file already exists. In this case, we have opted to overwrite the file if it already exists. Refer to the [docs](https://docs.microsoft.com/en-us/graph/api/driveitem-createuploadsession?view=graph-rest-1.0) for more info on this setting.

This request will return information about the setting including the URL where we will send the file chunks.

Now calculate the number of chunks you will need to send:

```python
st = os.stat(filename)
size = st.st_size
CHUNK_SIZE = 10485760
chunks = int(size / CHUNK_SIZE) + 1 if size % CHUNK_SIZE > 0 else 0
```

The documentation recommends a 10MB chunk size for most cases, but you may want to adjust this.

Now upload the chunks:

```python
with open(filename, 'rb') as fd:
    start = 0
    for chunk_num in range(chunks):
        chunk = fd.read(CHUNK_SIZE)
        bytes_read = len(chunk)
        upload_range = f'bytes {start}-{start + bytes_read - 1}/{size}'
        print(f'chunk: {chunk_num} bytes read: {bytes_read} upload range: {upload_range}')
        result = requests.put(
            upload_url,
            headers={
                'Content-Length': str(bytes_read),
                'Content-Range': upload_range
            },
            data=chunk
        )
        result.raise_for_status()
        start += bytes_read
```

See [this Gist](https://gist.github.com/keathmilligan/590a981cc629a8ea9b7c3bb64bfcb417) for a complete example.
