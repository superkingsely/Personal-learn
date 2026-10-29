find . -maxdepth 4
( -name .git -o -name bin -o -name obj -o -name node_modules ) -prune
-o -print | sort