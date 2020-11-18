# authenticate-in-rails

### Gemas y sesiones

```ruby
gem 'devise'
gem 'omniauth-facebook'
gem 'omniauth-github'
gem 'omniauth-linkedin-oauth2'
gem 'omniauth-twitter'
gem 'activerecord-session_store'`
```

`bundle install`

La última gema es para gestionar las sesiones y no tener que crear diferentes campos para manejar la integración de varios métodos de autenticación de terceros. 

Por terminal debemos crear el archivo para configurar el manejo de sesiones y algunas cosas más como configurar e instalar devise

```
rails generate devise:install
rails generate devise User
rails generate devise:views
rails generate active_record:session_migration
```

Crear el archivo **config/initializers/session_store.rb** y agregar en el
Rails.application.config.session_store :active_record_store

`rake db:migrate`

### Devise
Editamos el archivo **config/initializers/devise.rb** (Ojo con el callback en la integración de fb)
```ruby
config.omniauth :facebook, ENV['FB_ID'], ENV['FB_KEY'], callback_url: "http://localhost:3000/users/auth/facebook/callback"
config.omniauth :github, ENV['GITHUB_KEY'], ENV['GITHUB_SECRET'], scope: 'user:email'
config.omniauth :linkedin, ENV['LINKEDIN_ID'], ENV['LINKEDIN_SECRET']
config.omniauth :twitter, ENV['TWITTER_KEY'], ENV['TWITTER_SECRET']
```

### Modelo User
En el modelo **user.rb** debemos indicar que usaremos como proveedores de omniauth
```ruby
devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :validatable,
         :omniauthable, :omniauth_providers => [:linkedin, :google_oauth2, :github, :facebook]
```

Además agregamos el metodo en el que crearemos o solo autenticaremos al usuario en base a su email
```ruby
def self.from_omniauth(auth)
	where(email: auth.info.email).first_or_create do |user|
		user.email = auth.info.email
		user.password = Devise.friendly_token[0,10]
	end
end
```

### Rutas
Debemos actualizar las rutas para que los respectivos callback de cada integración lleguen al mismo método, por lo que dejamos las rutas de usuario de esta manera
```ruby
devise_for :users, controllers: {
	omniauth_callbacks: 'users/omniauth_callbacks'
}
```

### Callback Controller
Por último creamos el controlador correspondiente en **/app/controllers/users/omniauth_callbacks_controller.rb** (el codigo esta en el repo) donde especificamos para cada integración un método (podrían ser todos el mismo, pero asi es mas claro en un comienzo).

Eso es todo!
