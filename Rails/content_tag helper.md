```
def render_double_image_row(area_name, image_hash)
    content_tag(:div, :class => 'row pb-2') do
      content = []
      content << content_tag(:div, :class => 'col-12 col-sm-6 text-center pb-2 pb-sm-0') do
        image_tag("artifacts/our_areas/areas/#{formatted_area_name(area_name)}/#{image_hash[:image_1_name]}.jpg", :alt => "#{image_hash[:image_1_alt]}", :class => "img-fluid")
      end
      content << content_tag(:div, :class => 'col-12 col-sm-6 text-center pb-2 pb-sm-0') do
        image_tag("artifacts/our_areas/areas/#{formatted_area_name(area_name)}/#{image_hash[:image_2_name]}", :alt => "#{image_hash[:image_2_alt]}", :class => "img-fluid")
      end
      content.join(' ').html_safe
    end
  end
```

```
<%= render_double_image_row(@area,
  :image_1_name => 'blue_flag_beach',
  :image_1_alt => 'Blue flag beaches of Alum Chine',
  :image_2_name => 'balcony_view',
  :image_2_alt => 'Wonderful sea views')
%>
```

```
<div class="row pb-2">
  <div class="col-12 col-sm-6 text-center pb-2 pb-sm-0">
    <img alt="Blue flag beaches of Alum Chine" class="img-fluid" src="/blah/blue_flag_beach.jpg">
  </div> 
  <div class="col-12 col-sm-6 text-center pb-2 pb-sm-0">
    <img alt="Wonderful sea views" class="img-fluid" src="/blah/balcony_view.jpg">
  </div>
</div>
```
