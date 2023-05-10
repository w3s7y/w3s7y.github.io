# w3s7y.github.io

## Local run/test (windows)
```shell
docker run -it -v //${PWD}:/app -p 4000:4000 ruby
cd /app
bundle install
bundle exec jekyll serve -H 0.0.0.0 --drafts
```
