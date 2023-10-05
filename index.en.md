---
title : "Getting Started"
weight : 10
---

## Overview

If you are attending an AWS hosted event, you will have access to an AWS account with any optional pre-provisioned infrastructure and IAM policies needed to complete this workshop. The goal of this section is to help you access this AWS account. You may skip this section if you plan to use your own AWS account.

## Prerequisites

-   Access to the 12 digit event access code distributed by an event operator as part of an AWS hosted event.
-   Open a new window or tab in incognito mode to avoid any issues with existing login sessions to the AWS Console
-   Sign in via the Workshop Studio [join url](https://catalog.workshops.aws/join)  or the one-click join event link provided by the event operator.

![Workshop Studio](/static/img/getstarted/ws_join.png)

![Workshop Studio](/static/img/getstarted/ws_event_code.png)

-   Carefully review the terms and conditions associated with this event.

![Workshop Studio](/static/img/getstarted/ws_review_join.png)

## Best Practices

-   Review the terms and conditions of the event. Do not upload any personal or confidential information in the account.
-   The AWS account will only be available for the duration of this workshop and you will not be able to retain access after the workshop is complete. Backup any materials you wish to keep access to after the workshop.
-   Any pre-provisioned infrastructure will be deployed to a specific region. Check your workshop content to determine whether other regions will be used.

## Accessing your AWS Account

After joining the event, you should see the page with event information and workshop details. You should also see a section titled "AWS account access" on the left navigation bar. You can use these options to access the temporary AWS account provided to you.

![Workshop Studio](/static/img/getstarted/ws_account.png)

### AWS Management Console

The “AWS console” link will open the AWS Management Console home page. This is the standard AWS Console that provides access to each service. Please note that the infrastructure associated with the workshop will be deployed to a specific region and can be only accessed from that region.

Ensure that you are logged in as `WSParticipantRole`, as can be verified on the top right of the screen.

![Workshop Studio](/static/img/getstarted/ws_wsparticipantrole.png)

### AWS Command Line Interface (AWS CLI)

The “AWS CLI credentials” link provides environment variables to copy and paste into your terminal on your local machine to setup your AWS CLI for the temporary AWS account provided to you. The AWS CLI is an open source tool that enables you to interact with AWS services using commands in your command-line shell. The AWS CLI enables you to run commands that implement functionality equivalent to that provided by the browser-based AWS Management Console from the command prompt in your terminal.

**Note:** these credentials expire 60 minutes from the initial start of the CLI session. Once expired, you will need to copy a new set of credentials from this page into your terminal. 
 
**Note:** These are temporary credentials created by the Amazon STS service. This type of temporary credential has an Access Key, Secret Key, and a Session Token. All three must also be copied into your terminal. Copying just the access / secret key will result in unusable/invalid credentials.

## Summary

Once you've verified access to the AWS account provided to you and any relevant resources in that account, you should have everything you need to get started with this workshop.

In this section, you were introduced to accessing the AWS account that is pre-provisioned with infrastructure necessary for this workshop.