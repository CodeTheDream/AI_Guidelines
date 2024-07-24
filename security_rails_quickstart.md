# CTD Security Recommendations — Rails

## Intro

When building Rails apps, prioritizing security from the start can prevent problems later on. Set and forget instead of neglect and regret. It's easy but developers lack awareness. This guide outlines a few easy ways to boost your project's security. It can help protect your users and their sensitive data.

## Visible Security

First, let's go over security that is easy to see.

### Generic Messages

[OWASP recommends](https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets/Authentication_Cheat_Sheet.md#incorrect-and-correct-response-examples) responding to signing in/up in a generic way. Examples below:

<details><summary><code>Sign In</code></summary><br>

```ruby
# incorrect
"Login for User foo: invalid password."
"Login failed, invalid user ID."
"Login failed; account disabled."
"Login failed; this user is not active."

# correct
"Login failed; Invalid user ID or password."
```
</details>

<details><summary><code>Sign Up</code></summary><br>

```ruby
# incorrect
"This user ID is already in use."
"Welcome! You have signed up successfully."

# correct
"A link to activate your account has been emailed to the address provided."
```
</details>

<details><summary><code>Password Recovery</code></summary><br>

```ruby
# incorrect
"We just sent you a password reset link."
"This email address doesn't exist in our database."

# correct
"If that email address is in our database, we will send you an email to reset your password."
```
</details>

### Sessions

There are many ways to improve security around sessions. One popular session solution is Devise.

#### Devise

<details><summary><code>Devise Modules</code></summary><br>

When using Devise for sessions, make sure to use its built in `:lockable` and `:timeoutable` modules. They help increase security by protecting against brute force attacks and session hijacking.

```ruby
class User
  # ...
  devise :lockable, :timeoutable
  # ...
end
```
</details>

<details><summary><code>Devise Migrations</code></summary>
<br>

You will need to uncomment the corresponding sections in migrations created by Devise

```ruby
## Lockable
t.integer  :failed_attempts, default: 0, null: false # Only if lock strategy is :failed_attempts
t.string   :unlock_token # Only if unlock strategy is :email or :both
t.datetime :locked_at
```
</details>

<details><summary><code>Devise Configurations</code></summary>
<br>

You will also need to edit your Devise config at `/config/initializers/devise.rb`

```ruby
# ==> Configuration for :timeoutable
config.timeout_in = 30.minutes

# ==> Configuration for :lockable
config.lock_strategy = :failed_attempts

...

# Warn on the last attempt before the account is locked.
config.last_attempt_warning = true
```
</details>

<details><summary><code>Recommended Session Timeout</code></summary><br>

[OWASP recommends](https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets/Session_Management_Cheat_Sheet.md#session-expiration) 2-5 minute idle timeout windows for high risk applications and 15-30 minute idle timeout windows for low risk applications. For absolute timeout, a range of 4-8 hours is acceptable if the application is intended to be used by an office worker all day.
</details>

#### No Devise

The following advice can be used whether you're using Devise or not.

<details><summary><code>Absolute Timeout</code></summary><br>

Implement an absolute session timeout so people are not logged on for too long in one session. This can prevent session from being stolen.

```ruby
class ApplicationController < ActionController::Base
  # ...
  before_action :check_if_session_is_valid
  before_action :update_session
  # ...
  def check_if_session_is_valid
    session_timeout = 1
    if session[:timestamp] <= session_timeout.hours.ago.to_i
      session["warden.user.admin.key"] = nil
    end
  end

  def update_session
    session[:timestamp] = Time.now.to_i
  end
end
```
</details>

<details><summary><code>Password Strength</code></summary><br>

Implement strong password requirements to reduce account hacking.

```ruby
model User
  # ...
  validates :password, presence: true, on: [ :create]
  validates :password, length: { in: 6..128 }, on: [:update, :create]
  validates :password, format: { with: /[a-z]+/, message: 'should have at least 1 lower case letter' }, on: [:update, :create]
  validates :password, format: { with: /[A-Z]+/, message: 'should have at least 1 upper case letter' }, on: [:update, :create]
  validates :password, format: { with: /\d+/, message: 'should have at least 1 digit' }, on: [:update, :create]
  validates :password, format: { with: /\W+/, message: 'should have at least 1 special character' }, on: [:update, :create]
  # ...
end
```

Also make sure to *always* encrypt passwords. Devise already encrypts passwords with [bcrypt](https://github.com/bcrypt-ruby/bcrypt-ruby).
</details>

## Invisible Security

Next let's go over security in ways that are harder to see.

<!-- While you touch on some important aspects like SSL and CSP headers, consider expanding on other "invisible" security measures such as database security (e.g., SQL injection prevention), data validation, and secure deployment practices. -->

<details><summary><code>SSL</code></summary><br>

One easy way to boost security is by forcing your website to use the Secure Sockets Layer (SSL).

```ruby
# /config/environments/production.rb
# ...
config.force_ssl = true
# ...
```

This increases security in many ways.

- **Redirect to HTTPS**: Rails will automatically redirect HTTP requests to equivalent HTTPS requests. HTTPS requests use encryption, making them more secure.
- **Use Secure Cookies**: Rails will set the `Secure` flag on session cookies. This ensures the session ID will not be transmitted over HTTPS.
- **Strict Transport Security (HSTS)**: The `Strict-Transport-Security` header is included in responses, instructing the browser to always use HTTPS for future requests to the same domain.
</details>

### Headers

Headers are a part of every webpage you visit. They come in key-value pairs like a hash. Headers are a set of metadata that hold website information about content type, security policies, cookies, and more. 

<details><summary><code>Document Headers</code></summary>

#### Recommended Headers

Below is a short list of headers that satisfy CTD's Security Standards. For a comprehensive list, see OWASP's [Secure Headers Project](https://owasp.org/www-project-secure-headers/index.html#div-bestpractices_configuration-proposal).

|Header Name|Proposed Value|
|-|-|
|Content-Security-Policy|`default-src 'self'; form-action 'self'; object-src 'none'; frame-ancestors 'none'; upgrade-insecure-requests; block-all-mixed-content`
|Strict-Transport-Security|`max-age=SECONDS; includeSubDomains`
|X-Content-Type-Options|`nosniff`
|X-XSS-Protection|`0`

#### Recommended Headers Explanation

- See the `Content-Security-Policy` section of the [OWASP Secure Headers Project](https://owasp.org/www-project-secure-headers/index.html#div-headers) page for more details on this header.
- `Strict-Transport-Security` allows web servers to declare that web browsers should only interact with them using HTTPS. This header is ignored if the page is accessed over HTTP. `includeSubDomains` tells the browser to also only interact with subdomains over HTTPS.
- `nosniff` will help prevent attacks based on MIME-type confusion. Without this header, the browser might guess the type of file it is with the file extension or contents and start rendering it in a way it's not supposed to. This can lead to XSS attacks. A text file could be rendered as a JavaScript file and the browser starts executing foreign JavaScript.
- `X-XSS-Protection` has been [deprecated](https://owasp.org/www-project-secure-headers/index.html#x-xss-protection). Use `X-XSS-Protection: 0` to disable. Please use `Content-Security-Policy` instead.

#### How to Check Your Website's Headers

You can check your website's response headers in the browser console / Network / Your File / Headers / Response Headers. If nothing shows up, reload the page and click your page's file.

<img width="1440" alt="Screen Shot 2024-07-23 at 3 36 40 PM" src="https://github.com/user-attachments/assets/62e29d44-a800-49b5-8067-0240227a2192">

</details>

<details><summary><code>Cookie Headers</code></summary>

#### Recommended Cookie Headers

Cookies also have headers! Make sure your cookie headers satisfy the requirements below, especially `SameSite` because it helps prevent CSRF attacks.

|Header Name|Proposed Value|
|-|:-:|
|HttpOnly|✅
|Secure|✅
|SameSite|`Strict` or `Lax`

#### How to Check Your Website's Cookie Headers

You can check your cookie headers by pulling up the browser console / Application / Storage / Cookies.

<img width="1440" alt="Screen Shot 2024-07-23 at 3 36 51 PM" src="https://github.com/user-attachments/assets/243620fa-8ae4-4560-99c2-9e9ad2bf4a86">

Notice how Github's `_gh_sess` cookie has the `HttpOnly` and `Secure` flags checked and `SameSite` is set to `Lax`.
</details>

<details><summary><code>CSP Headers</code></summary><br>

Although Content Security Policy (CSP) Headers are only one part of the document's response headers, they are crucial for web security. 
Rails will send CSP headers to the browser and tell it to only trust certain sources. They help mitigate the risk of XSS attacks by specifying trusted sources for scripts, styles, and other resources.

#### How to Enable CSP Headers

Make sure you have a `csp_meta_tag` and you enable CSP settings that work for your app in `config/content_security_policy.rb`.

```erb
<!-- app/views/layouts/application.html.erb -->
<%= csp_meta_tag %>
```
</details>

## Conclusion

By following this guide, you can significantly strengthen the security of your Rails project. 

Remember, security is an ongoing process, and staying vigilant is key to protecting your application and user data.

## Resources

See these PRs for examples (will need access to NCFC repo to see examples)
- [15 Min Idle Timeout](https://github.com/CodeTheDream/nc_fair_chance/pull/89)
- [Absolute Timeout](https://github.com/CodeTheDream/nc_fair_chance/pull/88) (Example already in quickstart guide)
- [Max Login Attempts](https://github.com/CodeTheDream/nc_fair_chance/pull/87)
- [Password Strength](https://github.com/CodeTheDream/nc_fair_chance/pull/86) (Example already in quickstart guide)
