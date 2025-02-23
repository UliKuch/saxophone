<div align="center">
  <img alt="Saxophone" src="logo.svg" width="400px">
  <br>
  <br>
  <strong>A declarative SAX parsing library backed by Nokogiri, Ox or Oga.</strong>
  <br>
  <br>
  <a href="https://github.com/Absolventa/saxophone/actions/workflows/build.yml">
    <img src="https://github.com/Absolventa/saxophone/actions/workflows/build.yml/badge.svg" alt="Build Status" style="max-width: 100%;">
  </a>
</div>

## Origins

This repository is a fork of [pauldix/sax-machine](https://github.com/pauldix/sax-machine). We'd like to
thank all original authors and contributers for their work on the original repository. However, we have
the feeling that the original repository is not being actively maintained anymore - that's why we decided to
fork it and continue the work of the original authors in our façon. To make the distinction clear, we
renamed the project from that point to `Saxophone`.

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'saxophone'
```

And then execute:

```bash
$ bundle
```

## Usage

Saxophone can use either `nokogiri`, `ox` or `oga` as XML SAX handler.

To use **Nokogiri** add this line to your Gemfile:

```ruby
gem 'nokogiri', '~> 1.6'
```

To use **Ox** add this line to your Gemfile:

```ruby
gem 'ox', '>= 2.10.0'
```

To use **Oga** add this line to your Gemfile:

```ruby
gem 'oga', '>= 2.15'
```

You can also specify which handler to use manually, like this:

```ruby
Saxophone.handler = :nokogiri
```

## Examples

Include `Saxophone` in any class and define properties to parse:

```ruby
class AtomContent
  include Saxophone
  attribute :type
  value :text
end

class AtomEntry
  include Saxophone
  element :title

  # The :as argument makes this available through entry.author instead of .name
  element :name, as: :author
  element "feedburner:origLink", as: :url

  # The :default argument specifies default value for element when it's missing
  element :summary, class: String, default: "No summary available"

  element :content, class: AtomContent
  element :published

  ancestor :ancestor
end

class Atom
  include Saxophone

  # Use block to modify the returned value
  # Blocks are working with pretty much everything,
  # except for `elements` with `class` attribute
  element :title do |title|
    title.strip
  end

  # The :with argument means that you only match a link tag
  # that has an attribute of type: "text/html"
  element :link, value: :href, as: :url, with: {
    type: "text/html"
  }

  # The :value argument means that instead of setting the value
  # to the text between the tag, it sets it to the attribute value of :href
  element :link, value: :href, as: :feed_url, with: {
    type: "application/atom+xml"
  }

  elements :entry, as: :entries, class: AtomEntry
end
```

Then parse any XML with your class:

```ruby
feed = Atom.parse(xml)

feed.title # Whatever the title of the blog is
feed.url # The main URL of the blog
feed.feed_url # The URL of the blog feed

feed.entries.first.title # Title of the first entry
feed.entries.first.author # The author of the first entry
feed.entries.first.url # Permalink on the blog for this entry
feed.entries.first.summary # Returns "No summary available" if summary is missing
feed.entries.first.ancestor # The Atom ancestor
feed.entries.first.content # Instance of AtomContent
feed.entries.first.content.text # Entry content text
```

You can also use the elements method without specifying a class:

```ruby
class ServiceResponse
  include Saxophone
  elements :message, as: :messages
end

response = ServiceResponse.parse("
  <response>
    <message>hi</message>
    <message>world</message>
  </response>
")
response.messages.first # hi
response.messages.last  # world
```

To limit conflicts in the class used for mappping, you can use the alternate
`Saxophone.configure` syntax:

```ruby
class X < ActiveRecord::Base
  # This way no element, elements or ancestor method will be added to X
  Saxophone.configure(X) do |c|
    c.element :title
  end
end
```

Multiple elements can be mapped to the same alias:

```ruby
class RSSEntry
  include Saxophone

  element :pubDate,           as: :published
  element :pubdate,           as: :published
  element :"dc:date",         as: :published
  element :"dc:Date",         as: :published
  element :"dcterms:created", as: :published
end
```

If more than one of these elements exists in the source, the value from the *last one* is used. The order of
the `element` declarations in the code is unimportant. The order they are encountered while parsing the
document determines the value assigned to the alias.

If an element is defined in the source but is blank (e.g., `<pubDate></pubDate>`), it is ignored, and non-empty one is picked.

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request

## LICENSE

The MIT License

Copyright (c) 2009-2020:

* [Paul Dix](http://www.pauldix.net)
* [Julien Kirch](http://www.archiloque.net)
* [Ezekiel Templin](http://zeke.templ.in)
* [Dmitry Krasnoukhov](http://krasnoukhov.com)
* [Robin Neumann](https://github.com/neumanrq)

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
