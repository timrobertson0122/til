Found a strange bug at work when modifying an array of strings using `gsub` and a regex matcher...

```ruby
def modify_urls_to_grab_non_thumbnailed_images(thumbnail_urls)
      processed_urls = []
      thumbnail_urls.each do |thumbnail_url|
        # Bizarely thumbnails are always 3.jpg, and larger images are 11.jpg
        large_image = thumbnail_url.gsub(/(.+)3.jpg/, "#{$1}11.jpg")
        processed_urls << large_image
      end
      return processed_urls
    end
```

```
2.1.6 :008 > thumbnail_urls
 => ["https://img.cwom.co.uk/blm/89/1589/4364898/4364898_1_3.jpg", "https://img.cwom.co.uk/blm/89/1589/4364898/4364898_2_3.jpg", "https://img.cwom.co.uk/blm/89/1589/4364898/4364898_3_3.jpg", "https://img.cwom.co.uk/blm/89/1589/4364898/4364898_4_3.jpg", "https://img.cwom.co.uk/blm/89/1589/4364898/4364898_5_3.jpg", "https://img.cwom.co.uk/blm/89/1589/4364898/4364898_6_3.jpg"] 
```
```
2.1.6 :009 > processed_urls = []
 => [] 
```
```
2.1.6 :010 > thumbnail_urls.each do |thumbnail_url| large_image = thumbnail_url.gsub(/(.+)3.jpg/, "#{$1}5.jpg") ; puts "#{thumbnail_url} is now #{large_image} and $1 is #{$1}"; processed_urls << large_image end
https://img.cwom.co.uk/blm/89/1589/4364898/4364898_1_3.jpg is now 5.jpg and $1 is https://img.cwom.co.uk/blm/89/1589/4364898/4364898_1_
https://img.cwom.co.uk/blm/89/1589/4364898/4364898_2_3.jpg is now https://img.cwom.co.uk/blm/89/1589/4364898/4364898_1_5.jpg and $1 is https://img.cwom.co.uk/blm/89/1589/4364898/4364898_2_
https://img.cwom.co.uk/blm/89/1589/4364898/4364898_3_3.jpg is now https://img.cwom.co.uk/blm/89/1589/4364898/4364898_2_5.jpg and $1 is https://img.cwom.co.uk/blm/89/1589/4364898/4364898_3_
https://img.cwom.co.uk/blm/89/1589/4364898/4364898_4_3.jpg is now https://img.cwom.co.uk/blm/89/1589/4364898/4364898_3_5.jpg and $1 is https://img.cwom.co.uk/blm/89/1589/4364898/4364898_4_
https://img.cwom.co.uk/blm/89/1589/4364898/4364898_5_3.jpg is now https://img.cwom.co.uk/blm/89/1589/4364898/4364898_4_5.jpg and $1 is https://img.cwom.co.uk/blm/89/1589/4364898/4364898_5_
https://img.cwom.co.uk/blm/89/1589/4364898/4364898_6_3.jpg is now https://img.cwom.co.uk/blm/89/1589/4364898/4364898_5_5.jpg and $1 is https://img.cwom.co.uk/blm/89/1589/4364898/4364898_6_
```

On the first iteration $1 = nil as it isn't assigned until after the gsub completes, therefore the "first" image fails, the second gets the url of the first image and so on.

It has to do with timing and how ruby regexes work.

`gsub` sets `$1` but not until after it completes. So on the first iteration, it's blank. 
On the second loop through it has a value, but one set by the previous gsub. 

This can be rectified by using the block form of `gsub`. 

> "In the block form, the current match string is passed in as a parameter, and variables such as $1, $2, $`, $&, and $' will be set appropriately. The value returned by the block will be substituted for the match on each call.
