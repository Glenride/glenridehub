This PRD is designed to be fed into an LLM (like a specialized coding agent) to generate the boilerplate for your Hub and Sub-apps. It follows the **Multi-Repo Strategy**—the most compatible path for Laravel Cloud.

---

# PRD: Unified Brand Hub & Auth Ecosystem

## 1. Project Overview

Build a centralized "Identity Provider" (The Hub) and a "Service Provider" (Sub-app template) that allows for Single Sign-On (SSO) across multiple subdomains (e.g., `*.yourbrand.com`) on Laravel Cloud.

## 2. Technical Stack

* **Framework:** Laravel 11+
* **Infrastructure:** Laravel Cloud (Growth Plan for Wildcard SSL)
* **Identity Layer:** Laravel Passport (OAuth2)
* **Client Layer:** Laravel Socialite (Custom Provider)
* **Shared UI:** Tailwind CSS + Blade Components

---

## 3. Architecture: The "Hub & Spoke" Model

### The Hub (`accounts.yourbrand.com`)

* **Role:** The "Source of Truth" for User records and Billing.
* **Key Feature:** Acts as an OAuth2 Server.
* **Data:** Stores `email`, `password`, `avatar_url`, and `subscription_status`.

### The Sub-Apps (`app1.yourbrand.com`, `app2....`)

* **Role:** Specialized tools (Writing, Billing, Analytics).
* **Key Feature:** "Login with Brand" button only. No local passwords.
* **Data:** Stores `hub_user_id` and app-specific data (e.g., `posts`, `settings`).

---

## 4. Implementation Steps & Configurations

### Step A: Configure the Hub (The Server)

**Goal:** Transform a fresh Laravel app into an Identity Provider.

1. **Install Passport:** `composer require laravel/passport`.
2. **App Config:** In `AuthServiceProvider`, call `Passport::routes()`.
3. **Client Creation:** Run `php artisan passport:client --public`. This generates the `client_id` and `secret` you will give to your sub-apps.

### Step B: Configure the Sub-Apps (The Clients)

**Goal:** Allow users to log in using their Hub account.

1. **Install Socialite:** `composer require laravel/socialite`.
2. **Custom Driver Logic:** Add the following to your `EventServiceProvider` or a dedicated Service Provider to recognize the Hub:

```php
// config/services.php
'hub' => [
    'client_id' => env('HUB_CLIENT_ID'),
    'client_secret' => env('HUB_CLIENT_SECRET'),
    'redirect' => env('HUB_REDIRECT_URI'),
    'base_url' => 'https://accounts.yourbrand.com',
],

```

### Step C: The Shared Branding (UI Kit)

**Goal:** Ensure visual consistency across repos.

1. **Shared Tailwind Preset:** Create a `tailwind.config.js` in a shared Git repo.
2. **Sub-app Integration:**

```javascript
// tailwind.config.js in Sub-App
module.exports = {
  presets: [
    require('./vendor/yourbrand/brand-toolkit/tailwind.preset.js')
  ],
  // ...
}

```

---

## 5. Deployment Checklist (Laravel Cloud)

* **Environment Variables:** Every sub-app needs `HUB_CLIENT_ID` and `HUB_CLIENT_SECRET`.
* **Wildcard Domain:** Add `*.yourbrand.com` to your Laravel Cloud project.
* **Session Sharing (Optional):** Set `SESSION_DOMAIN=.yourbrand.com` in all `.env` files if you want "Zero-Click" transitions between apps.

---

## 6. Prompt for your Coding LLM

> "I am building a Laravel ecosystem on Laravel Cloud using a Multi-Repo strategy.
> 1. Please write a **Custom Socialite Provider** for my Sub-Apps that authenticates against my own Laravel Passport server located at `accounts.yourbrand.com`.
> 2. Include the **Controller logic** for the callback that checks if a user exists by `hub_user_id`. If they exist, log them in; if not, create a new user record without a password.
> 3. Provide a **Middleware** example that redirects unauthenticated users on the Sub-App directly to the Hub's OAuth authorization page."
> 
> 
