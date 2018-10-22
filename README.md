# Canvas GroupMaker

The GroupMaker addition to the Canvas LMS open source enables both
students and teachers to use a richer set of group features. Students
can rate other students' group work-ethic and utilize normalized 
ratings to make informed and accurate decisions about other students
to work with. Teachers can control groups for students and have greater
confidence in the groups the create.

## Setup

Use either Ubuntu 16.04.5 or macOS 10.14 for the development and production environment OS 
### Getting the code

If you don't already have [Git](http://git-scm.com/), you can install it on Debian/Ubuntu by running

```
$ sudo apt-get install git
```

Once you have a copy of Git installed on your system, getting the latest source for Canvas is as simple as checking out code from the repo, like so:

```
~$ git clone https://code.vt.edu/griffc1/canvas-lms.git canvas
~$ cd canvas
~$ git checkout stable
```

## Dependency Installation

Canvas requires Ruby 2.4 or greater. A minimum version of 2.4.4 is recommended.

### Debian/Ubuntu 

We now need to install the Ruby libraries and packages that Canvas needs. On Debian/Ubuntu, there are a few packages you're going to need to install. If you're running Ubuntu 14.04 Trusty, you'll need to add a PPA in order to get Ruby 2.4:

```
$ sudo apt-get install software-properties-common
$ sudo add-apt-repository ppa:brightbox/ruby-ng
$ sudo apt-get update
```

```
$ sudo apt-get install ruby2.4 ruby2.4-dev zlib1g-dev libxml2-dev \
                       libsqlite3-dev postgresql-9.5 libpq-dev \
                       libxmlsec1-dev curl make g++
```
> Note: In Ubuntu, in case you encounter any error such as E: Package 'postgresql-9.5' has no installation candidate, it may be because postgresql-9.5 is not available in that Ubuntu version. See  https://www.postgresql.org/download/linux/ubuntu/

Node.js installation:
```
$ curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
$ sudo apt-get install -y nodejs build-essential
```

[Yarn installation](https://github.com/instructure/canvas-lms/blob/stable/package.json#L7):
```
$ npm install -g yarn@1.9.4
```

After installing Postgres, you will need to set your system username as a postgres superuser.  You can do so by running the following commands:

```
sudo -u postgres createuser $USER
sudo -u postgres psql -c "alter user $USER with superuser" postgres
```

### macOS

For macOS, you'll need to install the [Command Line Tools for Xcode](http://developer.apple.com/downloads), and make sure you have Ruby 2.4. You can find out what version of Ruby your Mac came with by running:

```
$ ruby -v
```

You also need Postgres and the [xmlsec library](http://www.aleksey.com/xmlsec/) installed. The easiest way to get these is via [homebrew](http://brew.sh/). Once you have homebrew installed, just run:

```
$ brew install postgresql@9.5 nodejs xmlsec1
```

### Ruby Gems

Most of Canvas' dependencies are Ruby Gems. Ruby Gems are a Ruby-specific package management system that operates orthogonally to operating-system package management systems.


### Bundler

Canvas uses Bundler as an additional layer on top of Ruby Gems to manage versioned dependencies. Bundler is great!

You can install Bundler using Ruby Gems:

```
$ sudo gem install bundler:1.16.4
```

## Canvas Dependencies

```
~$ cd canvas
~/canvas$ bundle install
~/canvas$ yarn install --pure-lockfile
# Sometimes you have to run this command twice if there is an error 
~/canvas$ yarn install --pure-lockfile
```

If you're on macOS Mavericks or macOS Yosemite and hit an error with the thrift gem, you might have to set the following bundler flag and then run bundle install again (see https://issues.apache.org/jira/browse/THRIFT-2219):

```
~/canvas$ bundle config build.thrift --with-cppflags='-D_FORTIFY_SOURCE=0'
```

If you are on El Capitan and encounter an error with the thrift gem that has something like the following in the text.
```
compact_protocol.c:431:41: error: shifting a negative signed value is undefined [-Werror,-Wshift-negative-value]
```

Use the following command to install the gem and then continue with another bundle install.

```
gem install thrift -v 0.8.0 -- --with-cppflags=\"-D_FORTIFY_SOURCE=0 -Wno-shift-negative-value\"
```

The problem is the clang got updated in El Capitan and some of the flags cause problems now on install for the old version of thrift.

If you hit an error with the eventmachine gem, you might have to set the following bundler flag and then run bundle install again:

```
~/canvas$ bundle config build.eventmachine --with-cppflags=-I/usr/local/opt/openssl/include
```

### JavaScript Runtime

You'll also need a JavaScript runtime to translate our CoffeeScript code to JavaScript and a few other things.  We use Node.js for this. macOS users can download the installer from [node.js](http://nodejs.org). Linux users should already have it from the `apt-get install` step above.

CoffeeScript can be installed the same way as on other platforms, through `npm` (which is included with the nodeJS installation):
```
$ sudo npm install -g coffee-script@1.6.2
```

## Data setup

### Canvas default configuration

Before we set up all the tables in your database, our Rails code depends on a small few configuration files, which ship with good example settings, so, we'll want to set those up quickly. We'll be examining them more shortly. From the root of your Canvas tree, you can pull in the default configuration values like so:

```
~/canvas$ for config in amazon_s3 delayed_jobs domain file_store outgoing_mail security external_migration; \
          do cp -v config/$config.yml.example config/$config.yml; done
```

### Database configuration

Now we need to set up your database configuration. We have provided a sample file for quickstarts, so you just need to copy it in. You'll also want to create two databases. Depending on your OS (i.e. on Linux), you may need to use a postgres user to create the database, and configure database.yml to use a specific username to connect. See the [[Production Start]] tutorial for details on doing that. On macOS your local user will have permissions to create databases already, so no special configuration is necessary.

```
~/canvas$ cp config/database.yml.example config/database.yml
~/canvas$ createdb canvas_development
```

Note: When installing postgres with brew, you may have trouble connecting to the database and you may get an error like:

```
~/canvas$ createdb canvas_development
createdb: could not connect to database postgres: could not connect to server: No such file or directory
    Is the server running locally and accepting
    connections on Unix domain socket "/var/pgsql_socket/.s.PGSQL.5432"?
```

If you get a connection error when creating your databases, run the following and add it to your .bash_profile:

```
export PGHOST=localhost
```

If, after that, you get another error like:

```
~/canvas$ createdb canvas_development
createdb: could not connect to database template1: could not connect to server: Connection refused
    Is the server running on host "localhost" (::1) and accepting
    TCP/IP connections on port 5432?
could not connect to server: Connection refused
    Is the server running on host "localhost" (127.0.0.1) and accepting
    TCP/IP connections on port 5432?
could not connect to server: Connection refused
    Is the server running on host "localhost" (fe80::1) and accepting
    TCP/IP connections on port 5432?
```

then postgres may not be running.  To start it:

```
$ initdb /usr/local/var/postgres -E utf8
$ pg_ctl -D /usr/local/var/postgres -l /usr/local/var/postgres/server.log start
```

### Database population

Once your database is configured, we need to actually fill the database with tables and initial data. You can do this by running our migration and initialization tasks from your application's root:

```
~/canvas$ bundle exec rails canvas:compile_assets
~/canvas$ bundle exec rails db:initial_setup
```

Additionally consider setting up a shared database for all developers. For more information on how 
to manage any canvas_development database configuration and sharing a signle database checkout
the [Database Management Section] (docs/database_management.md) section

### File Generation

Canvas needs to build a number of assets before it will work correctly. You will need to rerun:

```
~/canvas$ bundle exec rails canvas:compile_assets
```

Note that we've seen trouble with npm trying to hold too many files open at once.  If you see an error with `libuv` while running npm, try increasing your `ulimit`.  To do this in macOS add `ulimit -n 4096` to your `~/.bash_profile` or `~/.zsh_profile`.

## Performance Tweaks

Installing redis will significantly improve your Canvas performance. For detailed instructions, see [[Production Start#redis]]. On Ubuntu, use the following:

```
sudo apt-get update
sudo apt-get install redis-server
redis-server
echo -e "development:\n  cache_store: redis_store" > config/cache_store.yml
echo -e "development:\n  servers:\n  - redis://localhost" > config/redis.yml
```

 On macOS, use the following:

```
brew install redis
redis-server /usr/local/etc/redis.conf
echo -e "development:\n  cache_store: redis_store" > config/cache_store.yml
echo -e "development:\n  servers:\n  - redis://localhost" > config/redis.yml
```

If you're just looking to try out Canvas, there's some minor configuration tweaks you can make to give yourself a real performance boost. All you need to do is add three lines to a file in your configuration directory. (If you plan on doing Canvas development, you may want to skip this step or only enable class caching, as these settings will require you to restart your server each time you change Ruby or ERB files.)

```
~/canvas$ echo -n 'config.cache_classes = true
config.action_controller.perform_caching = true
config.action_view.cache_template_loading = true
' > config/environments/development-local.rb
```

## A note about emails

Canvas will often attempt to send email. With the Quick Start instructions, email will go straight to the console that `rails server` is running on. If you want to set up email that actually goes to email addresses, please follow the Canvas LMS production instructions on emaill setup.

## Ready, Set, Go!

Now you just need to start the Canvas server! You will need to run the *rails server* daemon:

```
~/canvas$ bundle exec rails server
```

The site will be available on localhost:3000 after this.

## Logging in For the First Time

Your username and password will be whatever you set it up to be during the `rails db:initial_setup` step above. (You should have seen a prompt on the command line that asked for your email and password.)

## Production Deployment

Add additional notes about how to deploy this on a live system

## Built With

Ruby on Rails
React.js
HTML
CSS/SCSS
NPM
Yarn
PostgreSQL
RubyMine IDE

## Authors

* **Ben Austin**
* **Griffin College** 
* **Vamsi Manne**
* **Richard Patton**
* **Raghu Srinivasan**
* **David Van Scoik**

## License

This project rocks an MIT License! 

## Acknowledgments

* This README.md template was based off of the work done by [PurpleBooth](https://github.com/PurpleBooth)

