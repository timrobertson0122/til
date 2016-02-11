# Reload

This is called to reload the record from the database, often during tests to confirm that the record was successfully written to the database, or modified. E.g.

    test "UPDATE can update the pricelists for each service" do
    create_reservation_pricelist
    patch :update, update_params
    assert_equal 'New Name', @reservation_pricelist.reload.name
    end
