# [Rack::App](http://rack-app.com/) [![Build Status][travis-image]][travis-link]

[travis-image]: https://travis-ci.org/rack-app/rack-app.svg?branch=master
[travis-link]: http://travis-ci.org/rack-app/rack-app
[travis-home]: http://travis-ci.org/

![Rack::App](http://rack-app-website.herokuapp.com/image/msruby_old.png)

Your next favourite rack based micro framework that is totally addition free! 
Have a cup of awesomeness with your performance designed framework!

The idea behind is simple.
Keep the dependencies and everything as little as possible,
while able to write pure rack apps,
that will do nothing more than what you defined.

If you want see fancy magic, you are in a bad place buddy!
This includes that it do not have such core extensions like activesupport that monkey patch the whole world.

Routing can handle any amount of endpoints that can fit in the memory,
so if you that crazy to use more than 10k endpoint,
you still dont have to worry about response speed.

It was inspirited by sinatra, grape, and the pure use form of rack.
It's in production, powering Back Ends on Heroku

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'rack-app'
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install rack-app


## Is it Production ready?

Yes, in fact it's already powering heroku hosted micro-services.

## Features

* Null time routing that allows as many endpoint you just want without impact on route look up
* Unified Error handling for class level where you can describe error rescues for your endpoints
* simple Sinatra inspirited endpoint definition syntax sugar methods on class singleton level
 * get
 * post
 * put
 * delete
 * options
 * patch 


## Usage

config.ru

#### basic 

```ruby

require 'rack/app'

class App < Rack::App

  desc 'some hello endpoint'
  get '/hello' do
    'Hello World!'
  end

end

```

#### complex

```ruby

require 'rack/app'

class App < Rack::App

  mount SomeAppClass

  headers 'Access-Control-Allow-Origin' => '*',
          'Access-Control-Expose-Headers' => 'X-My-Custom-Header, X-Another-Custom-Header'

  serializer do |object|
    object.to_s
  end

  desc 'some hello endpoint'
  get '/hello' do
    return 'Hello World!'
  end

  namespace '/users' do 
  
    desc 'some restful endpoint'
    get '/:user_id' do
      response.status = 201
      params['user_id'] #=> restful parameter :user_id
      say #=> "hello world!"
    end
    
  end
   
  desc 'some endpoint that has error and will be rescued'
  get '/make_error' do
    raise(StandardError,'error block rescued')
  end


  def say
    "hello #{params['user_id']}!"
  end

  error StandardError, NoMethodError do |ex|
    {:error=>ex.message}
  end

  root '/hello'

end

```

you can access Rack::Request with the request method and 
Rack::Response as response method. 

By default if you dont write anything to the response 'body' the endpoint block logic return will be used

## Testing 

for testing use rack/test or the bundled testing module for writing unit test for your rack application

```ruby

require 'spec_helper'
require 'rack/app/test'

describe App do

  include Rack::App::Test

  rack_app described_class

  describe '/hello' do
    # example for params and headers and payload use
    subject{ get(url: '/hello', params: {'dog' => 'meat'}, headers: {'X-Cat' => 'fur'}, payload: 'some string') }

    it { expect(subject.status).to eq 200 }

    it { expect(subject.body.join).to eq "Hello World!" }
  end

  describe '/users/:user_id' do
    # restful endpoint example
    subject{ get(url: '/users/1234') }

    it { expect(subject.body.join).to eq 'hello 1234!'}

    it { expect(subject.status).to eq 201 }

  end

  describe '/make_error' do
    # error handled example
    subject{ get(url: '/make_error') }

    it { expect(subject.body.join).to eq '{:error=>"error block rescued"}' }
  end

end


```

## Example Apps To start with

* [Basic](https://github.com/rack-app/rack-app-example-basic)
  * bare bone simple example app 
  
* [Escher Authorized Api](https://github.com/rack-app/rack-app-example-escher)
  * complex authorization for corporal level api use

## [Benchmarking](https://github.com/adamluzsi/rack-app.rb-benchmark)

| name                   | user                   | system                 | total                  | real                   | current / fastest      |
| ---------------------- | ---------------------- | ---------------------- | ---------------------- | ---------------------- | ---------------------- |
| rack                   | 1.1654381654675966e-05 | 2.4570024569497616e-07 | 1.1900081900005149e-05 | 1.2464733825060753e-05 | 1.0                    |
| rack-app               | 3.6610644258403255e-05 | 5.210084033675504e-07  | 3.7131652662448054e-05 | 3.900771708675591e-05  | 3.129                  |
| ramaze                 | 4.25602409640276e-05   | 1.024096385549314e-06  | 4.358433734967247e-05  | 4.372164909660012e-05  | 3.508                  |
| hobbit                 | 8.1779279279624e-05    | 5.518018017874161e-07  | 8.23310810825621e-05   | 8.432744200307479e-05  | 6.765                  |
| brooklyn               | 0.00013620870870686714 | 1.1486486486805621e-06 | 0.00013735735735869695 | 0.0001414384894851337  | 11.347                 |
| nancy                  | 0.00015193693693615643 | 1.576576576582871e-06  | 0.00015351351350898534 | 0.00015808854805109845 | 12.683                 |
| sinatra                | 0.0002704504504498543  | 3.8738738738769526e-06 | 0.0002743243243240957  | 0.0002756480900905045  | 22.114                 |
| scorched               | 0.00027810810810750846 | 2.6126126126122157e-06 | 0.00028072072072019753 | 0.00028194524324291086 | 22.619                 |
| grape                  | 0.0005396232596183593  | 5.004095004135632e-06  | 0.0005446273546398578  | 0.0005578575077825894  | 44.755                 |
| rails                  | 0.0006839808153554279  | 2.4028776978980613e-05 | 0.0007080095923087499  | 0.0007334601023317118  | 58.843                 |
| camping                | 0.000893834281092368   | 1.7831031681169243e-05 | 0.0009116653127801263  | 0.000936258097467815   | 75.113                 |
| cuba                   | 0.0012919824140475418  | 1.5931254995760534e-05 | 0.001307913669066536   | 0.0013495860959551855  | 108.272                |

## Roadmap 

### Team [Backlog](https://docs.google.com/spreadsheets/d/19GGX51i6uCQQz8pQ-lvsIxu43huKCX-eC1526-RL3YA/edit?usp=sharing)

If you have anything to say, you can leave a comment. :)

## Development

After checking out the repo, run `bin/setup` to install dependencies. Then, run `rake spec` to run the tests. You can also run `bin/console` for an interactive prompt that will allow you to experiment.

To install this gem onto your local machine, run `bundle exec rake install`. To release a new version, update the version number in `version.rb`, and then run `bundle exec rake release`, which will create a git tag for the version, push git commits and tags, and push the `.gem` file to [rubygems.org](https://rubygems.org).

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/adamluzsi/rack-app.rb This project is intended to be a safe, welcoming space for collaboration, and contributors are expected to adhere to the [Contributor Covenant](contributor-covenant.org) code of conduct.

