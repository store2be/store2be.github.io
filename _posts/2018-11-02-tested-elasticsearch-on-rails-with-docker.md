---
date: 2018-11-02
categories:
  - Rails
  - Elasticsearch
  - Docker
title: Tested Elasticsearch on Rails with Docker
author_staff_member: 01_peter
medium_link: https://medium.com/store2be-tech/tested-elasticsearch-on-rails-with-docker-545565a6b6e7
practical_dev_link: https://dev.to/peterfication/tested-elasticsearch-on-rails-with-docker-2chi
---

Recently, the product team has been mentioning the need for a better search solution in our apps. That's why I took the time to look into Elasticsearch after a two year break. For me, it was amazing to see how easy it was to set up a proof of concept that was even integrated into CI. The last part is quite important for us because we are very keen on testing in general — especially when it comes to CI.

Our development setup is already running on docker compose, which while having some disadvantages of course, mostly regarding speed, has the benefit that you can share and spin up the whole infrastructure without installing anything but the docker related things. If you only have a Rails app with PostgreSQL this might not be worth it. Sharing your development infrastructure via code however also gives you the advantage of sharing the correct version of PostgreSQL with the team. Also, the more things you add to the mix, the more annoying it gets to spin everything up. For instance, we use Redis for caching, sessions and our worker queue: yet another infrastructure dependency we have to share across the team.

Having our development and CI setup already on docker made adding Elasticsearch to our app super easy! **I won't go into details about how to use Elasticsearch with Rails, [because there are other articles doing this well already](https://medium.com/@tranduchanh.ms/global-full-text-search-experiences-in-rails-with-aws-elasticsearch-ea3b47a00a80).**

Only one thing that I wasn’t getting right from the beginning: The `match` parameter of searchkick (Ruby library around Elasticsearch, used in the article mentioned above). You might want to think about whether you need `word_start` or `word_middle` when deciding how to match your queries ;)**

## Setup of Elasticsearch

I needed to add Elasticsearch to our development environment and to our CI setup. For the local environment, we use docker compose. So this addition to our `docker-compose.yml` was needed:

```yml
  ...
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.4.2
    hostname: elasticsearch
     ports:
      - "9200:9200"
```

This tells docker compose to spin up an Elasticsearch container and hook it up via DNS to our app container. So you just need to set environment variable `ELASTICSEARCH_URL: http://elasticsearch:9200` and you are good to go.

For CI we use [CircleCI with Docker](https://circleci.com/blog/build-cicd-piplines-using-docker/#configyml-file). There it was just a one-liner to get it up and running:

```yml
jobs:
  build:
    docker:
      ...
      - image: docker.elastic.co/elasticsearch/elasticsearch:6.4.2
```

And that’s it!

I won’t talk about Elasticsearch in production here, because this is a whole other thing of course and you should think about using a managed solution if you don’t have any experience with it.

## Testing the search endpoint

We use RSpec for testing. This is how a basic test of the search would look like:

```ruby
# searches_controller.rb
class SearchController < ApplicationController
  def execute
    data = if params[:query].blank?
      []
    else
      Store.search(params[:query]).map do |store|
        {
          id: store.id,
          title: store.title,
        }
      end
    end

    render json: { data: data }
  end
end

# searches_spec.rb
require 'rails_helper'

describe SearchController do
  describe '#execute' do
    it 'works' do
      store = Store.create!(title: 'Alexa Shopping Center')

      # The query is misspelled on purpose!
      get '/search?query=Alxa'

      expect(response).to have_http_status(200)
      expect(JSON.parse(response.body)).to eq({
        data: [{
          id: store.id,
          title: store.title,
        },],
      })
    end
  end
end
```

We also use [VCR](https://github.com/vcr/vcr) for recording HTTP requests to external APIs for future test runs. However, the Elasticsearch calls are local and can be executed during tests. That’s why we needed to add an exception to VCR:

```ruby
VCR.configure do |config|
  config.ignore_hosts '127.0.0.1', 'localhost', 'elasticsearch'
end
```

## Why not just use PostgreSQL full text search?

There are [good](http://rachbelaid.com/postgres-full-text-search-is-good-enough/) [articles](https://robots.thoughtbot.com/implementing-multi-table-full-text-search-with-postgres) out there that explain how to enable a similar full-text search in PostgreSQL without adding another dependency to the infrastructure. This is definitely an option as well!

Following these tutorials was not as easy as it was for me to setup Elasticsearch with searchkick, there is still a lot of setup required, even though it’s only code and not infrastructure. The option of using a managed Elasticsearch service is great in that it doesn’t add many operational difficulties. The argument for Elasticsearch in my opinion is the reduced complexity of development, however we are still just trying things out when it comes taking our app search to the next level and might go for a PostgreSQL solution in the end.
