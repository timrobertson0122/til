http://davidlesches.com/blog/titles-and-seo-tags-for-rails-5-a-better-way  

Use Rails' config locales translation mechanism to set all of your static titles and tags. 
This also makes it very easy to translate into other languages.

Then allow for defaults to be overridden with dynamic information (when on a show page for example).

application_helper.rb
```
def title override = nil
  meta_or_override :title, override
end

def meta_keywords override = nil
  meta_or_override :keywords, override
end

def meta_description override = nil
  meta_or_override :description, override
end

private

def page_key
  [ controller_name, action_name ].join('_').to_sym
end

def meta_or_override type, override
  if override
    content_for(type, override)
    return
  end

  if content_for?(type)
    content_for(type)
  else
    # config/locales/meta.en.yml
    t "meta.#{page_key}.#{type}", default: t("meta.default.#{type}")
  end
```
config/locales/meta.en.yml (set all your static and default titles and tags)
```
en:
meta:
  default:
    title: 'Welcome to my website'
    keywords: ''
    description: ''
    image:  ''
  sales_listing_index:
    title: ''
    keywords: ''
    description: ''
    image: ''
  sales_listings_show:
    title: ''
    keywords: ''
    description: ''
    image: ''
  contacts_show:
    title:  'Contact Us'
    keywords: ''
    description: 'contact us'
    image:  ''
```      
sales_listing/show.html.erb (overwrite the above defaults where needed)
```
<% title @listing.title %>
<% meta_description @listing.description %>
<% meta_image @listing.photos[0].main_url %>
<body.....
```
application.html.erb
```
<title><%= title.blank? ? base_title : title + " | " + base_title %></title>
<meta name="keywords" content="<%= meta_keywords %>"/>
<meta name="description" content="<%= meta_description %>"/>
<meta property="og:image" content="<%= meta_image %>"/>
<meta property="og:title" content="<%= title.blank? ? base_title : title + " | " + base_title %>"/>
<meta property="og:url" content="<%= canonical_url %>"/>
<meta property="og:site_name" content="<%= base_title %>"/>
<link rel="canonical" href="<%= canonical_url %>" />
 ```
