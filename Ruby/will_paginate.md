```
New in Active Record 3

# paginate in Active Record now returns a Relation
Post.where(:published => true).paginate(:page => params[:page]).order('id DESC')

# the new, shorter page() method
Post.order('created_at DESC').page(params[:page])
```

e.g
```
pagination_object = ReapitProperty.active.muva.page(params[:page])

render :json            => pagination_object.map(&:property_listing),
       :each_serializer => Web::V1::Sales::ListingSerializer,
       :meta            => {
         :total_entries => pagination_object.total_entries,
         :total_pages   => pagination_object.total_pages,
         :current_page  => pagination_object.current_page.to_i,
         :per_page      => ReapitProperty.per_page,
         :out_of_bounds => pagination_object.out_of_bounds?
       }
```
