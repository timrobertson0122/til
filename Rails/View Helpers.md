###### in helpers/application_helper.rb

    module ApplicationHelper
    
      def time(val)
        return nil unless val
        val.strftime("%H:%M")
      end
    
      def date(date)
        return '' unless date
        date.strftime "%e.%m.%Y"
      end
    
      def scroller_box(height = 500, &block)
        raw("<div style='overflow-y: auto; max-height:#{height}px'>" +
           capture(&block) + "</div>")
      end
    
      def currency(value)
        number_to_currency(value, precision: 2, delimiter: ".",
                                  separator: ',', unit: "€ ")
      end
    
      def num(str, precision = 0)
        number_to_currency(str, precision: precision, delimiter: ".",
                                unit: "", separator: ",")
      end
    
    end

###### in View file

###### e.g.

    | Total price: 
        b= currency @group.balance
