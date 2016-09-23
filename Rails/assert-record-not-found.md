        test "DESTROY: won't delete slots from other sites" do
          @request.env['HTTP_REFERER'] = root_url
          @site = sites(:two)
          @slot = FactoryGirl.create :slot, service: @service
          @slot.update date: @date, time: '08:00', site_id: @site.id
          @reservation = FactoryGirl.create :sale, slot: @slot
          assert_raises(ActiveRecord::RecordNotFound) do
            delete :destroy, params: {id: @slot.id}
          end
        end
