# Mail Interceptor

[![Circle CI](https://circleci.com/gh/bigbinary/mail_interceptor.svg?style=svg)](https://circleci.com/gh/bigbinary/mail_interceptor)


## TOC

* [About](#about)
* [Usage](#usage)
* [deliver_emails_to](#deliver_emails_to)
* [forward_emails_to](#forward_emails_to)
* [ignore_bcc and ignore_cc](#ignore_bcc-and-ignore_cc)
* [Custom Environment](#custom-environment)
* [Prefixing email with subject](#prefixing-email-with-subject)

## About

This gem intercepts and forwards email to a forwarding address in
a non-production environment. This is to ensure that in staging or 
in development by mistake we do not deliver emails to the real people. 
However we need to test emails time to time. 

Refer to https://github.com/bigbinary/wheel/blob/master/config/initializers/email_interceptor.rb
and you will notice that if an email ends with `deliver@bigbinary.com` then that emaill will be delivered.

So if Neeraj wants to test how the real email looks then he can use email `neeraj+deliver@bigbinary.com` and that email will be delivered. 
As long as an email ends with `deliver@bigbinary.com` then that email will not be intercepted.

If client wants to test something and the client expects an email to be delivered then we need to add client's email here. 
Say the client's email is `michael@timbaktu.com`. 
Change that line to deliver_emails_to: `["deliver@bigbinary.com", "timbaktu.com"]`. 
Now all emails ending with `timbaktu.com` would be delivered. 
If you want only Michael should get email and other emails ending with "timbaktu.com" to be intercepted then change that line to deliver_emails_to: `["deliver@bigbinary.com", "michael@timbaktu.com"]`.

## Usage

```ruby
# There is no need to include this gem for production or for test environment
gem 'mail_interceptor', group: [:development, :staging]
```

```ruby
# config/initializers/mail_interceptor.rb

options = { forward_emails_to: 'intercepted_emails@domain.com',
            deliver_emails_to: ["@wheel.com"] }

unless (Rails.env.test? || Rails.env.production?)
  interceptor = MailInterceptor::Interceptor.new(options)
  ActionMailer::Base.register_interceptor(interceptor)
end
```

Do not use this feature in test mode so that in tests
you can test against provided recipients of the email.

## deliver_emails_to

Passing __deliver_emails_to__ is optional. If no "deliver_emails_to"
is passed then all emails will be intercepted and forwarded in
non-production environment.

Let's say that you want to actually deliver all emails having the pattern
"@BigBinary.com". Here is how it can be accomplished.

```ruby
MailInterceptor::Interceptor.new({ forward_emails_to: 'intercepted_emails@domain.com',
                                   deliver_emails_to: ["@bigbinary.com"] })
```

If you want the emails to be delivered only if the email address is
`qa@bigbinary.com` then that can be done too.

```ruby
MailInterceptor::Interceptor.new({ forward_emails_to: 'intercepted_emails@domain.com',
                                   deliver_emails_to: ["qa@bigbinary.com"] })
```

Now only `qa@bigbinary.com` will get its emails delivered and all other emails
will be intercepted and forwarded.

The regular expression is matched without case sensitive. So you can mix lowercase
and uppercase and it won't matter.

## forward_emails_to

This is a required field.

It can take a single email or an array of emails.

```ruby
MailInterceptor::Interceptor.new({ forward_emails_to: 'intercepted_emails@bigbinary.com' })
```

It can also take an array of emails in which case emails are forwarded to each of those emails in the array.

```ruby
MailInterceptor::Interceptor.new({ forward_emails_to: ['intercepted_emails@bigbinary.com',
                                                       'qa@bigbinary.com' })
```

## ignore_bcc and ignore_cc

By default bcc and cc are ignored.
You can pass `:ignore_bcc` or `:ignore_cc` options as `false`,
if you don't want to ignore bcc or cc.

## Custom Environment

By default all emails sent in non production environment are
intercepted. However you can control this behavior by passing `env` as
the key. It accepts any ruby objects which responds to `intercept?`
method. If the result of that method is `true` then emails are
intercepted otherwise emails are not intercepted.

Below is an example of how to pass a custom ruby object as value for
`env` key.

Besides method `intercept?` method `name` is needed if you have provided
`subject_prefix`. This name will be appended to the `subject_prefix` to
produce something like `[WHEEL STAGING] Forgot password`. In this case
`STAGING` came form `name`.

```ruby
class MyEnv
  def name
    ENV["ENVIRONMENT_NAME"]
  end

  def intercept?
    ENV["INTERCEPT_MAIL"] == '1'
  end
end

MailInterceptor::Interceptor.new({ env: MyEnv.new,
                                   forward_emails_to: ['intercepted_emails@bigbinary.com',
                                   'qa@bigbinary.com' })
```

## Prefixing email with subject

If you are looking for automatically prefix all delivered emails with the application name and Rails environment
then we recommend using [email_prefixer gem](https://github.com/wireframe/email_prefixer) .

## Brought to you by
<a href='http://BigBinary.com'><img src="https://raw.githubusercontent.com/bigbinary/bigbinary-assets/press-assets/PNG/logo-light-solid-small.png?raw=true" width="200px"/></a>
