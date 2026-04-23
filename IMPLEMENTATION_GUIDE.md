# 📋 Signup & Authorization Code Management - Implementation Guide

## ✅ COMPLETED CHANGES

### 1. **Login Page Enhancement** (`login.html`)
- ✅ Added signup link at the bottom: **"Don't have an account? Sign up here"**
- Link directs to `/signup/` page
- Styled to match the existing design

### 2. **Signup Page Updates** (`signup.html`)
- ✅ Changed authorization code field to **optional** for first-time signups
- ✅ Updated metadata text to: "Program Chair · Auth code optional (first signup only)"
- ✅ Changed input type from password to text for auth code (for better UX)
- ✅ Updated placeholder: "Admin Authorization Code (optional for first signup)"

### 3. **Backend Model Changes** (`admin_models.py`)
Added new `SystemAuthCode` model:
```python
class SystemAuthCode(models.Model):
    code = models.CharField(max_length=50, default='ARCHI2025')
    changed_by = models.ForeignKey(Admin, ...)
    changed_at = models.DateTimeField(auto_now=True)
    created_at = models.DateTimeField(auto_now_add=True)
    
    @classmethod
    def get_current_code(cls):  # Get current system auth code
    
    @classmethod
    def set_code(cls, new_code, changed_by=None):  # Update system auth code
```

**Default Authorization Code:** `ARCHI2025`

### 4. **Forms Updates** (`forms.py`)
- ✅ Made `auth_code` field optional in `AdminSignupForm`
- ✅ Updated validation logic:
  - First signup (no admins exist) → auth code NOT required
  - Subsequent signups → auth code REQUIRED
  - Accepts: System auth code OR AdminAuthorization codes
- ✅ Added new form: `ChangeSystemAuthCodeForm`
  - Allows admins to change the system-wide authorization code
  - Validates code length (minimum 6 characters)
  - Requires confirmation of new code

### 5. **Views Updates** (`views.py`)

#### Modified `signup_view`:
```python
- First admin signup (no auth code required):
  * Creates admin with role='program_chair' (instead of 'admin')
  * Initializes SystemAuthCode with default "ARCHI2025"
  * Success message: "Account created as Program Chair! You can now manage auth codes."

- Subsequent signups (auth code required):
  * Checks if code matches SystemAuthCode OR AdminAuthorization codes
  * Validates expiration dates
  * Creates admin with role='admin'
  * Marks used codes as consumed
```

#### New View: `change_auth_code` (Line 161-190)
```python
@login_required
def change_auth_code(request):
    # Only program_chair or super_admin can access
    # GET: Shows current code and change form
    # POST: Updates system auth code
    # Tracks who changed it and when
```

### 6. **URL Routes** (`urls.py`)
- ✅ Added new route:
  ```python
  path('settings/change-auth-code/', views.change_auth_code, name='change_auth_code')
  ```

### 7. **Navigation Menu** (`navigation.html`)
- ✅ Added **Settings dropdown** (program_chair/super_admin only)
- ✅ Menu item: "Change Auth Code"
- ✅ Only visible to program chairs and super admins
- ✅ Positioned before Logout

### 8. **New Template** (`change_auth_code.html`)
- ✅ Beautiful settings page for changing authorization code
- ✅ Displays current authorization code prominently
- ✅ Warning box about impact of changing code
- ✅ Form with:
  - New authorization code input
  - Confirmation field
  - Submit and Cancel buttons
- ✅ Professional styling matching the application theme

### 9. **Database Migration** 
- ✅ Created migration file: `0008_systemauthcode.py`
- ✅ Creates `system_auth_code` table
- ✅ Fields: code, changed_by, changed_at, created_at

---

## 🔄 WORKFLOW

### First-Time Setup (Program Chair Sign-up):
```
1. User visits login page
2. Clicks "Sign up here" link
3. Fills signup form WITHOUT authorization code
4. Creates account with role='program_chair'
5. System initializes with default code "ARCHI2025"
6. User redirected to login page
7. Program chair logs in normally
```

### Program Chair Updates Authorization Code:
```
1. Program chair logs in
2. Navigates to: Settings → Change Auth Code (in sidebar)
3. Sees current code: "ARCHI2025"
4. Enters new code (minimum 6 characters)
5. Confirms new code
6. Submits change
7. System records: who changed it, when, new code
8. Success message displayed
9. Future admins must use the new code to sign up
```

### New Admin Sign-up (After Program Chair Setup):
```
1. User visits login page
2. Clicks "Sign up here" link
3. Fills signup form with authorization code
4. Code validated against:
   a) Current system auth code (from SystemAuthCode model)
   b) AdminAuthorization codes (existing codes from admin interface)
5. If valid: Account created with role='admin'
6. If admin code used: Marked as consumed
7. User redirected to login page
```

---

## 🔐 Authorization Code Flow

### System Authorization Code (Default):
- **Default Value:** `ARCHI2025`
- **Storage:** `system_auth_code` table
- **Usage:** New admin signups
- **Changeable:** Yes (by program chair)
- **Track Changes:** Yes (who & when recorded)

### Admin Authorization Codes:
- **Storage:** `admin_authorizations` table  
- **Usage:** Single-use codes created by admin
- **Expiration:** Yes (time-based)
- **Status:** Can be marked as used
- **Track Usage:** Yes (who used it recorded)

---

## 📝 IMPORTANT NOTES

1. **First Admin Only:** Only the first admin can sign up without an authorization code. This is the program chair.

2. **Default Code Remains:** The hardcoded "ARCHI2025" code remains in effect until the program chair actively changes it.

3. **Role Assignment:**
   - First signup → `role = 'program_chair'`
   - Subsequent signups → `role = 'admin'`

4. **Access Control:** Only program chairs and super admins can change the authorization code via the Settings page.

5. **Security:** Authorization code changes are tracked with:
   - Who changed it (Admin user)
   - When it was changed (timestamp)
   - New code value

6. **Admin Creation:** The form still references email field, but the Admin model doesn't store email (commented as "WALA NA! EMAIL IS GONE!"). This should be cleaned up if email is not used.

---

## ✨ FEATURES IMPLEMENTED

- [x] Signup link on login page
- [x] Optional auth code for first signup
- [x] First admin becomes program chair
- [x] Program chair can change auth code
- [x] Default code "ARCHI2025" stored in database
- [x] System tracks who changed code and when
- [x] Settings page accessible from navigation
- [x] Authorization code validation with multiple sources
- [x] Professional UI for all new pages
- [x] Proper error handling and user feedback

---

## 🚀 NEXT STEPS

1. **Apply Migrations:**
   ```bash
   python manage.py migrate
   ```

2. **Test First Signup:**
   - Clear all admins from database (if testing)
   - Visit signup page
   - Sign up without authorization code
   - Verify admin is created with role='program_chair'
   - Verify can login

3. **Test Auth Code Change:**
   - Login as program chair
   - Go to Settings → Change Auth Code
   - Change code to something else (e.g., "NEW2025")
   - Logout

4. **Test Second Admin Signup:**
   - Visit signup page
   - Try with old code → Should fail
   - Try with new code → Should succeed
   - Verify admin is created with role='admin'

5. **Test Admin Authorization Codes:**
   - Create AdminAuthorization codes
   - Test signup with those codes
   - Verify they get marked as used

---

## 📁 FILES MODIFIED

1. `admin_models.py` - Added SystemAuthCode model
2. `forms.py` - Updated AdminSignupForm, added ChangeSystemAuthCodeForm
3. `views.py` - Modified signup_view, added change_auth_code view
4. `urls.py` - Added route for change_auth_code
5. `templates/login.html` - Added signup link
6. `templates/signup.html` - Made auth code optional
7. `templates/navigation.html` - Added Settings dropdown
8. `templates/change_auth_code.html` - NEW settings page
9. `migrations/0008_systemauthcode.py` - NEW database migration

---

**Implementation Date:** April 23, 2026
**Status:** ✅ COMPLETE & READY FOR TESTING
