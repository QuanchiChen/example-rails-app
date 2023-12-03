# Example Rails Application

Develop an example Rails application and deploy it to Heroku.

## Useful Link

[Step-by-step tutorial on deploying a Rails application to Heroku](https://devcenter.heroku.com/articles/getting-started-with-rails6)

Please use this README as a supplement to the above tutorial. The reason is that only following the above tutorial does not work for me.

## Prerequisites

```
ruby -v
ruby 3.1.2p20 (2022-04-12 revision 4491bb740a) [x86_64-linux]

rails -v
Rails 6.1.5

node -v
v16.18.1

yarn -v
1.22.19
```

Please also install [PostgreSQL](https://www.postgresql.org/download/), [Git](https://git-scm.com/downloads), and [Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli) on your local machine.

## Step 1 - Local Setup

Log in to Heroku via the Terminal. If you have enabled Multi-Factor Authentication (MFA), use your Heroku API Key as the password. You can find your API key in Account Settings.

```
heroku login
```

Use the Rails new application generator to create a Rails application.

```
rails _6.1.5_ new example-app --database=postgresql

cd example-app/

bundle lock --add-platform x86_64-linux --add-platform ruby

code .
```

Modify the content of  `config/database.yaml` . Please refer to  `config/database.yaml` provided by this repository.

Note that it may not work on your local machine. My local machine is Ubuntu 22.04.1.

Create two local databases.

```
bin/rails db:create
```

Define a hello action in ApplicationController and set the route to map the root to that controller action. Please refer to `app/controllers/application_controller.rb` and `config/routes.rb`.

Start the Puma server.

```
bin/rails server
```

Go to `http://localhost:3000/`. Hello, world!

## Step 2 - Deployment Preparation

Create a Procfile in the root directory of the project.

```
web: bundle exec puma -C config/puma.rb
```

Fix babel regression errors. Your application may contain bugs! Try executing the following command in Terminal.

```
rake webpacker:clobber webpacker:compile  2>&1 | grep "Cannot find package '@babel/plugin-proposal-private-methods'"
Error: Cannot find package '@babel/plugin-proposal-private-methods' imported from /home/quanchic/Desktop/RailsLearning/example-app/babel-virtual-resolve-base.js
```

No worries. Let's fix the errors. Open `babel.config.js`.

Replace `@babel/plugin-proposal-private-methods` with `@babel/plugin-transform-private-methods`.

Replace `@babel/plugin-proposal-private-property-in-object` with `@babel/plugin-transform-private-property-in-object`.

Hopefully, the following command now works!

```
rake webpacker:clobber webpacker:compile
```

Also, please [specify the Node.js version used by Heroku](https://devcenter.heroku.com/articles/nodejs-support#specifying-a-node-js-version). Add the following code to `package.json`.

```
"engine": {
  "node": "16.18.1"
}
```

Commit the code.

```
git add -A

git commit -m "Initialise repository"

git log
```



## Step 3 - Deployment

Create a Heroku application.

```
heroku create --stack heroku-20

git remote -v
```

Provision a **non-free** database.

```
heroku addons:create heroku-postgresql:mini
```

Wait a minute. Before deploying the code to Heroku, we need to execute a few more commands.

```
heroku buildpacks

heroku buildpacks:set heroku/nodejs -i 1

heroku buildpacks:set heroku/ruby -i 2

heroku buildpacks

heroku config:set NODE_OPTIONS=--openssl-legacy-provider
```

For the last command, you may have a look at this [issue](https://github.com/webpack/webpack/issues/14532). You may skip executing that command and see if Heroku gives a similar error.

Deploy the application to Heroku.

```
git push heroku master
```

Now we are at the end of this amazing journey!

## Step 4 - Finally

Remove the database and the application from your Heroku account to minimise the cost.

```
heroku addons:destroy heroku-postgresql

heroku apps:destroy

heroku addons --all

heroku apps --all
```

Goodbye! Enjoy coding and deploying!
