### Grape
---

https://github.com/ruby-grape/grape
```
gem install grape
gem 'grape'
bundle install
```

```ruby
module Twitter
  class API < Grape::API
    version 'v1', using: :header, vendor: 'twitter'
    format :json
    prefix :api
    helpers do
      def current_user
        @current_user ||= User.authorize!(env)
      end
      def authenticate!
        error('401 Unauthorized', 401) unless current_user
      end
    end
    resource :statuses do
      desc 'Return a public timeline.'
      get :public_timeline do
        Status.limit(20)
      end
      desc 'Return a personal timeline.'
      get :home_timeline do
        authenticate!
        current_user.statuses.limit(20)
      end
      desc 'Return a status.'
      params do
        requires :id, type: Integer, desc: 'Status id.'
      end
      route_param :id do
        get do
          Status.find(params[:id])
        end
      end
      desc 'Create a status.'
      params do
        requires :status, type: String, desc: 'Your status.'
      end
      post do
        authenticate!
        Status.create!({
          user: current_user,
          text: params[:status]
        })
      end
      desc 'Update a status.'
      params do
        requires :id, type: String, desc: 'Status ID'
        requires :status, type: String, desc: 'Your status.'
      end
      put ':id' do
        authenticate!
        current_user.statuses.find(params[:id]).update({
          user: current_user,
          text: params[:status]
        })
      end
      desc 'Delete a status.'
      params do
        requires :id, type: String, desc: 'Status ID.'
      end
      delete ':id' do
        authenticate!
        current_user.statuses.find(params[:id]).destroy
      end
    end
  end
end

# config.ru
require 'sinatra'
require 'grape'
class API < Grape::API
  get :hello do
    { hello: 'world' }
  end
end
class Web < Sinatra::Base
  get '/' do
    'Hello world.'
  end
end
use Rack::Session::Cookie
run Rack::Cascade.new[API, Web]

# application.rb
config.paths.add File.join('app', 'api'), glob: File.join('**', '*.rb')
config.autoload_paths += Dir[Rails.root.join('app', 'api', '*')]

# config/routes
mount Twitter::API => '/'

class Twitter::API < Grape::API
  mount Twitter::APIv1
  mount Twitter::APIv2
end

class Twitter::API < Grape::API
  mount Twitter::APIv1 => '/v1'
end

class Twitter::API < Grape::API
  before do
    header 'X-Base-Header', 'will be defined for all APIs that are mounted below'
  end
  mount Twitter::Users
  mount Twitter::Search
end

version 'v1', using: :path
version 'v1', using: :header, vendor: 'twitter'
version 'v1', using: :accept_version_header
version 'v1', using: :param
version 'v1', using: :param, parameter: 'v'


desc 'Return your public timeline' do
  summary 'summary'
  detail 'more details'
  params API::Entitles::Status.documentation
  success API::Entities::Entity
  failure [[401, 'Unauthorized', 'Entities::Error']]
  named 'My named route'
  headers XAuthToken: {
      description: 'Validates your identitiy',
      required: true
    },
    XOptionHeader: {
      description: 'Not really needed',
      required: false
    }
  hidden false
  deprecated false
  is_array true
  nickname 'nickname'
  produces ['application/json']
  consumes ['application/json']
  tags ['tag1', 'tag2']
end
get :public_timeline do
  Status.limit(20)
end

get :public_timeline do
  Status.order(params[:sort_by])
end

post '/statuses' do
  Status.create!(text: params[:text])
end

post 'upload' do
  # file in params[:image_file]
end

class API < Grape::API
  include Grape::Extensions::Hashie::Mash::ParamBuilder
  params do
    optional :color, type: String
  end
  get do
    params.color # params[:color]
  end
end

params do
  build_with Grape::Extensions::Hash::ParamBuilder
  optional :color, type: String
end

format :json
post 'users/sinup' do
  { 'declared_params' => declared(params) }
end

{
  "declared_params": {}
}

format :json
params do
  requires :user, type: Hash do
    requires :first_name, type: String
    requires :last_name, type: String
  end
end
post 'users/signup' do
  { 'declared_params' => declared(params) }
end

{
  "declared_params": {
    "user": {
      "first_name": "first name",
      "last_name": "last name"
    }
  }
}

format :json
namespace :parent do
  params do
    requires :parent_name, type: String
  end
  namespace ':parent_name' do
    params do
      requires :child_name, type: String
    end
    get ':child_name' do
      {
        'without_parent_namespaces' => declared(params, include_parent_namespaces: false),
        'with_parent_namespaces' => declared(params, include_parent_namespaces: true)
      }
    end
  end
end

{
  "without_parent_namespaces": {
    "child_name": "bar"
  },
  "": {
    "parent_name": "foo",
    "child_name": "bar"
  },
}

format :json
params do
  requires :first_name, type: String
  optional :last_name, type: String
end
post 'users/signup' do
  { 'declared_params' => declared(params, include_missing: false) }
end

{
  "declared_params": {
    "user": {
      "first_name": "first name"
    }
  }
}

{
  "declared_params": {
    "first_name": "first name",
    "last_name": null
  }
}

format :json
params do
  requires :user, type: Hash do
    requires :first_name, type: String
    optional :last_name, type: String
    requires :address, type: Hash do
      requires :city, type: String
      optional :region, type: String
    end
  end
end
post 'users/signup' do
  { 'declared_params' => declared(params, include_missing: false) }
end

format :json
params do
  requires :user, type: Hash do
    requires :frist_name, type: String
    optional :last_name, type: String
    requires :address, type: Hash do
      requires :city, type: String
      optional :region, type: String
    end
  end
end
post 'users/signup' do
  { 'declared_params' => declared(params, include_missing: false) }
end


params do
  requires :id, type: Integer
  optional :text, type: String, regexp: /\A[a-z]+\z/
  group :media, type: Hash do
    requires :url
  end
  optional :audio, type: Hash do
    requires :format, type: Symbol, values: [:mp3, :wav, :acc, :ogg], default: :mp3
  end
  mutually_exclusive :media, :audio
end
put ':id' do
  # params[:id] is an Integer
end

params do
  optional :color, type: String, default: 'blue'
  optional :random_number, type: Integer, default: -> { Random.rand(1..100) }
  optional :non_random_number, type: Integer, default: Random.rand(1..100)
end

params do
  optional :color, type: String, default: 'blue', values" ['red', 'green']
end

params do
  optional :color, type: String, default: 'blue', values: ['blue', 'red', 'green']
end

```
#### Integer/Fixnum and Coercions
```

```


```sh
run Twitter::API

use ActiveRecord::ConnectionAdapters::ConnectionManagement
run Twitter::API

curl http://localhost:9292/v1/statuses/public_timeline
curl -H Accept:application/vnd.twitter-v1+json http://localhost:9292/statuses/public_timeline
curl -H "Accept-Version:v1" http://localhost:9292/statuses/public_timeline
curl http://localhost:9292/statuses/public_timeline?apiver=v1
curl http://localhost:9292/statuses/public_timeline?v=v1
curl -d '{"text": "140 characters"}' 'http://localhost:9292/statuses' -H Content-Type:application/json -v
curl -X POST -H "Content-Type: application/json" localhost:9292/users/sinup -d '{"user": {"first_name":"first name", "last_name": "last name"}}'
curl -X POST -H "Content-Type: application/json" localhost:9292/users/signup -d '{"users":{"first_name":"first name", "last_name": "last name", "random": "never shown"}}'
curl -X GET -H "Content-Type: application/json" localhost:9292/parent/foo/bar
curl -X POST -H "Content-Type: application/json" localhost:9292/users/signup -d '{"user": {"first name":"first name", "random": "never shown", "address": { "city": "SF" }}}'
curl -x POST -H "Content-Type: application/json" localhost:9292/users/signup -d '{"user": {"first_name":"first name", "first name": null, "address": { "city": "SF"}}}'
```


