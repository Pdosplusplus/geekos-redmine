# How to install Redmine on Heroku


## Install packages to build redmine in Debian Jessie

```bash
$ sudo apt install git gcc build-essential zlib1g zlib1g-dev zlibc ruby-zip libssl-dev libyaml-dev libcurl4-openssl-dev ruby gem libapr1-dev libxslt1-dev checkinstall libxml2-dev ruby-dev vim libmagickwand-dev imagemagick libsqlite3-dev libpq-dev
```

## Now clone ```Redmine```

```bash
$ git clone https://github.com/redmine/redmine.git -b 2.6-stable
```

And position inside

```bash
$ cd redmine
```

## Remove those files from  ```.gitignore```

```bash
Gemfile.lock
Gemfile.local
public/plugin_assets 
config/initializers/session_store.rb 
config/initializers/secret_token.rb 
config/configuration.yml 
config/email.yml
```

## Remove those block from  the file ```Gemfile```

```ruby
platforms :mri, :mingw do
  # Optional gem for exporting the gantt to a PNG file, not supported with jruby
  group :rmagick do
    # RMagick 2 supports ruby 1.9
    # RMagick 1 would be fine for ruby 1.8 but Bundler does not support
    # different requirements for the same gem on different platforms
    gem "rmagick", (RUBY_VERSION < "1.9" ? "2.13.3" : ">= 2.0.0")
  end

  # Optional Markdown support, not for JRuby
  group :markdown do
    # TODO: upgrade to redcarpet 3.x when ruby1.8 support is dropped
    gem "redcarpet", "~> 2.3.0"
  end
end

platforms :jruby do
  # jruby-openssl is bundled with JRuby 1.7.0
  gem "jruby-openssl" if Object.const_defined?(:JRUBY_VERSION) && JRUBY_VERSION < '1.7.0'
  gem "activerecord-jdbc-adapter", "~> 1.3.2"
end

# and
database_file = File.join(File.dirname(__FILE__), "config/database.yml") if File.exist?(database_file)
  database_config = YAML::load(ERB.new(IO.read(database_file)).result)
    ...
    else
  warn("No adapter found in config/database.yml, please configure it first") end
    else
warn("Please configure your config/database.yml first") end
```

## In the same file, replace above block with this

```ruby
group :development, :test do
  gem 'sqlite3'
end

group :production do
  gem 'pg'
  gem 'rails_12factor'
  gem 'thin' # change this if you want to use other rack web server
end
```

## Install bundler the Packet handler of ruby

```bash
$ gem install bundler
```

## Install the dependencies to run redmine

bundle it without production

```bash
$ bundle install --without production test
``` 

## Generate secret token for redmine

```bash
$ rake generate_secret_token
```

# To avoid aborting when deploying to heroku, we have to do the following two steps: In ```config/environment.rb``` we have to remove (or comment) line 10, where it says

```ruby
exit 1
# remove it or comment it
# exit 1
```

## Add this under in the file ```config/application.rb```

```ruby
...
module RedmineApp
  classApplication<Rails::Application
    config.assets.initialize_on_precompile = false # add this line
...
```

## Now Configure E-mail SMTP with Gmail

Then you do create the following file for the email configurations: ```config/configuration.yml```

````yaml
production:
  delivery_method: :smtp
  smtp_settings:
    enable_starttls_auto: true
    address: "smtp.gmail.com" 
    port: 587
    domain: "smtp.gmail.com" 
    authentication: :plain
    user_name: "user-name@gmail.com" 
    password: "your-password" 
```

# Now Install heroku

First make sure you have it installed

```bash
$ sudo apt install curl apt-transport-https
```

Run the following to add our apt repository and install the CLI:

```bash
$ sudo add-apt-repository "deb https://cli-assets.heroku.com/branches/stable/apt ./"
$ curl -L https://cli-assets.heroku.com/apt/release.key | sudo apt-key add -
$ sudo apt-get update
$ sudo apt-get install heroku
```

## Create heroku ```app```

```
$ heroku create APP_NAME
```

## We uploaded the data to heroku

```bash
$ git add -A
$ git commit -m "preparing for heroku" 
$ git push heroku 2.6-stable:master
```

## Create the database in heroku

```
$ heroku run rake db:migrate
```

## Load the default data 

```
$ heroku run rake redmine:load_default_data
```

And finnish To check the installation execute:

```bash
$ heroku open
```

## Logging into the application

Use default administrator account to log in:

* login: admin
* password: admin
 
## if you want to reset db, run the following command

```bash
$ heroku pg:reset DB_NAME
```


[SOURCES]

http://www.redmine.org/projects/redmine/wiki/HowTo_Install_Redmine_on_Debian_8_with_Apache2-Passenger
http://andystu.github.io/blog/2015/02/16/how-to-install-and-deploy-redmine-on-heroku/