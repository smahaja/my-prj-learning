# Azure AD (Entra ID) Authentication Setup Guide

## 📋 Overview

This guide will help you configure Azure AD (Microsoft Entra ID) authentication for the Cybersecurity Training Module. Once configured, users will need to sign in with their Microsoft accounts before accessing the training.

---

## 🎯 Prerequisites

- Azure subscription with Azure Active Directory (Microsoft Entra ID)
- Global Administrator or Application Administrator role in Azure AD
- The training module files deployed to a web server or accessible URL

---

## 🔧 Step-by-Step Configuration

### Step 1: Register Application in Azure Portal

1. **Navigate to Azure Portal**
   - Go to https://portal.azure.com
   - Sign in with your Azure AD administrator account

2. **Access Azure Active Directory**
   - In the left sidebar, click **"Azure Active Directory"** (or search for it)
   - If using Microsoft Entra admin center, go to https://entra.microsoft.com

3. **Register New Application**
   - Click **"App registrations"** in the left menu
   - Click **"+ New registration"** button
   
4. **Configure Application Registration**
   - **Name**: `Cybersecurity Training Module` (or your preferred name)
   - **Supported account types**: 
     - Choose **"Accounts in this organizational directory only"** (Single tenant)
     - OR **"Accounts in any organizational directory"** (Multi-tenant) if needed
   - **Redirect URI**:
     - Select **"Single-page application (SPA)"** from the dropdown
     - Enter your application URL, for example:
       - `https://yourdomain.com/cybersecurity-module-auth.html`
       - `https://yourcompany.github.io/training/cybersecurity-module-auth.html`
       - For local testing: `http://localhost:8080/cybersecurity-module-auth.html`
   - Click **"Register"**

### Step 2: Configure Authentication Settings

1. **After registration, you'll see the Overview page**
   - **Copy the Application (client) ID** - You'll need this
   - **Copy the Directory (tenant) ID** - You'll need this

2. **Configure Authentication**
   - Click **"Authentication"** in the left menu
   - Under **"Implicit grant and hybrid flows"**, check:
     - ✅ **"ID tokens"** (used for user sign-in)
   - Under **"Advanced settings"**:
     - Set **"Allow public client flows"** to **"No"**
   - Click **"Save"**

3. **Add Additional Redirect URIs (Optional)**
   - If you have multiple environments (dev, staging, production), add all URLs
   - Click **"+ Add URI"** for each additional URL
   - Examples:
     - Development: `http://localhost:8080/cybersecurity-module-auth.html`
     - Staging: `https://staging.yourdomain.com/cybersecurity-module-auth.html`
     - Production: `https://yourdomain.com/cybersecurity-module-auth.html`

### Step 3: Configure API Permissions

1. **Set Permissions**
   - Click **"API permissions"** in the left menu
   - You should see **"Microsoft Graph"** with **"User.Read"** permission (added by default)
   - This permission allows the app to read basic user profile information

2. **Grant Admin Consent (Optional but Recommended)**
   - Click **"✓ Grant admin consent for [Your Organization]"**
   - Click **"Yes"** to confirm
   - This pre-approves the permission for all users in your organization

### Step 4: Update Application Code

1. **Open the file**: `cybersecurity-module-auth.html`

2. **Locate the configuration section** (around line 150):

```javascript
const msalConfig = {
    auth: {
        clientId: "YOUR_CLIENT_ID_HERE",  // Replace this
        authority: "https://login.microsoftonline.com/YOUR_TENANT_ID_HERE", // Replace this
        redirectUri: window.location.origin + window.location.pathname,
    },
    cache: {
        cacheLocation: "localStorage",
        storeAuthStateInCookie: false,
    }
};
```

3. **Replace the placeholder values**:
   - Replace `YOUR_CLIENT_ID_HERE` with your **Application (client) ID**
   - Replace `YOUR_TENANT_ID_HERE` with your **Directory (tenant) ID**

**Example**:
```javascript
const msalConfig = {
    auth: {
        clientId: "12345678-1234-1234-1234-123456789abc",
        authority: "https://login.microsoftonline.com/87654321-4321-4321-4321-abcdef123456",
        redirectUri: window.location.origin + window.location.pathname,
    },
    cache: {
        cacheLocation: "localStorage",
        storeAuthStateInCookie: false,
    }
};
```

4. **Save the file**

### Step 5: Deploy and Test

1. **Deploy the Updated File**
   - Upload `cybersecurity-module-auth.html` to your web server
   - Ensure the URL matches one of the Redirect URIs you configured

2. **Test Authentication**
   - Open the application in a web browser
   - Click **"Sign in with Microsoft"**
   - You should be redirected to Microsoft login
   - Sign in with your organizational account
   - After successful authentication, you'll be redirected back to the training module

---

## 🌐 Deployment Options

### Option 1: GitHub Pages (Recommended for Quick Testing)

1. Enable GitHub Pages in your repository settings
2. Add redirect URI: `https://yourusername.github.io/repository-name/cybersecurity-module-auth.html`
3. Update the configuration in the HTML file
4. Commit and push changes
5. Access via the GitHub Pages URL

### Option 2: Azure Static Web Apps

1. Deploy to Azure Static Web Apps
2. Add redirect URI with your Azure Static Web Apps URL
3. Configure as described above
4. Deploy your changes

### Option 3: Corporate Web Server

1. Upload files to your internal web server
2. Use your corporate domain URL as redirect URI
3. Ensure HTTPS is enabled (required for production)
4. Update configuration and deploy

### Option 4: Local Testing

1. Use a simple HTTP server:
   ```bash
   # Python 3
   python -m http.server 8080
   
   # Node.js
   npx http-server -p 8080
   ```
2. Add redirect URI: `http://localhost:8080/cybersecurity-module-auth.html`
3. Access via `http://localhost:8080/cybersecurity-module-auth.html`

**Note**: For production use, always use HTTPS!

---

## 🔐 Security Best Practices

### 1. Use HTTPS in Production
- ✅ Always use HTTPS for production deployments
- ❌ HTTP is only acceptable for local development

### 2. Validate Redirect URIs
- Only add trusted URLs as redirect URIs
- Remove development/testing URLs in production app registration

### 3. Limit Permissions
- Only request necessary permissions (User.Read is sufficient for this app)
- Don't request admin-only permissions unless absolutely needed

### 4. Enable Conditional Access (Optional)
- Configure Conditional Access policies in Azure AD
- Examples:
  - Require MFA for accessing the training
  - Restrict access to specific locations
  - Allow access only from managed devices

### 5. Monitor Sign-in Logs
- Regularly review Azure AD sign-in logs
- Monitor for suspicious activities
- Set up alerts for unusual sign-in patterns

---

## 📊 Tracking and Reporting

### User Completion Tracking

The authenticated module logs the following information:

```javascript
{
    name: "John Doe",
    email: "john.doe@company.com",
    authTime: "2025-10-16T10:30:00.000Z",
    score: 85,
    completionTime: "2025-10-16T11:45:00.000Z"
}
```

### Option A: Log to Browser Console (Current)
- Completion data is logged to browser console
- Useful for testing and debugging
- Not suitable for production reporting

### Option B: Send to Backend API (Recommended for Production)

Add a backend endpoint to receive completion data:

```javascript
async function sendCompletionToBackend(user, score) {
    try {
        const response = await fetch('https://your-api.com/api/training/completion', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
            },
            body: JSON.stringify({
                userId: user.email,
                userName: user.name,
                score: score,
                completedAt: new Date().toISOString(),
                moduleName: 'Cybersecurity for Day-to-Day Work'
            })
        });
        
        if (response.ok) {
            console.log('Completion recorded successfully');
        }
    } catch (error) {
        console.error('Failed to record completion:', error);
    }
}
```

### Option C: Azure Application Insights

Add Application Insights for comprehensive tracking:

```javascript
// Add to your HTML
<script src="https://js.monitor.azure.com/scripts/b/ai.2.min.js"></script>

<script>
var appInsights = new Microsoft.ApplicationInsights.ApplicationInsights({
    config: {
        instrumentationKey: 'YOUR_INSTRUMENTATION_KEY'
    }
});
appInsights.loadAppInsights();
appInsights.trackPageView();

// Track completion
function trackCompletion(user, score) {
    appInsights.trackEvent({
        name: 'TrainingCompleted',
        properties: {
            userId: user.email,
            userName: user.name,
            score: score,
            module: 'Cybersecurity'
        }
    });
}
</script>
```

---

## 🛠️ Troubleshooting

### Issue: "Configuration error" on sign-in

**Cause**: Placeholder values not replaced in configuration

**Solution**:
- Verify you replaced `YOUR_CLIENT_ID_HERE` with actual Client ID
- Verify you replaced `YOUR_TENANT_ID_HERE` with actual Tenant ID
- Check that IDs are correctly formatted (GUIDs)

### Issue: "Redirect URI mismatch" error

**Cause**: Current URL doesn't match registered redirect URIs

**Solution**:
- Check Azure portal → App registrations → Authentication → Redirect URIs
- Add the exact URL where you're accessing the application
- URLs must match exactly (including protocol, domain, path, and query parameters)

### Issue: Popup blocked by browser

**Cause**: Browser security settings block popup windows

**Solution**:
- The application will automatically try redirect flow as fallback
- Allow popups for your domain in browser settings
- Or use redirect flow exclusively (modify code)

### Issue: "AADSTS50011: The reply URL specified in the request does not match"

**Cause**: Redirect URI type mismatch

**Solution**:
- In Azure portal, ensure redirect URI is set as **"Single-page application (SPA)"**
- Not "Web" type

### Issue: Users can't access training after sign-in

**Cause**: Iframe loading issue or file path problem

**Solution**:
- Verify `cybersecurity-module.html` exists in same directory
- Check browser console for errors
- Ensure both files are deployed to the same location

### Issue: "User.Read permission not granted"

**Cause**: API permissions not configured

**Solution**:
- Go to Azure portal → App registrations → API permissions
- Verify "Microsoft Graph - User.Read" is listed
- Click "Grant admin consent" button

---

## 📱 Multi-Tenant Setup (Optional)

To allow users from any Azure AD organization:

1. **Change supported account types**:
   - Go to Azure portal → App registrations → Authentication
   - Under "Supported account types", select:
     - **"Accounts in any organizational directory (Any Azure AD directory - Multitenant)"**

2. **Update authority in code**:
   ```javascript
   const msalConfig = {
       auth: {
           clientId: "YOUR_CLIENT_ID_HERE",
           authority: "https://login.microsoftonline.com/common", // Use 'common' instead of tenant ID
           redirectUri: window.location.origin + window.location.pathname,
       },
       // ... rest of config
   };
   ```

---

## 🔄 Updating Configuration

If you need to update configuration after deployment:

1. **Update Azure Portal Settings**:
   - Add/remove redirect URIs
   - Update permissions
   - Change supported account types

2. **Update Code**:
   - Modify `msalConfig` in `cybersecurity-module-auth.html`
   - Commit changes to git
   - Deploy updated file

3. **Test Changes**:
   - Clear browser cache and localStorage
   - Sign out if already signed in
   - Test sign-in flow again

---

## 📞 Support Resources

### Microsoft Documentation
- [Microsoft Authentication Library (MSAL.js)](https://github.com/AzureAD/microsoft-authentication-library-for-js)
- [Azure AD App Registration](https://docs.microsoft.com/en-us/azure/active-directory/develop/quickstart-register-app)
- [MSAL.js Tutorial](https://docs.microsoft.com/en-us/azure/active-directory/develop/tutorial-v2-javascript-auth-code)

### Common Endpoints
- Azure Portal: https://portal.azure.com
- Microsoft Entra Admin Center: https://entra.microsoft.com
- Graph Explorer (test Graph API): https://developer.microsoft.com/en-us/graph/graph-explorer

### Troubleshooting Tools
- Azure AD Sign-in Logs: Azure Portal → Azure AD → Sign-in logs
- App Registration Overview: Azure Portal → Azure AD → App registrations
- Browser Developer Tools: F12 → Console tab (for errors)

---

## ✅ Pre-Deployment Checklist

Before deploying to production:

- [ ] Application registered in Azure AD
- [ ] Client ID and Tenant ID obtained
- [ ] Redirect URIs configured correctly
- [ ] ID tokens enabled in Authentication settings
- [ ] User.Read permission granted
- [ ] Configuration values updated in HTML file
- [ ] Application deployed to web server with HTTPS
- [ ] Test sign-in with multiple user accounts
- [ ] Verify training module loads after authentication
- [ ] Test sign-out functionality
- [ ] Verify completion tracking works
- [ ] Review security settings
- [ ] Document configuration for team
- [ ] Set up monitoring/logging (optional)
- [ ] Train administrators on user management

---

## 📧 Getting Help

If you encounter issues not covered in this guide:

1. Check Azure AD sign-in logs for error codes
2. Review browser console for JavaScript errors
3. Verify all configuration steps were completed
4. Test with a different user account
5. Try incognito/private browsing mode
6. Contact your Azure AD administrator

---

**Last Updated**: October 2025  
**Version**: 1.0  
**Compatibility**: MSAL.js 2.x, Azure AD (Microsoft Entra ID)
