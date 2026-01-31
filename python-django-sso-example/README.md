# WorkOS SSO Django Integration - Technical Challenge

**Repository:** [https://github.com/Mr-Sean/WorkOS_SSO_Tech_Challenge](https://github.com/Mr-Sean/WorkOS_SSO_Tech_Challenge)

A Django application demonstrating Single Sign-On (SSO) integration using the WorkOS API and Test Identity Provider.

## Overview

This project extends the [WorkOS Python Django SSO example](https://github.com/workos/python-django-example-applications) to implement a complete SSO authentication flow that:
- Authenticates users via the WorkOS Test Identity Provider
- Displays user profile information (first name, last name)
- Shows organization details (organization ID and name)
- Makes an additional API call to fetch the organization name

## Prerequisites

Before you begin, ensure you have the following installed:
- **Python 3.10+** (Python 3.10.0 or higher)
- **Git** (for cloning the repository)
- **pip** (Python package manager)
- A **WorkOS account** ([Sign up here](https://dashboard.workos.com/))

## Project Setup

### Step 1: Clone the Repository

```bash
git clone https://github.com/Mr-Sean/WorkOS_SSO_Tech_Challenge.git
cd WorkOS_SSO_Tech_Challenge
```

### Step 2: Create a Virtual Environment

Create and activate a Python virtual environment to isolate project dependencies:

**On Windows (PowerShell or Command Prompt):**
```bash
python -m venv venv
venv\Scripts\activate
```

**On macOS/Linux:**
```bash
python3 -m venv venv
source venv/bin/activate
```

You should see `(venv)` at the beginning of your terminal prompt, indicating the virtual environment is active.

### Step 3: Install Dependencies

Install all required Python packages:

```bash
pip install -r requirements.txt
```

This will install:
- Django 4.1.3
- WorkOS Python SDK 5.40.0
- python-dotenv for environment variable management
- Other required dependencies

### Step 4: Configure Environment Variables

Create a `.env` file in the project root directory with your WorkOS credentials:

```env
WORKOS_API_KEY=your_workos_api_key_here
WORKOS_CLIENT_ID=your_workos_client_id_here
REDIRECT_URI=http://localhost:8000/auth/callback
CUSTOMER_ORGANIZATION_ID=org_01KFFRB2F17SH6C3JRX0Q45HQ5
```

**Where to find these values:**
1. Log into your [WorkOS Dashboard](https://dashboard.workos.com/)
2. Go to **API Keys** (in the Developer section) to get your:
   - `WORKOS_API_KEY` (starts with `sk_test_`)
   - `WORKOS_CLIENT_ID` (starts with `client_`)
3. The `CUSTOMER_ORGANIZATION_ID` uses the Test Organization that comes pre-configured in your staging environment
4. The `REDIRECT_URI` is the callback endpoint where WorkOS redirects after authentication

**Important:** Never commit the `.env` file to version control. It's already included in `.gitignore`.

### Step 5: Run Database Migrations

Set up the Django database:

```bash
python manage.py migrate
```

You should see output confirming that migrations were applied successfully.

### Step 6: Start the Development Server

Run the Django development server:

```bash
python manage.py runserver
```

The server will start at `http://127.0.0.1:8000/` (or `http://localhost:8000/`)

## Testing the SSO Integration

### Step 1: Access the Application

Open your web browser and navigate to:
```
http://localhost:8000
```

You should see a login page with three authentication options:
- Google OAuth
- Microsoft OAuth
- **Enterprise SAML** ← Click this one

### Step 2: Authenticate with Test Provider

1. Click the **"Enterprise SAML"** button
2. You'll be redirected to the WorkOS Test Identity Provider page
3. Fill in the test user information:
   - **Email:** Must use `@example.com` domain (e.g., `john.doe@example.com`)
   - **First name:** Any first name (e.g., `John`)
   - **Last name:** Any last name (e.g., `Doe`)
   - **Group:** (Optional) Any group name
4. Click **"Continue"**

**Important:** The email must use the `@example.com` domain to match the Test Organization's verified domain.

### Step 3: View Authentication Results

After successful authentication, you'll be redirected back to your application and see:

**User Information:**
- First Name (from the identity provider)
- Last Name (from the identity provider)
- Organization ID
- Organization Name (fetched via additional API call)

**Raw Profile Data:**
- Complete JSON profile data returned from WorkOS

## Code Changes Made

This project includes the following modifications to the original WorkOS example:

### 1. Environment Configuration (`.env`)
Added the `CUSTOMER_ORGANIZATION_ID` environment variable to specify which organization to authenticate against.

### 2. Updated `sso/views.py`

**Disabled local API base URL override** (lines 54-56):
```python
# Commented out to use the actual WorkOS API instead of localhost
# if settings.DEBUG:
#     os.environ["WORKOS_API_BASE_URL"] = "http://localhost:8000/"
```

**Enhanced `auth_callback` function** (lines 135-160):
- Extracts `first_name`, `last_name`, and `organization_id` from the SSO profile
- Makes an additional API call to fetch the organization name:
  ```python
  organization = client.organizations.get_organization(organization_id)
  organization_name = organization.name
  ```
- Stores all information in the Django session

**Updated `login` function** (lines 67-79):
- Passes additional variables to the template:
  - `first_name`
  - `last_name`
  - `organization_id`
  - `organization_name`

### 3. Updated `sso/templates/sso/login_successful.html`

Added a clean display section for the required information (lines 46-53):
```html
<div style="background: white; padding: 20px; border-radius: 8px; margin-top: 20px;">
  <h3 style="margin-top: 0;">User Information</h3>
  <p><strong>First Name:</strong> {{ first_name }}</p>
  <p><strong>Last Name:</strong> {{ last_name }}</p>
  <p><strong>Organization ID:</strong> {{ organization_id }}</p>
  <p><strong>Organization Name:</strong> {{ organization_name }}</p>
</div>
```

## Project Structure

```
WorkOS_SSO_Tech_Challenge/
├── sso/                          # Django app for SSO functionality
│   ├── views.py                  # SSO authentication logic (modified)
│   ├── templates/
│   │   └── sso/
│   │       ├── login.html        # Login page
│   │       └── login_successful.html  # Success page (modified)
│   └── ...
├── workos_django/                # Main Django project
│   ├── settings.py               # Django settings
│   ├── urls.py                   # URL routing
│   └── ...
├── .env                          # Environment variables (create this)
├── .gitignore                    # Git ignore rules
├── manage.py                     # Django management script
├── requirements.txt              # Python dependencies
└── README.md                     # This file
```

## Troubleshooting

### Issue: "No Connection associated with Organization"

**Solution:** Make sure you're using the Test Organization ID (`org_01KFFRB2F17SH6C3JRX0Q45HQ5`) in your `.env` file, which comes pre-configured in the WorkOS staging environment.

### Issue: "Profile does not belong to the target organization"

**Solution:** Use an email address with the `@example.com` domain when testing (e.g., `test@example.com`). This matches the Test Organization's verified domain.

### Issue: "This is not a valid redirect URI"

**Solution:** 
1. Go to your [WorkOS Dashboard](https://dashboard.workos.com/)
2. Navigate to **Redirects** (in the Developer section)
3. Ensure `http://localhost:8000/auth/callback` is listed as an allowed redirect URI

### Issue: Virtual environment won't activate

**Windows PowerShell users:** If you get a security error, run:
```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

### Issue: Port 8000 is already in use

**Solution:** Either stop the other process using port 8000, or run Django on a different port:
```bash
python manage.py runserver 8080
```
Then update your redirect URI to `http://localhost:8080/auth/callback` in both the `.env` file and WorkOS dashboard.

## Resources

- [WorkOS Documentation](https://workos.com/docs)
- [WorkOS SSO Documentation](https://workos.com/docs/sso)
- [WorkOS Test Provider Documentation](https://workos.com/docs/sso/test-sso)
- [WorkOS Python SDK](https://github.com/workos/workos-python)
- [Django Documentation](https://docs.djangoproject.com/)

## Technical Challenge Completion

This project fulfills all requirements of the WorkOS Technical Challenge:

✅ **Setup:** Cloned the WorkOS Python Django example application  
✅ **SSO Integration:** Connected to the Test Provider  
✅ **Authentication:** Users can sign in via the Test Provider  
✅ **Display Requirements:**
  - First name and last name from identity provider
  - Organization ID
  - Organization name (via additional GET request)  
✅ **Documentation:** Comprehensive README with setup instructions  
✅ **Demo Recording:** Available in repository

## Support

If you encounter any issues:
1. Check the Troubleshooting section above
2. Review the [WorkOS Documentation](https://workos.com/docs)
3. Contact WorkOS support at support@workos.com

## Author

**Sean** - [GitHub Profile](https://github.com/Mr-Sean)

## License

This project is based on the [WorkOS Python Django example](https://github.com/workos/python-django-example-applications) and follows the same license terms.
