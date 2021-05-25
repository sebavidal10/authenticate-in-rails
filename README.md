# authenticate-in-rails

### Gems and sessions

```ruby
gem 'devise'
gem 'omniauth-facebook'
gem 'omniauth-github'
gem 'omniauth-linkedin-oauth2'
gem 'omniauth-twitter'
gem 'activerecord-session_store'`
```

`bundle install`

The last gem is to manage sessions and not have to create different fields to handle the integration of various third-party authentication methods.

By terminal we must create the file to configure session management and some more things like configure and install devise

```
rails generate devise:install
rails generate devise User
rails generate devise:views
rails generate active_record:session_migration
```

Create the file **config/initializers/session_store.rb** and add Rails.application.config.session_store :active_record_store

`rake db:migrate`

### Devise
Edit the file **config/initializers/devise.rb** (Be careful with the callback in fb integration)
```ruby
config.omniauth :facebook, ENV['FB_ID'], ENV['FB_KEY'], callback_url: "http://localhost:3000/users/auth/facebook/callback"
config.omniauth :github, ENV['GITHUB_KEY'], ENV['GITHUB_SECRET'], scope: 'user:email'
config.omniauth :linkedin, ENV['LINKEDIN_ID'], ENV['LINKEDIN_SECRET']
config.omniauth :twitter, ENV['TWITTER_KEY'], ENV['TWITTER_SECRET']
```

### User Model
In User Model **user.rb** we must indicate that we will use omniauth as providers
```ruby
devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :validatable,
         :omniauthable, :omniauth_providers => [:linkedin, :google_oauth2, :github, :facebook]
```

We also add the method in which we will create or only authenticate the user based on their email
```ruby
def self.from_omniauth(auth)
	where(email: auth.info.email).first_or_create do |user|
		user.email = auth.info.email
		user.password = Devise.friendly_token[0,10]
	end
end
```

### Routes
We must update the routes so that the respective callbacks of each integration reach the same method, so we leave the user routes in this way
```ruby
devise_for :users, controllers: {
	omniauth_callbacks: 'users/omniauth_callbacks'
}
```

### Callback Controller
Finally we create the corresponding controller in **/app/controllers/users/omniauth_callbacks_controller.rb** (the code is in the repo) where we specify a method for each integration (they could all be the same, but it is clearer in the beginning).

That's all!
