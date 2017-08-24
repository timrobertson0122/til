        def setup_index_params
          @index_params = client_params.to_hash
          # Do no keep search history for a long period of time.
          cookies.signed['lets_search_params'] = {
            :value   => JSON.generate(:params => @index_params),
            :expires => 1.hour.from_now
          }
        end
        
`setup_index_params` is a `before_action` on our LetsListingsController. We use a signed cookie, setting it's value to the serialized
search criteria (for our JSON API) so that we can return the user to their search results.         
        
        # Sets a simple session cookie.
        # This cookie will be deleted when the user's browser is closed.
        cookies[:user_name] = "david"

        # Cookie values are String based. Other data types need to be serialized.
        cookies[:lat_lon] = JSON.generate([47.68, -122.37])

        # Sets a cookie that expires in 1 hour.
        cookies[:login] = { value: "XJ-122", expires: 1.hour.from_now }

        # Sets a signed cookie, which prevents users from tampering with its value.
        # The cookie is signed by your app's `secrets.secret_key_base` value.
        # It can be read using the signed method `cookies.signed[:name]`
        cookies.signed[:user_id] = current_user.id

        # Sets an encrypted cookie value before sending it to the client which
        # prevent users from reading and tampering with its value.
        # The cookie is signed by your app's `secrets.secret_key_base` value.
        # It can be read using the encrypted method `cookies.encrypted[:name]`
        cookies.encrypted[:discount] = 45

        # Sets a "permanent" cookie (which expires in 20 years from now).
        cookies.permanent[:login] = "XJ-122"

        # You can also chain these methods:
        cookies.permanent.signed[:login] = "XJ-122"
