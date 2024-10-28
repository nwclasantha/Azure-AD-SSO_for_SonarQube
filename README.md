To set up SonarQube with Single Sign-On (SSO) using Azure Active Directory (Azure AD), you can follow these steps. This integration will enable users to sign into SonarQube using their Azure AD credentials.

### Prerequisites
1. **SonarQube** is installed and running (preferably SonarQube Enterprise Edition or Data Center Edition, as SSO might not be fully supported on the Community Edition).
2. **Administrator access** to both SonarQube and Azure AD.
3. **An Azure AD Premium P1 or P2 license**, as these licenses include SSO functionality.

Here's a structured outline with **Introduction**, **Objectives**, **Security Requirements**, and **Conclusion** for setting up SonarQube with Azure AD SSO.


### **Introduction**

As organizations adopt cloud-based solutions and centralized access management, integrating Single Sign-On (SSO) with tools like SonarQube is essential to simplify authentication, improve security, and enhance user experience. This guide provides detailed instructions on configuring SonarQube with Azure Active Directory (Azure AD) as an SSO provider. By integrating SonarQube with Azure AD, users can access the SonarQube platform using their existing Azure AD credentials, reducing the need for multiple passwords and enabling centralized control over user access.


### **Objectives**

The primary objectives of integrating SonarQube with Azure AD SSO are:

1. **Streamline User Access**: Enable users to log into SonarQube using their Azure AD credentials, minimizing the need for additional passwords and making access more convenient.
2. **Enhance Security**: Implement robust authentication protocols using Azure AD, ensuring that only authorized users can access SonarQube. By leveraging Azure AD’s security features, such as Multi-Factor Authentication (MFA) and Conditional Access, this integration significantly improves security.
3. **Centralize Access Management**: Use Azure AD as a single source of truth for managing user accounts and permissions, simplifying user provisioning and deprovisioning.
4. **Automate User Provisioning (Optional)**: Set up automated provisioning and role assignments using System for Cross-domain Identity Management (SCIM) if supported, which allows Azure AD to automatically add or remove users in SonarQube based on directory changes.

### **Security Requirements**

To securely implement SonarQube SSO with Azure AD, the following security requirements must be met:

1. **Azure AD Premium Licenses**: Ensure that users have Azure AD Premium P1 or P2 licenses, as these include SSO and conditional access features.
2. **HTTPS Configuration for SonarQube**: SonarQube should be configured with HTTPS to secure data transmitted between the browser, SonarQube server, and Azure AD. This is critical to protect authentication tokens and other sensitive data during transmission.
3. **Client Secret Management**: Use a secure method to store the client secret generated in Azure AD. The client secret should be kept confidential and not shared publicly, as it allows access to SonarQube’s authentication process.
4. **Multi-Factor Authentication (MFA)**: To enhance security, enable MFA for Azure AD users accessing SonarQube, reducing the risk of unauthorized access even if user credentials are compromised.
5. **Conditional Access Policies**: Configure conditional access policies in Azure AD to control access to SonarQube based on user location, device compliance, or other risk factors. This provides an added layer of security by preventing access from untrusted locations or devices.

Certainly! Here’s an even more detailed guide to setting up **SonarQube** with **Azure AD SSO**.

---

### Step 1: Register an Application in Azure AD

1. **Sign in to Azure Portal**:
   - Open a web browser and navigate to [https://portal.azure.com](https://portal.azure.com).
   - Sign in with an account that has administrative access to Azure Active Directory (Azure AD).

2. **Go to App Registrations**:
   - In the left sidebar, select **Azure Active Directory**.
   - In the Azure AD navigation pane, select **App registrations**.

3. **Register a New Application**:
   - Click **+ New registration** at the top of the page.
   - Fill out the registration form:
     - **Name**: Enter a name that helps you identify this app (e.g., `SonarQube SSO`).
     - **Supported account types**:
       - Choose **Single tenant** if only users in your directory will sign in.
       - Choose **Multi-tenant** if users from other Azure AD directories should be able to sign in.
     - **Redirect URI**:
       - Set **Type** to **Web**.
       - In the **URI** field, enter `https://your-sonarqube-url/oauth2/callback/azure`. Replace `your-sonarqube-url` with the URL where SonarQube is hosted (for example, `https://sonarqube.example.com/oauth2/callback/azure`).

4. **Complete the Registration**:
   - Click **Register** to create the application. Once created, you’ll be taken to the application’s **Overview** page.

5. **Copy the Application (Client) ID and Directory (Tenant) ID**:
   - On the **Overview** page, locate the **Application (client) ID** and **Directory (tenant) ID**. Save these values, as you’ll need them to configure SonarQube.

---

### Step 2: Configure Authentication for the Application

1. **Set Up Authentication Settings**:
   - In your registered app, go to **Authentication** under the **Manage** section on the left sidebar.
   - Under **Platform configurations**, select **+ Add a platform** and choose **Web**.

2. **Add Redirect URI**:
   - In the **Redirect URI** field, enter the URL you configured earlier for SonarQube (e.g., `https://your-sonarqube-url/oauth2/callback/azure`).
   - Make sure it matches the URL in SonarQube settings to avoid redirection issues.

3. **Enable ID Token**:
   - Scroll down to the **Implicit grant and hybrid flows** section.
   - Check the box for **ID tokens** to enable OpenID Connect authentication, which allows SonarQube to authenticate users.
   - Click **Save** to confirm your settings.

---

### Step 3: Add API Permissions

1. **Navigate to API Permissions**:
   - In the app’s left navigation, select **API Permissions**.

2. **Add Microsoft Graph API Permissions**:
   - Click **+ Add a permission** > **Microsoft Graph**.
   - Under **Permissions**, choose **Delegated permissions** (permissions granted on behalf of the signed-in user).
   - In the search bar, type `User.Read` and select it. This permission allows SonarQube to retrieve basic information about users.
   - Click **Add permissions** to confirm.

3. **Grant Admin Consent**:
   - Under the **API permissions** list, click **Grant admin consent for [Your Organization]** to allow the application to access the API on behalf of users.
   - Confirm the action. Once consent is granted, the permission status should show as **Granted**.

---

### Step 4: Generate a Client Secret

1. **Create a New Client Secret**:
   - In your application settings, go to **Certificates & secrets** under the **Manage** section.
   - Click **+ New client secret**.

2. **Add a Description and Expiration**:
   - Give the client secret a description (e.g., `SonarQube SSO Secret`).
   - Choose an expiration period based on your organization’s policies (e.g., 6 months, 1 year).
   - Click **Add**.

3. **Copy the Client Secret**:
   - After the secret is created, copy the **Value** immediately. This is your **client secret**, which will be needed in SonarQube settings.
   - **Important**: The client secret value is shown only once. Store it securely for later use.

---

### Step 5: Configure SonarQube for Azure AD SSO

1. **Log in to SonarQube as Admin**:
   - Open your SonarQube instance in a web browser and log in with an account that has administrative privileges.

2. **Navigate to Security Settings**:
   - In SonarQube, go to **Administration** > **Configuration** > **General Settings** > **Security**.

3. **Configure Azure AD Authentication Settings**:
   - Scroll down to the **Authentication** section, and enter the following settings:
     - **sonar.auth.aad.enabled**: Set this to `true` to enable Azure AD authentication.
     - **sonar.auth.aad.clientId**: Enter the **Application (client) ID** from your Azure AD app registration.
     - **sonar.auth.aad.clientSecret.secured**: Paste the **Client Secret** value from Step 4.
     - **sonar.auth.aad.tenantId**: Enter your **Directory (tenant) ID**.
     - **sonar.auth.aad.allowUsersToSignUp**: Set to `true` if you want new users to be able to sign up automatically the first time they log in. If not, set it to `false`.
     - **sonar.auth.aad.scopes**: Set this to `openid` (specifies the OpenID Connect scope needed to retrieve the user's identity).

4. **Save Changes**:
   - Click **Save** to apply your settings. This should enable Azure AD as an SSO option in SonarQube.

### Step 6: Test the SSO Configuration

1. **Log Out of SonarQube**:
   - Log out of your admin account to test the SSO login flow.

2. **Access the Login Page**:
   - Go to the SonarQube login page (`https://your-sonarqube-url/sessions/new`).

3. **Sign In with Azure AD**:
   - You should now see an option to **Log in with Azure Active Directory**.
   - Click this option, which will redirect you to the Azure AD login page.
   - Sign in with an Azure AD account. If everything is configured correctly, you should be authenticated and redirected back to SonarQube.

---

### Optional Step: Enable Automatic User Provisioning (SCIM) in Azure AD

1. **Set Up SCIM Provisioning in Azure AD**:
   - Go to the **Provisioning** tab in your Azure AD app registration.
   - Set the **Provisioning Mode** to **Automatic**.

2. **Configure SCIM Endpoint in SonarQube**:
   - In SonarQube’s settings, enable SCIM if your license supports it (Enterprise or Data Center Edition). The SCIM URL and authentication token would be generated in SonarQube for Azure AD to use.

3. **Set Up Automatic Provisioning**:
   - Enter the SCIM endpoint and secret token from SonarQube into Azure AD provisioning settings. This will allow Azure AD to manage user accounts in SonarQube automatically.

### Troubleshooting Tips

- **Redirect Errors**: Verify that the redirect URI entered in both Azure AD and SonarQube matches exactly, as mismatches can cause errors.
- **Permissions Denied**: Ensure `User.Read` permission is granted and admin consent is applied. Without this, SonarQube cannot retrieve user information from Azure AD.
- **Sign-Up Issues**: If users are not able to sign up automatically, confirm that `allowUsersToSignUp` is set to `true` in SonarQube settings.
- **Access Control**: Use SonarQube's role-based access settings to control user permissions after successful login.

This detailed setup will allow your users to log in to SonarQube using Azure AD credentials, enhancing security and simplifying the login process. Let me know if you need any additional help with specific steps!

### **Conclusion**

Integrating SonarQube with Azure AD for SSO is a significant step towards centralized, secure, and streamlined user authentication. By enabling users to access SonarQube with their existing Azure AD credentials, this integration reduces password fatigue and improves security through Azure AD’s advanced authentication and access control features. Additionally, with optional SCIM provisioning, administrators can further simplify user management, ensuring that access to SonarQube is consistently aligned with organizational roles and policies. Overall, this configuration not only strengthens the security of SonarQube but also enhances operational efficiency and user experience.

