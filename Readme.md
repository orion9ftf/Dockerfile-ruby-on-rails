### Create App using Docker

***Creating files***

```shell
Dockerfile
Gemfile
Gemfile.lock
entrypoint.sh
docker-compose.yml
```

***Dockerfile (ruby version?)***

```Dockerfile
FROM ruby:3.0.4

WORKDIR /name-app

RUN apt-get update -qq && apt-get install -y nodejs postgresql-client imagemagick libvips

COPY Gemfile /name-app/Gemfile
COPY Gemfile.lock /name-app/Gemfile.lock

RUN bundle install

COPY entrypoint.sh /usr/bin/
RUN chmod +x /usr/bin/entrypoint.sh
ENTRYPOINT ["entrypoint.sh"]

EXPOSE 3000

CMD ["rails", "server", "-b", "0.0.0.0"]
```

***Gemfile file, with:***

```rb
source "https://rubygems.org"
git_source(:github) { |repo| "https://github.com/#{repo}.git" }

# Bundle edge Rails instead: gem "rails", github: "rails/rails", branch: "main"
gem "rails", "~> 7.0.4" # Rails version
```

***Gemfile.lock file***

```rb
nothing ...
```

***entrypoint.sh file***

```sh
#!/bin/bash
set -e

# Remove a potentially pre-existing server.pid for Rails.
rm -f /name-app/tmp/pids/server.pid

# Then exec the container's main process (what's set as CMD in the Dockerfile).
exec "$@"
```

***docker-compose.yml file***
```yml
version: "3.9"
services:
  db:
    image: postgres:latest
    volumes:
      - ./tmp/db:/var/lib/postgresql/data
    environment:
      POSTGRES_USERNAME: postgres
      POSTGRES_PASSWORD: password

  redis:
    image: 'redis:latest'
    ports:
      - '6379:6379'

  web:
    build: .
    command: bash -c "rm -f tmp/pids/server.pid && bundle exec rails s -p 3000 -b '0.0.0.0'"
    volumes:
      - .:/name-app
    ports:
      - "3000:3000"
    depends_on:
      - db
      - redis
    environment:
      - REDIS_URL=redis://redis:6379/
```

### Good luck!
