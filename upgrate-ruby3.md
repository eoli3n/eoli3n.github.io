
bundle remove webrick
bundle add webrick
sudo xbps-install libedit-devel
sudo xbps-install libyaml-devel
gem update

```
/home/user/dev/eoli3n.github.io/vendor/bundle/ruby/3.2.0/gems/jekyll-gallery-generator-1.2.4/lib/jekyll-gallery-generator.rb:98:in `initialize': undefined method `exists?' for File:Class (NoMethodError)

      unless File.exists?(gallery_index)
                 ^^^^^^^^
Did you mean?  exist?
```

https://stackoverflow.com/a/75353113

rbenv install 3.1.3
