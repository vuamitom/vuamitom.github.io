
# Install

1. install gem
```bash
# On ubuntu 18
sudo apt-get install ruby-dev
``` 
2. install jekyll
```bash
gem install jekyll bundler
```
3. run in this folder
```bash
bundle install
bundle exec jekyll serve
```

# Update
```bash
bundle update
```
# Custom layout

1. Look for file path
```bash
# On MacOS
open $(bundle show minima)
# On Windows
explorer /usr/local/lib/ruby/gems/2.3.0/gems/minima-2.1.0
# On Linux
xdg-open $(bundle show minima)

```

2. copy to equivalent position to overwrite

