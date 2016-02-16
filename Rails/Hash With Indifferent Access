This is something that Rails quietly gives you, and which you won't get with a PORO (plain old Ruby object).

Essentially it doesn't care whether you pass `:symbols` or `"strings"` to the hash, it will convert them all into strings intelligently.

This is particularly useful when dealing with `params`, which we do all the time in Rails. I believe it also provides a slight performance boost too, though that may just be because we're using symbols.
