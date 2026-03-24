# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Nova Ledger is a Django REST backend application (no frontend). The project is in its initial phase: setting up the Django project with a custom authentication system.

## Tech Stack

- Python 3.12+
- Django 5.x
- django-allauth (for social auth providers)
- Django REST Framework (API layer)
- SQLite for development, PostgreSQL for production

## Architecture Decisions

### Custom User Model

- The `User` model uses **email as the unique identifier** — there is no username field at all.
- `USERNAME_FIELD = 'email'`, no `username` field on the model.
- The user model is intentionally minimal for now (email + password). It will be extended later.
- Custom user model lives in a dedicated `accounts` app.

### Authentication

Three authentication methods, all managed via django-allauth:

1. **Email + password** (standard login/signup)
2. **Google OAuth** (Gmail) via allauth provider
3. **Microsoft OAuth** (Outlook) via allauth provider

No other social providers. Do not add any.

### OAuth Scopes

During Google and Microsoft OAuth login, request **email read permissions** in addition to standard profile/email scopes:

- **Google**: request `https://www.googleapis.com/auth/gmail.readonly` scope
- **Microsoft**: request `Mail.Read` scope (Microsoft Graph API)

These scopes are needed for a future feature that reads emails from connected accounts. Store the OAuth tokens (access + refresh) so they can be reused later for email access.

## Commands

```bash
# Create virtual environment and install dependencies
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Run migrations
python manage.py migrate

# Run development server
python manage.py runserver

# Create superuser
python manage.py createsuperuser

# Run tests
python manage.py test

# Run a single test module
python manage.py test accounts.tests.test_auth

# Run a specific test class or method
python manage.py test accounts.tests.test_auth.LoginTestCase.test_login_with_email
```

## Project Structure (Target)

```
nova-ledger/
├── config/              # Django project settings package
│   ├── settings/
│   │   ├── base.py      # Shared settings
│   │   ├── dev.py       # Development overrides
│   │   └── prod.py      # Production overrides
│   ├── urls.py
│   └── wsgi.py
├── accounts/            # Custom user model + auth logic
│   ├── models.py        # Custom User (email-based, no username)
│   ├── managers.py      # Custom UserManager
│   ├── adapters.py      # allauth account adapter + social adapter
│   ├── serializers.py
│   └── views.py
├── requirements.txt
└── manage.py
```

## Key Configuration Notes

- `AUTH_USER_MODEL = 'accounts.User'` — must be set before first migration.
- allauth config: `ACCOUNT_USER_MODEL_USERNAME_FIELD = None`, `ACCOUNT_EMAIL_REQUIRED = True`, `ACCOUNT_USERNAME_REQUIRED = False`, `ACCOUNT_AUTHENTICATION_METHOD = 'email'`.
- Social app credentials (Google client ID/secret, Microsoft client ID/secret) are loaded from environment variables, never hardcoded.
- `SOCIALACCOUNT_STORE_TOKENS = True` to persist OAuth tokens for later email API access.

## Language

The codebase, comments, commit messages, and documentation are in **English**. The developer speaks French.
