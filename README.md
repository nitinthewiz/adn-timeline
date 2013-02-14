## App.net Timeline

Display one or more App.net account's recent posts. Comes in coffeescript and javascript. The coffeescript is well documented. You should read it if you're curious about how this works.

## Dependencies

1. jQuery (for ajax and dom manipulation)
2. jQuery.cookie (optional but recommended)

Note: I generally dislike expecting people to use jQuery for such a simple plugin, but chances are, you're already using it. If enough people are interested, I can write a
standalone version with jXHR or something.

## Usage

There are many ways to add a timeline.

#### Add the script.
After including jQuery, jQuery.cookie (optionally) and adn-timeline.js, add a script to initialize your ADN timeline.

    $(document).ready(function(){
      AdnTimeline.init()
    })

#### Add an element.

Somehwere on your page, add a block-level element (probably a `div` or a `section`) with a class `adn-timeline` and add your username a `data-username` attribute.

    <div class='adn-timeline' data-username='imathis'></div>

#### Want more than one account?

If you want to have more than one feed on your page, just add another div with a different username.

    <div class='adn-timeline' data-username='imathis'></div>
    <div class='adn-timeline' data-username='octopress'></div>

Note: Data is appended to the target elements so we can have content in these elements without worring that it will be overwritten. It would make sense to do something
like this.

    <div class='adn-timeline' data-user='imathis'>
      <h4>@imathis on App.net</h4>
    </div>

The heading will remain and posts will be added below in a `<ul>`.

## Configuration

### Defaults

| Config     | Default                 | Description
|:-----------|:------------------------|:----------------------------------------------|
| `el`       | '.adn-timeline'         | A jQuery selector (handles multiple elements) |
| `count`    | 4                       | How many posts to show from your timeline     |
| `replies`  | false                   | Include reply posts                           |
| `reposts`  | false                   | Include reposts                               |
| `cookie`   | 'adn-timeline-username' | Choose a different name for the cookie        |
| `callback` | (function(){})          | Provide a callback function                   |
| `render`   | AdnTimeline.render      | Provide a custom html render function         |


### Configure with data attributes

In the simple example above we configured the username using an HTML5 data attribute. All other configuration (aside from `el` and `callback`) can be set that way as well.
Here's an example.

    <div class='.adn-timeline' data-username='imathis' data-count='10' data-replies='true' data-reposts='true' data-cookie='delicious-cookie'></div>

### Configure with an options hash

Using an options hash allows us to change the selector and use a custom callback.

    $(document).ready(function(){
      AdnTimeline.init({
          el:      '.timeline'
        , count:   5 
        , replies: true 
        , reposts: false 
        , cookie:  'delicious-cookie' 
        , callback: function (data) { console.log(data) }
        , renderer: function (el, data) { /* rendering code here */ }
      })
    })

### Combine the options hash and the data attributes

Options are loaded in order of the element data attributes, the options hash, and finally the defaults. This means we can combine methods of setting options like this.
    
    $(document).ready(function(){
      AdnTimeline.init({
          el:      '.timeline'
        , replies: true 
      })
    })

    <div class='timeline' data-username='imathis'></div>
    <div class='timeline' data-username='octopress' data-reposts='true'></div>

Here we have changed the selector used, both accounts will show replies, and the Octopress account will show reposts.

### HTML Rendering

The default render function appends the following HTML template to the target element(s).

    <ul>
      <li>
        <figure class="post">
          <blockquote>
            <p>Post Text</p>
          </blockquote>
          <figcaption>
            <a href='http://alpha.app.net/username/post/id'>
              <time datetime="2013-02-14T05:03:14Z">2h</time>
            </a>
          </figcaption>
        </figure>
      </li>
      ...
    </ul>


While this may seem somewhat verbose, I believe it is a semantically appropriate way to display a timeline feed. After all they are a series of quotes with timestamped meta data linking to the source.
If you haven't seen a figure used to group a `blockquote` with metadata, take a look at the [blockquote spec](http://www.whatwg.org/specs/web-apps/current-work/multipage/grouping-content.html#the-blockquote-element) where you'll find an exmaple just like what I've done.

If you have an alternate proposal I'd love to hear it. Currently the strongest argument against this markup is that people often treat blockquotes or figures with global styling not anticipating them to be used
like this. This would mean they would have to change their site designs to be less broad or override some styles to integrate this timeline.

### Using a custom render function

It's fairly easy add a custom render function. The function will receive an jQuery wrapped element and a array of post data hashes with the following format:

    author: { 
      username:   post.user.username,
      name:       post.user.name,
      url:        post.user.canonical_url 
    }
    url:          post.canonical_url
    date:         post.created_at
    display_date: post.created_at (converted to relative time)
    text:         post.html (with trimmed display urls and auto-linked mentions and hashtags)

To write a custom render function we can simply pass it as an option in the options hash. Adding a custom render function could be as simple as this.

    AdnTimeline.init({
      render: (function(el, posts) {
        var html, i, len
        html  = "<ul>"
        for (i = 0, len = posts.length; i < len; i++) {
          html += "<li>"
          html += ("<p>" + posts[i].text + "</p>")
          html += "</li>"
        }
        html += "</ul>"
        el.append(html)
      })
    })

Which would output:

    <ul>
      <li>
        <p>post text<p>
      </li>
    <ul>

Note: The current HTML render function doesn't make use of the author data. But I've considered reformating reposts to use this data in a nicer way. Currently reposts use
the `>> @username: ` format.

## License MIT

Copyright 2013, Brandon Mathis.
Released under an [MIT license](http://opensource.org/licenses/MIT).
