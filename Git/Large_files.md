Accidentally logged output to a text file which got caught in a commit. Despite deleting the file locally every time I tried to push Github told me I couldn't because of this file. Found the following command on StackOverflow to fix it!

`git filter-branch --index-filter 'git rm -r --cached --ignore-unmatch <file/dir>' HEAD`

https://stackoverflow.com/questions/19573031/cant-push-to-github-because-of-large-file-which-i-already-deleted
