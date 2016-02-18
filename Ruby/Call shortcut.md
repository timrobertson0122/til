This isn't something Ruby does, just a handy shortcut I learnt from the senior dev I work with.

      module ServiceObject
        extend ActiveSupport::Concern
      
        included do
          def self.call(*args)
            new(*args).perform
          end
        end
      end

If I then include this ServiceObject into my classes, I can call `.call` on the Class object, which will initialize a new instance, passing it all the args from `initialize`, and then call `.perform` - which is just the name of a method we're using to calculate some value associated to that object.
