---
name: set-up-django-gdpr-cookie-consent
description: >
  Step-by-step guided setup of the django-gdpr-cookie-consent package into a Django project.
  Use this skill whenever the user wants to add GDPR cookie consent to a Django project,
  set up a cookie consent dialog/banner, configure cookie categories, integrate Google Analytics
  or other tracking scripts with GDPR compliance, or asks anything about django-gdpr-cookie-consent.
  Even if they just mention "cookie consent", "GDPR cookies", or "cookie banner" in a Django context,
  use this skill — it handles the full automated setup end to end.
---

# Setting Up django-gdpr-cookie-consent

This skill guides you through integrating `django-gdpr-cookie-consent` into a Django project.
Automate everything you can via file inspection and code edits; ask the user only when something
is genuinely ambiguous or requires a judgment call.

---

## Phase 1: Discover the project layout

Run these commands to understand the project before touching anything. Adapt paths if the user
tells you they're working in a specific subdirectory.

```bash
# Find manage.py — its directory is the Django project root
find . -name "manage.py" -not -path "*/node_modules/*" -not -path "*/.venv/*" -not -path "*/venv/*" 2>/dev/null

# Find settings files (may be split: base.py, _base.py, local.py, production.py, etc.)
find . \( -name "settings*.py" -o -path "*/settings/*.py" \) \
  -not -path "*/node_modules/*" -not -path "*/.venv/*" -not -path "*/venv/*" 2>/dev/null

# Find URL root config
grep -r "ROOT_URLCONF" --include="*.py" -l . 2>/dev/null | head -5

# Find base/layout templates likely to need the cookie widget
find . -name "base.html" -o -name "base_site.html" -o -name "layout.html" 2>/dev/null | grep -v ".venv" | grep -v "/venv/"

# Check if package is already installed
pip show django-gdpr-cookie-consent 2>/dev/null || echo "NOT INSTALLED"

# Detect pip-compile workflow (*.in files mean pip-compile or pip-compile-multi is in use)
find . -name "requirements*.in" -o -path "*/requirements/*.in" 2>/dev/null \
  -not -path "*/node_modules/*" -not -path "*/.venv/*" -not -path "*/venv/*"
which pip-compile 2>/dev/null || echo "pip-compile NOT FOUND"
which pip-compile-multi 2>/dev/null || echo "pip-compile-multi NOT FOUND"

# Try to find the wheel file before asking the user
find . -name "django_gdpr_cookie_consent*.whl" \
  -not -path "*/node_modules/*" -not -path "*/.venv/*" -not -path "*/venv/*" 2>/dev/null
```

From this, establish:
- **Project root** (where `manage.py` lives)
- **Settings file(s)** to edit — if there are multiple, ask which one is the development/primary settings file unless it's obvious (e.g. `settings/base.py` is the clear answer)
- **Root URL config** file
- **Base template** file (if multiple candidates, ask which one wraps every page)
- **Requirements workflow**: plain `*.txt`, or pip-compile (`*.in` → `*.txt`), or pip-compile-multi
- **Wheel file path**: found automatically, or unknown (will ask user in Phase 2)

---

## Phase 2: Check installation

The package is a paid private wheel.

If the `find` command in Phase 1 located a `.whl` file, use that path directly and skip asking.
If no wheel was found, ask the user:

> "I couldn't find `django_gdpr_cookie_consent-*.whl` anywhere in the project.
> Do you have it somewhere on your machine? If not, you'll need to download it from Gumroad first."

Once the wheel path is known, help them install it. The conventional approach is:

```bash
# Place wheel in a private_wheels/ directory at the project root
mkdir -p private_wheels
cp /path/to/wheel private_wheels/
```

Then add the wheel reference to the appropriate source file and install, depending on the workflow
detected in Phase 1:

**Plain `*.txt` workflow** (no `*.in` files found):
Add to `requirements.txt` (or `requirements/base.txt` if split):
```
file:./private_wheels/django_gdpr_cookie_consent-5.x.x-py3-none-any.whl
```
Then install:
```bash
pip install -r requirements.txt
```

**`pip-compile` workflow** (`requirements.in` or `requirements/*.in` files found, `pip-compile` available):
Add to the relevant `*.in` file (e.g. `requirements.in` or `requirements/base.in`):
```
file:./private_wheels/django_gdpr_cookie_consent-5.x.x-py3-none-any.whl
```
Then recompile and install:
```bash
pip-compile requirements.in          # or the specific *.in file you edited
pip install -r requirements.txt      # use the compiled output file
```

**`pip-compile-multi` workflow** (`*.in` files found, `pip-compile-multi` available):
Add to the relevant `*.in` file (e.g. `requirements/base.in`):
```
file:./private_wheels/django_gdpr_cookie_consent-5.x.x-py3-none-any.whl
```
Then recompile all outputs and install:
```bash
pip-compile-multi                    # recompiles all *.in → *.txt files
pip install -r requirements/base.txt # or whichever compiled file is used for local dev
```

If the package is already installed (detected in Phase 1), skip this phase.

---

## Phase 3: Gather configuration decisions from the user

Before making any edits, ask these questions in a single message (don't pepper them one at a time):

**1. Cookie categories** — Which of these do you need beyond Essential?
   - **Functionality** (e.g. remembering language/theme preferences)
   - **Performance/Analytics** (e.g. Google Analytics, Matomo)
   - **Marketing** (e.g. Google Ads, Facebook Pixel)

**2. Dialog position** — Where should the consent dialog appear?
   - center (modal), top, bottom, left, right
   - Default: **center**

**3. Base template name** — What is the `extends` value your templates use?
   - e.g. `"base.html"`, `"layouts/base.html"` — needed for the cookie management page
   - You can usually auto-detect this from the base template's own filename/path relative to the templates directory

**4. Services in use** — Run this to detect common third-party services already in templates:
   ```bash
   grep -r "google-analytics\|gtag\|GTM-\|googletagmanager\|facebook\.net\|fbq\|hotjar\|clarity\|matomo\|plausible" \
     --include="*.html" --include="*.js" -l . 2>/dev/null | grep -v ".venv" | grep -v "/venv/"
   ```
   Tell the user what you found and ask them to confirm which services they use.

**5. Website domain** — What is the production domain? (e.g. `.example.com`) — used in cookie entries.

**6. Button CSS classes (auto-detect)** — Run these commands to find button styles already used in the project's templates, so the consent dialog buttons match the site's design:

```bash
# Extract class attributes from <button> and <input type="button/submit"> tags
grep -rEoh '<(button|input)[^>]*(class="[^"]*")[^>]*>' \
  --include="*.html" . 2>/dev/null | grep -v ".venv" | grep -v "/venv/"

# Also search for common CSS framework button class patterns directly
grep -rEoh 'class="[^"]*b(tn|utton)[^"]*"' \
  --include="*.html" . 2>/dev/null | grep -v ".venv" | grep -v "/venv/" | sort -u

# Tailwind: look for bg-* / text-* combos on button elements
grep -rEoh '<(button|input)[^>]*class="[^"]*bg-[a-z]+-[0-9]+[^"]*"[^>]*>' \
  --include="*.html" . 2>/dev/null | grep -v ".venv" | grep -v "/venv/"
```

Analyse the output and determine:
- **Primary button classes**: the classes used on the most prominent/call-to-action buttons
  (e.g. `btn btn-primary`, `bg-blue-600 text-white hover:bg-blue-700 px-4 py-2 rounded`)
- **Secondary button classes**: the classes used on less-prominent/cancel buttons
  (e.g. `btn btn-secondary`, `bg-gray-200 text-gray-800 hover:bg-gray-300 px-4 py-2 rounded`)

If no clear pattern is found, leave both values as empty strings and note it in Phase 6 so the
user can fill them in manually.

Once you have answers, proceed to Phase 4.

---

## Phase 4: Make the code changes

Make all changes in this order:

### 4a. Add to INSTALLED_APPS

Edit the settings file — add `"gdpr_cookie_consent"` to `INSTALLED_APPS`. Place it near the end of your own apps, before third-party apps if there's a clear grouping.

```python
INSTALLED_APPS = [
    # ...existing apps...
    "gdpr_cookie_consent",
]
```

### 4b. Add URL configuration

Edit the root URL config (found via `ROOT_URLCONF`). Add:

```python
from django.urls import path, include

urlpatterns = [
    # ...existing patterns...
    path("cookies/", include("gdpr_cookie_consent.urls", namespace="cookie_consent")),
]
```

### 4c. Add widget to base template

Edit the base template. Make three additions:

**In `<head>`** — add the CSS (after existing `<link>` tags, or before `</head>`):
```html
<link href="{% static 'gdpr-cookie-consent/css/gdpr-cookie-consent.css' %}" rel="stylesheet" />
```

Make sure `{% load static %}` is already at the top of the template; add it if missing.

**Before `</body>`** — add the widget include:
```html
{% include "gdpr_cookie_consent/includes/cookie_consent.html" %}
```

**In the footer or navigation** — add the "Manage Cookies" link (ask the user where they want it,
or add a comment showing where it should go):
```html
{% url "cookie_consent:cookies_management" as cookie_management_url %}
<a href="{{ cookie_management_url }}">{% trans "Manage Cookies" %}</a>
```
(Add `{% load i18n %}` at the top if not already present.)

### 4d. Add COOKIE_CONSENT_SETTINGS to settings

Add the full configuration block. Build it based on the cookie categories the user selected.
Always include Essential. Add the other sections only if the user selected them.

Use this template, filling in the domain and adjusting sections:

```python
from django.utils.translation import gettext_lazy as _

COOKIE_CONSENT_SETTINGS = {
    "base_template_name": "base.html",  # ← use the value from Phase 3, question 3
    "description_template_name": "gdpr_cookie_consent/descriptions/what_are_cookies.html",
    "extra_information_template_name": "gdpr_cookie_consent/descriptions/extra.html",
    "dialog_position": "center",  # ← use value from Phase 3, question 2
    "consent_cookie_max_age": 60 * 60 * 24 * 30 * 6,  # 6 months
    "styling": {
        # ← Fill primary_button_css_classes with the primary button classes detected in Phase 3
        #   e.g. "btn btn-primary" or "bg-blue-600 text-white hover:bg-blue-700 px-4 py-2 rounded"
        "primary_button_css_classes": "",
        # ← Fill secondary_button_css_classes with the secondary button classes detected in Phase 3
        #   e.g. "btn btn-secondary" or "bg-gray-200 text-gray-800 hover:bg-gray-300 px-4 py-2 rounded"
        "secondary_button_css_classes": "",
    },
    "sections": [
        {
            "slug": "essential",
            "title": _("Essential Cookies"),
            "required": True,
            "summary": _(
                "These cookies are necessary for the website to function and cannot be disabled."
            ),
            "providers": [
                {
                    "title": _("This website"),
                    "description": _("Session and security cookies set by this website."),
                    "cookies": [
                        {
                            "cookie_name": "sessionid",
                            "duration": _("2 Weeks"),
                            "description": _("Maintains your login session."),
                            "domain": ".example.com",  # ← replace with actual domain
                        },
                        {
                            "cookie_name": "csrftoken",
                            "duration": _("1 Year"),
                            "description": _("Protects against cross-site request forgery attacks."),
                            "domain": ".example.com",
                        },
                        {
                            "cookie_name": "cookie_consent",
                            "duration": _("6 Months"),
                            "description": _("Stores your cookie consent preferences."),
                            "domain": ".example.com",
                        },
                    ],
                }
            ],
        },
        # Additional sections added below based on user selections
    ],
}
```

**Functionality section** (add if user selected it):
```python
{
    "slug": "functionality",
    "title": _("Functionality Cookies"),
    "preselected": True,
    "summary": _(
        "These cookies allow us to remember your preferences and personalise your experience."
    ),
    "conditional_html_template_name": "conditional_html/functionality.html",
    "providers": [
        {
            "title": _("This website"),
            "description": _("Cookies that remember your settings and preferences."),
            "cookies": [
                {
                    "cookie_name": "django_language",
                    "duration": _("1 Year"),
                    "description": _("Remembers your preferred language."),
                    "domain": ".example.com",
                },
            ],
        }
    ],
},
```

**Performance/Analytics section** (add if user selected it — adapt provider based on detected service):
```python
{
    "slug": "performance",
    "title": _("Performance Cookies"),
    "preselected": False,
    "summary": _(
        "These cookies help us understand how visitors use our website so we can improve it."
    ),
    "conditional_html_template_name": "conditional_html/performance.html",
    "providers": [
        {
            "title": _("Google Analytics"),
            "description": _(
                "Collects anonymous statistics about how visitors use the website."
            ),
            "cookies": [
                {
                    "cookie_name": "_ga",
                    "duration": _("2 Years"),
                    "description": _("Distinguishes unique users."),
                    "domain": ".example.com",
                },
                {
                    "cookie_name": "_ga_*",
                    "duration": _("2 Years"),
                    "description": _("Stores session state for Google Analytics 4."),
                    "domain": ".example.com",
                },
            ],
        }
    ],
},
```

**Marketing section** (add if user selected it):
```python
{
    "slug": "marketing",
    "title": _("Marketing Cookies"),
    "preselected": False,
    "summary": _(
        "These cookies are used to show you relevant advertising on other websites."
    ),
    "conditional_html_template_name": "conditional_html/marketing.html",
    "providers": [
        {
            "title": _("Google Ads"),
            "description": _("Used to track conversions and show relevant ads."),
            "cookies": [
                {
                    "cookie_name": "_gcl_au",
                    "duration": _("3 Months"),
                    "description": _("Stores and tracks conversions."),
                    "domain": ".example.com",
                },
            ],
        }
    ],
},
```

### 4e. Create conditional HTML templates

For each non-essential section selected, create the corresponding template. These templates contain
the actual tracking/service scripts that should only load when the user has consented.

Create `templates/conditional_html/` directory and add a file for each section:

**`templates/conditional_html/performance.html`** (if Google Analytics is in use):
```html
{% load i18n %}
<!-- Google Analytics - only loaded with performance cookie consent -->
<script async src="https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXXXX"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());
  gtag('config', 'G-XXXXXXXXXX');
</script>
```

Ask the user to fill in their actual tracking ID. For other analytics tools, adapt accordingly.

**`templates/conditional_html/functionality.html`** (if functionality section added):
```html
{# Scripts or inline code that should run when functionality cookies are accepted #}
{# e.g. language/timezone detection, chat widgets, etc. #}
```

**`templates/conditional_html/marketing.html`** (if marketing section added):
```html
{# Ad/tracking pixels — e.g. Facebook Pixel, Google Ads conversion #}
```

If the user's services are already embedded in their templates (detected in Phase 3),
suggest moving those scripts into the appropriate conditional template and removing them
from the base template. Show them exactly which lines to move.

---

## Phase 5: Run database migration and verify

```bash
# Apply the new migration (the package adds a ConsentLog model)
python manage.py migrate gdpr_cookie_consent

# Verify the configuration is valid
python manage.py check gdpr_cookie_consent
```

If `check` reports errors:
- **E001/E002**: `COOKIE_CONSENT_SETTINGS` not found or missing `base_template_name` — fix settings
- **E003**: A template path in settings doesn't exist — check the path
- **E005**: Invalid `dialog_position` — must be one of: center, top, left, right, bottom
- **E007/E008**: Section slug missing or not unique — fix the `sections` list
- **E009/E010**: Section has no provider, or provider has no cookies — add them, or silence with
  `SILENCED_SYSTEM_CHECKS = ["gdpr_cookie_consent.E009"]` in settings if intentionally empty

---

## Phase 6: Confirm and wrap up

Tell the user what was done:
1. ✅ Package added to requirements and/or installed
2. ✅ `gdpr_cookie_consent` added to `INSTALLED_APPS`
3. ✅ Cookie URL routes added to URL config
4. ✅ CSS and widget added to base template
5. ✅ `COOKIE_CONSENT_SETTINGS` added to settings
6. ✅ Conditional HTML templates created for: [list of sections]
7. ✅ Migration applied, configuration validated

Then mention any follow-up steps the user should handle manually:
- Replace placeholder domain (`.example.com`) with their actual domain throughout the settings
- If button CSS classes could not be auto-detected, fill in `primary_button_css_classes` and
  `secondary_button_css_classes` in the `styling` block of `COOKIE_CONSENT_SETTINGS` manually
- Fill in real tracking IDs in the conditional templates (e.g. `G-XXXXXXXXXX`)
- Add the "Manage Cookies" link to their footer if not already done
- (Optional) Customize button labels, dialog position, CSS variables, or translations
- (Optional) Add the context processor to `TEMPLATES` if they need template-level consent checks:
  ```python
  "gdpr_cookie_consent.context_processors.gdpr_cookie_consent"
  ```
- (Optional) Set up Google Consent Mode if using Google Analytics — see the package docs

---

## Notes and guardrails

- **Don't over-configure**: Generate only the sections the user actually needs. An empty
  functionality section is worse than no functionality section.
- **Don't move scripts without confirming**: If you find existing analytics scripts in templates,
  point them out and explain what moving them means before doing it.
- **settings split across files**: If there are separate base/local/production settings, add
  `COOKIE_CONSENT_SETTINGS` to the base settings file so it applies everywhere.
- **Multiple templates directories**: If the project has multiple `templates/` dirs, create
  the conditional templates in the app-level templates dir that is most appropriate, or ask.
- **Wheelhouse vs. direct install**: If the user already has the package installed in their
  virtualenv, skip the wheel/requirements.txt step — just proceed with the configuration.
