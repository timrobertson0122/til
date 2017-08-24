    class Model < BaseModel
      ARRAY_VARIABLE_NAMES = [ :stuff_array, :stuff_ids_array].freeze
      
      attr_accessor :id, :name, :something, :something_else, *ARRAY_VARIABLE_NAMES
      
      def initialize(attributes = {})
        super
        ARRAY_VARIABLE_NAMES.each do |attribute|
          public_send("#{attribute}=", []) if public_send(attribute).blank?
      end    
    end
    
`*` - Allow any number of additional attributes  
`public_send` - respect private methods  
`("#{attribute}=", []) if public_send(attribute).blank?` - send the attribute as an empty array if it's empty, rather than as `nil` which will blow up on array methods.
