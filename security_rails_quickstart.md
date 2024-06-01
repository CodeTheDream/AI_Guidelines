## CTD Security Recommendations — Rails

### Intro

When building Rails apps, it's good to start off on the right foot in terms of security. Set and forget instead of neglect and regret. It's easy but developers are just not aware. The following are a few easy ways to boost your project's security. It can help protect your users and their sensitive data.

### Data Protection

###### Force SSL

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


###### CSP Headers

Prevent Cross-Site Scripting (XSS) attacks by implementing a Content Security Policy (CSP). Rails will send CSP headers to the browser and tell it to only trust certain sources. These will help mitigate the risk of XSS attacks. Specify trusted sources for scripts, styles, and other resources. Make sure you have a `csp_meta_tag` and you define the CSP settings in `config/content_security_policy.rb`.

```erb
<!-- app/views/layouts/application.html.erb -->
<%= csp_meta_tag %>
```

### Generic Messages

[OWASP recommends](https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets/Authentication_Cheat_Sheet.md#incorrect-and-correct-response-examples) responding to authentication/registration attempts in a generic manner. Examples below:

###### Login
Incorrect
```ruby
"Login for User foo: invalid password."
"Login failed, invalid user ID."
"Login failed; account disabled."
"Login failed; this user is not active."
```
Correct
```ruby
"Login failed; Invalid user ID or password."
```

###### Password recovery
Incorrect
```ruby
"We just sent you a password reset link."
"This email address doesn't exist in our database."
```
Correct
```ruby
"If that email address is in our database, we will send you an email to reset your password."
```

###### Account creation
Incorrect
```ruby
"This user ID is already in use."
"Welcome! You have signed up successfully."
```
Correct
```ruby
"A link to activate your account has been emailed to the address provided."
```

### Sessions

#### Devise

When using Devise for sessions, make sure to use its built in `:lockable` and `:timeoutable` modules. 

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

# Defines which key will be used when locking and unlocking an account
# config.unlock_keys = [:email]

# Defines which strategy will be used to unlock an account.
# :email = Sends an unlock link to the user email
# :time  = Re-enables login after a certain amount of time (see :unlock_in below)
# :both  = Enables both strategies
# :none  = No unlock strategy. You should handle unlocking by yourself.
config.unlock_strategy = :time

# Number of authentication tries before locking an account if lock_strategy
# is failed attempts.
config.maximum_attempts = 20

# Time interval to unlock the account if :time is enabled as unlock_strategy.
config.unlock_in = 1.hour

# Warn on the last attempt before the account is locked.
config.last_attempt_warning = true
```

[OWASP recommends](https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets/Session_Management_Cheat_Sheet.md#session-expiration) 2-5 minute idle timeout windows for high risk applications and 15-30 minute idle timeout windows for low risk applications. For absolute timeout, a range of 4-8 hours is acceptable if the application is intended to be used by an office worker all day.

#### Generic Session Advice

The following advice can be used whether you're using Devise or not.

###### Absolute Timeout

Implement an absolute session timeout so people are not logged on for to long in one session. This can prevent session from being stolen.

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

###### Password Strength

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

### Conclusion

By implementing these strategies, you can significantly enhance the security of your Rails project. Regularly review and update security practices to stay ahead of emerging threats.

Remember, security is an ongoing process, and staying vigilant is key to protecting your application and user data.

### Resources

See these PRs for examples
- [15 Min Idle Timeout](https://github.com/CodeTheDream/nc_fair_chance/pull/89)
- [Absolute Timeout](https://github.com/CodeTheDream/nc_fair_chance/pull/88)
- [Max Login Attempts](https://github.com/CodeTheDream/nc_fair_chance/pull/87)
- [Password Strength](https://github.com/CodeTheDream/nc_fair_chance/pull/86)
