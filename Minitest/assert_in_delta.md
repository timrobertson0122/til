http://ruby-doc.org/stdlib-2.0.0/libdoc/minitest/rdoc/MiniTest/Assertions.html#method-i-assert_in_delta

        test 'valid (1st try)' do
          result = GroupEntry.call(@group.site, @barcode)
          assert_equal true, result[:success]
          assert_not_nil result[:group][:entered_at]
          assert_in_delta Time.current, result[:group][:entered_at], 1.second
        end        
