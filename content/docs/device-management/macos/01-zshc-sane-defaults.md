---
title: "Sane defaults for scripts"
description: "A few simple hints for keeping your scripts secure"
layout: page
parent: macOS
nav_order: 1
date: 2026-11-13
last_modified_at: 2026-11-13
categories: device-management
tags: [apple, macos, snippets, scripting, shell, zsh]
---

This page shows a few hints I compiled over the past few years to keep your scripts secure and working on macOS.
Keep in mind tho, that this page is not complete by any means!

## Decisions, decisions...

...are not to be made on macOS when it comes to what shell you should use for your script.
With the introduction of macOS 10.15 Catalina back in 2019, Apple switched the default shell to `zsh`.
That's what I recommend scripting for. It is the Apple approved way, built into macOS and compatible with the majority of Shell and Bash scripts.
Enough reasons not to leave Apple's paved path 🙂

## Start clean

I run a lot of scripts triggered by Jamf Pro and I am sure you are, too!

To make sure your scripts run secure and consistent on your fleet, I highly recommend you tell zsh to ignore any user level configurations.
This can easily be done by adding `--no-rcs` to the _shebang_:

```shell
#!/bin/zsh --no-rcs
echo "Hello, $(whoami)"
```

(Source: [Scripting OS X](https://scriptingosx.com/2024/03/zsh-scripts-and-root-escalations/))

## SIP by default

Back in 2015, Apple introduced the [_System Integrity Protection_](https://support.apple.com/102149) (usually referred to as **SIP**).
This feature does one simple thing: Preventing key system components from modification!

{: .note-title}

> Pro-Tip
>
> All locations protected by SIP can be listed by running:
> ```shell
> cat /System/Library/Sandbox/rootless.conf
> ```

To ensure your script runs SIP-protected binaries exclusively, add this line to the top of your script (after the Shebang):

```shell
# Use only SIP protected binaries
export PATH=/usr/bin:/bin:/usr/sbin:/sbin
```

### Run non-SIP binaries alsongside

If your script calls binaries located outside the SIP protection, you are left with three options:

1. Call them by specifying the absolute path to the binary, e.g. like this: `/usr/local/bin/jamf policy -verbose`
2. Store the path to the binary as a variable, e.g.:

```shell
JAMF_BIN="/usr/local/bin/jamf"
${JAMF_BIN} policy -verbose
```

3. Use a function:

```shell
jamf() {
    "/usr/local/bin/jamf" "$@"
}

jamf policy -verbose
```

Personally, I like the function approach. But if you run a small script where an affected binary is just called once, the first option may be the easiest.
