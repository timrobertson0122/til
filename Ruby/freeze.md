```
Prevents further modifications to obj. A RuntimeError will be raised if modification is attempted. 
There is no way to unfreeze a frozen object. See also Object#frozen?.

This method returns self.

a = [ "a", "b", "c" ]
a.freeze
a << "z"
produces:

prog.rb:3:in `<<': can't modify frozen Array (RuntimeError)
 from prog.rb:3
 ```
 
 Additionally, if we freeze string literals, the Ruby interpreter will only create one String object and will cache it for future use, 
 therefore this vastly improves performance.
