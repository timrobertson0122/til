        class BottleNumber
          def self.for(number)
            begin
              const_get("BottleNumber#{number}")
            rescue NameError
              BottleNumber
            end.new(number)
         #...
       end  
