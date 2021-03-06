## Docker Rails Starters

These are starter files for a Rails 6 dev environment using Docker & Docker Compose

### Setup

From your main code directory run the following, replacing your required ruby version

```sh
docker run --rm -it -v "$PWD:/app" ruby:2.7.2 bash
```

This will boot up a ruby container ready for you to generate a new rails project

From inside this container run the following, replacing _my_app_name_ with the name of your app

```sh
gem install rails
cd app
rails new my_app_name -d postgresql --skip-test --skip-bundle --skip-webpack-install
exit
```

Note how we skip 'bundle install'. This will be run later when building our app container.

Now copy all the files from the lib folder in this project into your newly created project and add the following lines to your .gitignore

```
# local docker setup
.env
Dockerfile
docker-compose.yml
docker-rails-entrypoint.sh
```

Edit the COMPOSE_PROJECT_NAME and APP_NAME entries in the .env file to the name of your new app

```sh
COMPOSE_PROJECT_NAME=my_app_name
APP_NAME=my_app_name
```

Make the rails entrypoint file executable

```sh
chmod +x docker-rails-entrypoint.sh
```

Build and run the app and db containers

```sh
docker-compose up -d --build app
```

Edit your `config/database.yml` and run the rails task to create the databases

```sh
docker-compose exec app rails db:create
```

You app will now be available at http://localhost:4000

### Webpacker (optional)

Install webpacker on the running app container

```sh
docker-compose exec app rails webpacker:install
```

Copy the webpacker entrypoint file from this project into yours and make it executable

```sh
chmod +x docker-webpacker-entrypoint.sh
```

Add the following line to your .gitinore

```sh
docker-webpacker-entrypoint.sh
```

Run the webpack-dev-server container

```sh
docker-compose up -d --build webpacker
```

Your app will now use the webpack-dev-server

### Hot reload your views over webpack

To use hot reloading for edits to your rails views open your config/webpacker.yml and change the hmr setting to true under the dev_server block

```yml
  dev_server:
    ...
    hmr: true
```

Add the following to your config/webpack/development.js above the module.exports line

```js
const chokidar = require("chokidar");
if (environment.config.devServer) {
  const chokidar = require("chokidar");
  environment.config.devServer.before = (app, server) => {
    chokidar
      .watch([
        "config/locales/**/*.yml",
        "app/views/**/*.html.erb",
        "app/assets/**/*.scss",
      ])
      .on("change", () => server.sockWrite(server.sockets, "content-changed"));
  };
}
```

Restart your webpacker container

```sh
docker-compose restart webpacker
```
