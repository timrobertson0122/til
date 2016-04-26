```git branch -r | grep -v '\->' | while read remote; do git branch --track "${remote#origin/}" "$remote"; done```

https://stackoverflow.com/questions/10312521/how-to-fetch-all-git-branches
