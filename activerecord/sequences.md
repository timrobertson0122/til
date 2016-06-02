        def next_id
          Credential.connection.execute("SELECT nextval('id_seq')")[0]['nextval'].to_i
        end
        
        class CreateCredentialSequence < ActiveRecord::Migration
          def up
            execute "CREATE SEQUENCE id_seq;"
          end
        
          def down
            execute "DROP SEQUENCE id_seq;"
          end
        end

Create a sequence of numbers to ensure uniqueness

        You are now connected to database "vat_backend_dev" as user "timrobertson".
        [local] timrobertson@vat_backend_dev=# CREATE SEQUENCE foo_bar;
        CREATE SEQUENCE
        Time: 4.421 ms
        [local] timrobertson@vat_backend_dev=# SELECT nextval('foo_bar');
         nextval
        ---------
               1
        (1 row)
        
        Time: 2.733 ms
        [local] timrobertson@vat_backend_dev=# SELECT nextval('foo_bar');
         nextval
        ---------
               2
        (1 row)
        
        Time: 0.249 ms
        [local] timrobertson@vat_backend_dev=# SELECT nextval('foo_bar');
         nextval
        ---------
               3
        (1 row)
