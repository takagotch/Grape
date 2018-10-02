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


desc '' do
end
get :public_timeline do
  Status.limit(20)
end


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
```


