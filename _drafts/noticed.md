# Day 4 - Noticed gem - Notifications for your Ruby on Rails app

Written by [Chris Oliver](https://twitter.com/excid3) from [GoRails](https://gorails.com/) {% avatar excid3 %}

## Notifications are complex

When you hear the word "notifications" a lot of different things can come to mind. You might be thinking about:

* Notifications in the navbar
* Email notifications
* Slack notifications
* Text Message / SMS notifications
* etc

But really, these are delivery methods for an event that happened in your app. 

Let's say a user comments on a Post you made in an app. We definitely want to store the event in the database to display in the navbar. We can keep track if the event was read or not by storing that in the database. The user will be able to see this immediately if they're using our website right now. 

What if they aren't on our site? We can send them a notification through another channel like Email, SMS, or Slack. Each of these require different formats, options, and data. Slack needs a channel and some JSON while email needs an address and the HTML content of the email. Plus, users have preferences so they may not want to receive SMS or Email notifications.

## Creating Notifications with Noticed

Noticed is designed to organize all this. We can define a class for the notification and all the different ways it could be delivered.

```ruby
class CommentNotification < Noticed::Base
  deliver_by :database
  deliver_by :action_cable
  deliver_by :email, mailer: 'CommentMailer', delay: 5.minutes, if: :email_notifications?

  # I18n helpers
  def message
    t(".message")
  end

  # URL helpers are accessible in notifications
  # Don't forget to set your default_url_options so Rails knows how to generate urls
  def url
    post_path(params[:post])
  end

  def email_notifications?
    !!recipient.preferences[:email]
  end
end
```

This example shows how we could define a notification when a new comment is posted. We have it write to the database and broadcast to the browser immediately with ActionCable.

After 5 minutes, we'll deliver by email if the user has enabled email notifications.

We can also use this class to handle URLs and internationalization. For example, the notification is about a comment, but we want to link to the Post that the comment replied to in our navbar or emails.

To send the notifcation, we simply create it and tell it who to deliver to:

```ruby
CommentNotification.with(comment: @comment).deliver_later(@comment.post.author)
```

Noticed works just like ActionMailer so you can pass params and decide to deliver the notification immediately or later (in a background job).

## Find Out More

### References

- Home :: [github.com/excid3/noticed](https://github.com/excid3/noticed)
- Gem :: [rubygems.org/gems/noticed](https://rubygems.org/gems/noticed)
- Screencast :: [How to add Notifications with Noticed](https://gorails.com/episodes/rails-notifications-with-noticed?autoplay=1)

