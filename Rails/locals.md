http://api.rubyonrails.org/classes/ActionView/PartialRenderer.html

####  _table.html.slim
    tbody
    - for grp in groups do #groups is the local
    
#### index.html.slim
    ul.nav.nav-tabs role="tablist"
          li.active role="presentation" 
            a aria-controls="cash" data-toggle="tab" href="#cash" role="tab"  Cash
    .tab-content
          #cash.tab-pane.active role="tabpanel"
            = render 'table', groups: @report.cash_groups #shorthand for render partial: "table", locals: { groups: @report.cash_groups }. Sets instance variable to local
            
#### Class/Controller

    def cash_groups
      groups.where(pay_method_id: 1)
    end
