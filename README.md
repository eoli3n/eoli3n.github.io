# Create a new category

```
cat > newcat.html <<"EOF"
---
layout: newcat
title: Newcat
icon: fa-pencil-alt
order: 3
---
EOF

mkdir -p newcat/_posts

cp _layouts/archlinux.html _layouts/newcat.html
sed -i 's/archlinux/newcat/g' _layouts/newcat.html
```
