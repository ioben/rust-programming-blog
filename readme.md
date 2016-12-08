This is a blog for the Rust programing language.  It should be useful those beginning to use Rust, the safe systems language from Mozilla.

[The blog is available here](https://ioben.github.io/rust-programming-blog/)

## Setup

1. Install RVM

2. Install Ruby 2.3.1 - `rvm install 2.3.1`

3. Install bundler - `gem install bundler`

4. Install project gems - `bundle install`

Then during development run:

```
bundle exec jekyll serve
```

And to build a distributable:

```
./tools/build BASE_URL
./tools/distribute WEBSITE DIRECTORY
```
