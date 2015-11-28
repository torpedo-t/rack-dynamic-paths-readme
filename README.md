# Dynamic URL Paths

When you create a new repository on GitHub, how do URLs like `github.com/jmburges/my-repo` get generated? In our current examples, we would have to create a new `if` statement for each possible URL path. We are a dynamic application, and our application can't have to be rewritten every time a new user signs up. So, the concept of "dynamic routes" was created.

Let's assume we have a playlister app which has an array of Songs. First let's look at our `Song` object

```ruby
#song.rb

class Song

attr_accessor :title, :artist

end
```

Pretty simple class. Now we have our web app.

```ruby
class Application

  @@songs = [Song.new("Sorry", "Justin Bieber"),
            Song.new("Hello","Adele")]

  def call(env)
    resp = Rack::Response.new
    req = Rack::Request.new(env)
    
    @@songs.each do |song|
      resp.write "#{song.title}\n"
    end

    resp.finish
  end
end
```

We want more information about each song though. Similarily to GitHub I want to be able to go to a URL like `localhost:9292/songs/Sorry` and get all the information on Sorry. We are doing routes like this instead of just plain `GET` params because it's easier to read. Remember the path is given to us as a `String`. So we could write something like this:


```ruby
class Application

  @@songs = [Song.new("Sorry", "Justin Bieber"),
            Song.new("Hello","Adele")]

  def call(env)
    resp = Rack::Response.new
    req = Rack::Request.new(env)

    if req.path=="/songs/Sorry"
      resp.write @@songs[0].artist
    elsif req.path == "/songs/Hello"
      resp.write @@songs[1].artist
    end

    resp.finish
  end
end
```

That would be silly. Every time we create a new `Song` we would have to create a new statement in our `if`. Thankfully, we can just use the fact that paths are `Strings` like anything else. Let's do a regex match against the path, and then just grab the content after the `/song/` to figure out which `Song` our user would like.


```ruby
class Application

  @@songs = [Song.new("Sorry", "Justin Bieber"),
            Song.new("Hello","Adele")]

  def call(env)
    resp = Rack::Response.new
    req = Rack::Request.new(env)

    if req.path.match(/songs/)

      song_title = req.path.split("/songs/").last #turn /songs/Sorry into Sorry
      song = @@songs.find{|s| s.title == song_title}

      resp.write song.artist
    end

    resp.finish
  end
end
```

Now our routes are dynamic! We are able to just add songs, and everything else works for us. Remembering that everything is just Ruby is important. You have written a lot of Ruby, take comfort in your skills.
