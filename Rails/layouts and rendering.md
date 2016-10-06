*Rails first looks for a file in app/views/layouts with the same base name as the controller. For example, rendering actions from the PhotosController class will use app/views/layouts/photos.html.erb (or app/views/layouts/photos.builder). If there is no such controller-specific layout, Rails will use app/views/layouts/application.html.erb or app/views/layouts/application.builder. If there is no .erb layout, Rails will use a .builder layout if one exists. Rails also provides several ways to more precisely assign specific layouts to individual controllers and actions.*

To override the default settings you can declare a layout for the entire controller, or you can defer the choice of layout until runtime by using a symbol. In my example below I'm rendering either the `Provider` or `Print` layouts when a request is made on the index action in my controller.

------------------

        class Providers::ReportsController < Providers::ApplicationController

          include SetsMonth

          before_action ->(c) { c.set_month :provider_reports }

          layout :reports_layout

          def index
            @provider = current_provider
            @report = AdminProviderDetailReport.new(current_site, @provider, @m, @y)
          end

          private

          def reports_layout
            params[:print] == "1" ? "print" : "providers"
          end

        end
 -----------------
 
 I'm passing a param into my print button link to determine that the print layout should be used in this instance.
 
        = link_to fa_icon('print'), providers_reports_path(print: 1), target: :blank, class: "btn btn-lg btn-default noprint"
