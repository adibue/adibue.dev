---
title: "Jamf Platform API"
description: "A few notes and snippets to make scripting a little easier :-)"
layout: page
parent: Jamf
nav_order: 1
date: 2026-11-13
last_modified_at: 2026-11-13
categories: device-management
tags:
  [
    jamf,
    device-management,
    jamf-pro,
    jamf-api,
    notes,
    snippets,
    mdm,
    apple,
    macos,
  ]
---

If your company partners with Jamf for its device management, chances are you do have multiple Jamf products in use - each of them having their own API endpoints and clients.
To make the life of SysAdmins a little easier, Jamf introduced the [Platform API Gateway](https://developer.jamf.com/platform-api/reference/getting-started-with-platform-api) - one API platform to rule 'em all!

This page serves as a collection of samples and snippets (mainly for using in shell scripts), so one does not have to write one's common API calls over and over again.

<!-- prettier-ignore-start -->

{: .highlight-title }
> Table of Contents
>
>- TOC
>{:toc}

<!-- prettier-ignore-end -->

---

## Platform API Basics

For interacting with the Platform API, create an [_Integration_](https://account.jamf.com/platform-apis/mfe) in your Jamf Account first.
In my examples, I use the level _**Tenant**_ in the _Region_ **EU**. If your setup differs, adjust the variables mentioned in the next section accordingly.
If you are unsure what permission your integration client is going to require, just have a look at the [Platform API Gateway documentation](https://developer.jamf.com/platform-api/reference/getting-started-with-platform-api), where all the permission requirements for every API endpoint are listed.

### Basic variables

The first things we need, are four variables:

- `JAMF_API_URL`: The base URL in the format `https://{region}.api.jamfcloud.com`, where `region` is either `us`, `eu` or `apac`, depending on your location.
- `TENANT_ID`: The _Tenant ID_ the API calls are being sent to. Should work similarly for `Platform Environment ID` and `Organization ID`
- `CLIENT_ID`: The _Client ID_ of your integration (surprise!)
- `CLIENT_SECRET`: You will never guess this one 🙃

{: .note-title }

> Local .env for testing
>
> In Jamf Pro, such variables can be passed as script parameters (_\$4-\$11_).
> For developing and testing locally, just write them into an `.env` file (_**you absolutely want this in your `.gitignore`!**_) placed next to the script you are working on and use the following function to source them:
>
> ```shell
> overrideFromEnv() {
>    # Override variables from .env file if found
>    if [[ -f ".env" ]]; then
>        echo "Loading variables from .env file..."
>        source .env
>    fi
> }
> ```

### Bearer Token creation

With the required variables in place, this function will generate a bearer token and provide 3 variables for you to utilize:

- `access_token`: The generated bearer token used for authenticating subsequent API calls

Helpful in specific cases, but unlikely you will ever need them:

- `token_expires_in`: Expiration time of the generated bearer token
- `token_expiration_epoch`: Same as above, but in epoch (useful if you want to extend the expiration)

```shell
## Request Bearer Token from Jamf Platform API
getBearerToken() {
    # Request bearer token from Jamf Platform API
    local current_epoch=$(date +%s)
    local response=$(curl -sX POST "${JAMF_API_URL}/auth/token" \
        -H "Content-Type: application/x-www-form-urlencoded" \
        -d "client_id=${CLIENT_ID}" \
        -d "client_secret=${CLIENT_SECRET}" \
        -d "grant_type=client_credentials")
    access_token=$(echo "$response" | jq -r '.access_token')
    token_expires_in=$(echo "$response" | jq -r '.expires_in')
    token_expiration_epoch=$(($current_epoch + $token_expires_in - 1))
}
```

---

## Example workflows

Just a few basic example workflows you can build upon.

### Device ID from Serial

For interacting with all sorts of things, e.g. adding a device to a static group, Jamf Pro usually requires the _Computer ID_ rather than the serial number. That's why we need some kind of conversion that ingests the serial number (which we know) and hands us the device's ID (which we don't know).
Something that you might use quite frequently (at least I do :-)), so here's how.

First of all, we will read the serial number and store it as `HW_SERIAL`:
`HW_SERIAL=$(system_profiler SPHardwareDataType | awk '/Serial/ {print $4}')`

This function now calls the API (`/proclassic/computermanagement/serialnumber/${HW_SERIAL}`) to request the details of the device matching the serial number, isolates the ID, stores it as `computer_id` and finally echoes it out:

#### Function: getComputerID

```shell
## Get Computer ID from Jamf Platform API
getComputerID() {
    local id_response=$(curl -sX GET "${JAMF_API_URL}/proclassic/computermanagement/serialnumber/${HW_SERIAL}" \
        -H "X-Tenant-Id: ${TENANT_ID}" \
        -H "Authorization: Bearer ${access_token}" \
        -H "Accept: application/json" )
    computer_id=$(echo "$id_response" | jq -r '.computer_management.general.id')
    echo "Computer ID: $computer_id"
}
```

#### Example Script: getIdFromSerial

The whole workflow as a simple script:

```shell
#!/bin/zsh --no-rcs

# Store serial number as variable
HW_SERIAL=$(system_profiler SPHardwareDataType | awk '/Serial/ {print $4}')

# Local override for testing
overrideFromEnv() {
    # Override variables from .env file if found
    if [[ -f ".env" ]]; then
        echo "Loading variables from .env file..."
        source .env
    fi
}

## Request Bearer Token from Jamf Platform API
getBearerToken() {
    # Request bearer token from Jamf Platform API
    local current_epoch=$(date +%s)
    local response=$(curl -sX POST "${JAMF_API_URL}/auth/token" \
        -H "Content-Type: application/x-www-form-urlencoded" \
        -d "client_id=${CLIENT_ID}" \
        -d "client_secret=${CLIENT_SECRET}" \
        -d "grant_type=client_credentials")
    access_token=$(echo "$response" | jq -r '.access_token')
    token_expires_in=$(echo "$response" | jq -r '.expires_in')
    token_expiration_epoch=$(($current_epoch + $token_expires_in - 1))
}

## Get Computer ID from Jamf Platform API
getComputerID() {
    local id_response=$(curl -sX GET "${JAMF_API_URL}/proclassic/computermanagement/serialnumber/${HW_SERIAL}" \
        -H "X-Tenant-Id: ${TENANT_ID}" \
        -H "Authorization: Bearer ${access_token}" \
        -H "Accept: application/json" )
    computer_id=$(echo "$id_response" | jq -r '.computer_management.general.id')
    echo "Computer ID: $computer_id"
}

# Function calling
overrideFromEnv
getBearerToken
getComputerID
```
