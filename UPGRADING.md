# Upgrade Guide

## honeycomb-rails to beeline-ruby

1. Update Gemfile, remove `honeycomb-rails` and add `beeline-ruby`
1. Run `bundle install`
1. Remove the `honeycomb.rb` initializer from `config/initializers`
1. Add the following to the `config.ru` file
    ```ruby
    # config.ru
    require 'honeycomb-beeline'

    Honeycomb.init(writekey: 'YOUR_API_KEY', dataset: 'YOUR_DATASET')

    # these next two lines should already exist in some form in this file, it's important to init the honeycomb library before this
    require ::File.expand_path('../config/environment', __FILE__)
    run Rails.application
    ```
1. You can use the same write key and dataset from the honeycomb initialiser above, note: the honeycomb-beeline only supports sending events to one dataset. This is due to the fact that the new beeline will include traces for your application by default and these are only viewable from within the same dataset
1. Replace any `honeycomb_metadata` calls in your controllers like the following
    ```ruby
    def index
      @bees = Bee.all
      Rack::Honeycomb.add_field(request.env, :bees_count, @bees.count)
      # honeycomb_metadata[:bees_count] = @bees.count
    end
    ```
1. If you are manually using the libhoney client as well, it is suggested that you remove the usages of it and rely on the beeline.
1. Instrument interesting calls using the new `span` API as per the example below
    ```ruby
      class HomeController < ApplicationController
        def index
          Honeycomb.span do
            @interesting_information = perform_intensive_calculations(params[:honey])
          end
        end
      end
    ```
1. `honeycomb-rails` had the ability to automatically populate user information onto your events. Unfortunately `beeline-ruby` does not support this out of the box. You can use something like this snippet below to continue populating this (example for Devise)
    ```ruby
    class ApplicationController < ActionController::Base
      before_action do
        Rack::Honeycomb.add_field(request.env, "user.id", current_user.id)
        Rack::Honeycomb.add_field(request.env, "user.email", current_user.email)
      end
    end
    ```
1. (Optional) If you are using `Sequel` for database access there are some additional steps to configure
    ```ruby
    # config.ru
    require 'honeycomb-beeline'
    require 'sequel-honeycomb/auto_install'

    Honeycomb.init(writekey: 'YOUR_API_KEY', dataset: 'YOUR_DATASET')
    Sequel::Honeycomb::AutoInstall.auto_install!(honeycomb_client: Honeycomb.client, logger: Honeycomb.logger)

    # these next two lines should already exist in some form in this file, it's important to init the honeycomb library before this
    require ::File.expand_path('../config/environment', __FILE__)
    run Rails.application
    ```
