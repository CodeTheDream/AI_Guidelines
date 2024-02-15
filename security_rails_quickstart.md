## CTD Security Recommendations — Rails

### Intro

When building Rails apps, it's good to start off on the right foot in terms of security. Set and forget instead of neglect and regret. It's easy but developers are just not aware. The following are a few easy ways to boost your project's security. It can help protect your users and their sensitive data.

### Force SSL

SSL should be enforced in production.

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

### Authentication & Password Reset

[OWASP recommends](https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets/Authentication_Cheat_Sheet.md#incorrect-and-correct-response-examples) responding to login attempts in a generic manner. Examples below:


###### Login

Incorrect response examples:

- "Login for User foo: invalid password."
- "Login failed, invalid user ID."
- "Login failed; account disabled."
- "Login failed; this user is not active."

Correct response example:

- "Login failed; Invalid user ID or password."

###### Password recovery

Incorrect response examples:

- "We just sent you a password reset link."
- "This email address doesn't exist in our database."

Correct response example:

- "If that email address is in our database, we will send you an email to reset your password."

###### Account creation

Incorrect response examples:

- "This user ID is already in use."
- "Welcome! You have signed up successfully."

Correct response example:

- "A link to activate your account has been emailed to the address provided."


### Devise

If you are using Devise, make sure to use its built in `:lockable` and `:timeoutable` modules. 

```ruby
class User
  # ...
  devise :lockable, :timeoutable
  # ...
end
```


This helps increase security strength by preventing brute force attacks and session hijacking. 

You will need to uncomment the corresponding sections in migrations created by Devise

```ruby
## Lockable
t.integer  :failed_attempts, default: 0, null: false # Only if lock strategy is :failed_attempts
t.string   :unlock_token # Only if unlock strategy is :email or :both
t.datetime :locked_at
```

You will also need to enable the correct configurations in `/config/initializers/devise.rb`

```ruby
# ==> Configuration for :timeoutable
# The time you want to timeout the user session without activity. After this
# time the user will be asked for credentials again. Default is 30 minutes.
config.timeout_in = 30.minutes

# ==> Configuration for :lockable
# Defines which strategy will be used to lock an account.
# :failed_attempts = Locks an account after a number of failed attempts to sign in.
# :none            = No lock strategy. You should handle locking by yourself.
config.lock_strategy = :failed_attempts

# and much more below
```

[OWASP recommends](https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets/Session_Management_Cheat_Sheet.md#session-expiration) 2-5 minute idle timeout windows for high risk applications and 15-30 minute idle timeout windows for low risk applications. For absolute timeout, a range of 4-8 hours is acceptable if the application is intended to be used by an office worker all day.

### CSP Headers

Prevent Cross-Site Scripting (XSS) attacks by implementing a Content Security Policy (CSP). Rails will send CSP headers to the browser and tell it to only trust certain sources. These will help mitigate the risk of XSS attacks. Specify trusted sources for scripts, styles, and other resources. Make sure you have a `csp_meta_tag` and you define the CSP settings in `config/content_security_policy.rb`.

```erb
<!-- app/views/layouts/application.html.erb -->
<%= csp_meta_tag %>
```

### Conclusion

By implementing these strategies, you can significantly enhance the security of your Rails project. Regularly review and update security practices to stay ahead of emerging threats.

Remember, security is an ongoing process, and staying vigilant is key to protecting your application and user data.
