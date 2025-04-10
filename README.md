# Okta SAML Integration Setup Guide

This guide outlines the steps for creating and configuring an Okta SAML application and integrating it with your backend. The backend uses Okta as the Identity Provider (IdP) to handle SAML assertions and manage user sessions via JWT tokens.

---

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Okta Application Setup](#okta-application-setup)
- [Retrieving Okta Variables](#retrieving-okta-variables)
- [Integration Details](#integration-details)

---

## Overview

Your backend leverages Okta as its SAML Identity Provider. When users log in:

- They are redirected to Okta for authentication.
- Okta authenticates users and sends a SAML assertion back to the backend.
- The backend validates the assertion, extracts key user attributes (such as email, first name, and last name), and generates a JWT token to manage the session.

---

## Prerequisites

- **Okta Account:** Ensure you have an Okta Developer or Admin account to access the Admin Console.
- **Backend Application:** A backend integrated with the SAML code.
- **Metadata File:** Download the Okta SAML metadata XML file from your Okta application settings and place it in your project directory (for example, as `metadata.xml`).

---

## Okta Application Setup

Follow these steps to set up your Okta SAML application:

1. **Log In to the Okta Admin Console:**
   - Navigate to [Okta Admin Console](https://login.okta.com) (or your developer portal if using Okta Developer Edition).

2. **Create a New Application Integration:**
   - In the dashboard, go to **Applications** > **Applications**.
   - Click **Create App Integration**.
   - Select **SAML 2.0** as the sign-in method and click **Next**.

3. **Configure General Settings:**
   - **App Name:** Enter a descriptive name (e.g., “My Backend SAML App”).
   - **Single Sign-On URL (SSO URL):**  
     Provide the URL where Okta will send authentication requests. This is typically provided by Okta or can be set to your custom SSO endpoint.
   - **Audience URI (SP Entity ID):**  
     Enter the Service Provider Entity ID that matches your backend’s configuration.

4. **Set Up Attribute Statements (Claims):**
   - Configure the attributes you want included in the SAML assertion:
     - `email`
     - `firstName`
     - `lastName`  
   - The names should match the expectations in your backend SAML callback logic.

5. **Download the Metadata File:**
   - After configuration, download the metadata XML file from Okta.
   - Save this file in your backend project directory (e.g., as `metadata.xml`). The backend checks for this file during startup.

6. **Finish and Activate:**
   - Review the settings, complete the setup, and activate your Okta application.

---

## Retrieving Okta Variables

Your backend code requires several variables from Okta. Here is where you can find each of them in the Okta Admin Console:

- **OKTA_ENTITY_ID (Service Provider Entity ID/Audience URI):**
  - **Where to Find:** In your Okta SAML application settings, under the **General** section, locate the **Audience URI** (sometimes also referred to as the SP Entity ID). This value should be used as your `OKTA_ENTITY_ID` in the backend configuration.

- **OKTA_SSO_URL (Single Sign-On URL):**
  - **Where to Find:** Navigate to the **Sign On** tab or section within your Okta application configuration. The **SSO URL** is provided as the endpoint to which users are redirected for SAML authentication. Use this value as your `OKTA_SSO_URL`.

- **OKTA_ISSUER (Identity Provider Issuer):**
  - **Where to Find:** This is typically found in the Okta metadata XML file under the `<EntityDescriptor>` element or within the **Identity Provider** details of your Okta configuration. It represents the issuer of SAML assertions and should be set as `OKTA_ISSUER`.

- **OKTA_CERTIFICATE (X.509 Certificate):**
  - **Where to Find:** In your Okta SAML application's settings, look for the **SAML Signing Certificate** or a similar section. You can either copy the certificate details or download it. This certificate (in PEM format) is used to verify the SAML response signature and must be set as `OKTA_CERTIFICATE` in your backend configuration.

---

## Integration Details

### SAML Configuration

A dedicated configuration file (for example, `saml_constants.py`) loads the Okta SAML parameters such as `OKTA_ENTITY_ID`, `OKTA_SSO_URL`, `OKTA_ISSUER`, and `OKTA_CERTIFICATE`. These parameters are bundled into a dictionary (`SAML_SETTINGS`) used by the OneLogin SAML toolkit to manage authentication flows.

### Authentication Endpoints

Your FastAPI backend includes several endpoints:

- **`/saml/login`:**  
  Redirects the user to Okta’s SSO URL for authentication.

- **`/saml/callback`:**  
  Processes the SAML response from Okta. It validates the SAML assertion, extracts attributes (`email`, `firstName`, `lastName`), and generates a JWT token to manage the session. The token is set as an HTTP-only cookie.

- **`/me`:**  
  Verifies the JWT token and returns the authenticated user’s details.

- **`/logout`:**  
  Clears the authentication token and session data, redirecting the user to the login page.
