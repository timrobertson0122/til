```
rel = Model.some_scope
rel = rel.where(:attribute => attribute) if attribute
rel = rel.where("otherAttribute = 'C' AND relation_id IS NOT NULL AND some_column IS NOT NULL")
        .order("some_column DESC")
        .page(params[:page])    #### will_paginate #####
        .per_page(params[:limit].to_i)

pagination_object = rel
      
my_results_array = pagination_object.map(&:whatever_column).compact
```
ActiveRecord queries (such as `.where`) return ActiveRecord relation *objects*, which means you can build them up incrementally.
This enables me to use the conditional on line 2. The underlying SQL doesn't actually get fired until I call the `.map` method (which calls `.to_a`).
