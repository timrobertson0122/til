Split a list of objects into two (or more) groups for spreading across columns

      - @credentials.in_groups(2) do |credentials|
        .col-lg-6.col-md-6.col-xs-6
          - credentials.each do |credential|
            table.table.table-dotted
              thead
                tr
                  th.h4 Username
                  th.h4 Password
              tbody
                tr
                  td.h5 = credential.username
                  td.h5 = credential.password
