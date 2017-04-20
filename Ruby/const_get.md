 Checks for a constant with the given name, fails to NameError if not found. This method will recursively look up constant names if a namespaced class name is provided. For example, below it is looking up subclasses of BottleNumber that match the interpolated string "BottleNumber#{number}".
 
        class BottleNumber
          def self.for(number)
            begin
              const_get("BottleNumber#{number}")
            rescue NameError
              BottleNumber
            end.new(number)
           
           #...
        end  
