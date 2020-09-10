<div id="git-data-element" data-org="learn-co-curriculum" data-repo="active-record-associations-and-migrations"></div><header class="fis-header" style="visibility: hidden;"><a class="fis-fork-link" id="fork-link" href="#" target="_blank"><img id="fork-img" title="Fork This Assignment" alt="Fork This Assignment"></a><a class="fis-git-link" href="https://github.com/learn-co-curriculum/active-record-associations-and-migrations" target="_blank"><img id="repo-img" title="Open GitHub Repo" alt="GitHub Repo"></a><a class="fis-git-link" href="https://github.com/learn-co-curriculum/active-record-associations-and-migrations/issues/new" target="_blank"><img id="issue-img" title="Create New Issue" alt="Create New Issue"></a></header><h2>Objectives</h2>

  <ol>
  <li>Understand how and why Active Record implements associations between models.</li>
  <li>Use Active Record migrations and methods to build out a domain model that
  associates classes via the has-many/belongs-to <em>and</em> the many-to-many (or
  has-many-through) relationships.</li>
  </ol>

  <h2>What are Active Record Associations?</h2>

  <p>We already know that we can build our classes such that they associate with one
  another. We also know that it takes a lot of code to do it. Active Record
  associations allow us to associate models <em>and their analogous database tables</em>
  without having to write tons of code.</p>

  <p>Additionally, Active Record associations make actually working with our
  associated objects even quicker, neater and easier.</p>

  <p>Sounds great, right? Now that we have you totally hooked, let's take a look at
  how we use these AR associations.</p>

  <h2>How do we use AR Associations?</h2>

  <p>Active Record makes it easy to implement the following relationships between
  models:</p>

  <ul>
  <li>belongs_to</li>
  <li>has_one</li>
  <li>has_many</li>
  <li>has_many :through</li>
  <li>has_one :through</li>
  <li>has<em>and</em>belongs<em>to</em>many</li>
  </ul>

  <p>We don't need to worry about most of these right now. We'll concern ourselves
  with relationships that should sound familiar:</p>

  <ul>
  <li>belongs to</li>
  <li>has many</li>
  <li>has many through</li>
  </ul>

  <p>In order to implement these relationships, we will need to do two things:</p>

  <ol>
  <li>Write a migration that creates tables with associations. For example, if a
  cat belongs to an owner, the cats table should have an <code>owner_id</code> column.</li>
  <li>Use Active Record macros in the models.</li>
  </ol>

  <p>We'll go through both of these steps together, using our Playlister domain model.</p>

  <h2>Overview</h2>

  <p>In this walk-through, we'll be building out a domain model for our fictitious
  music playing app, Playlister. This app will catalog songs and their
  associated artists and genres.  </p>

  <p>We'll have three models: Artists, Songs, and Genres. By writing a few migrations
  and making use of the appropriate Active Record macros (more on that later), we
  will be able to:</p>

  <ul>
  <li>ask an Artist about its songs and genres</li>
  <li>ask a Song about its genre and its artist</li>
  <li>ask a Genre about its songs and artists.</li>
  </ul>

  <p>The relationships between artists, songs and genres will be enacted as follows:</p>

  <ul>
  <li>Artists have many songs and a song belongs to an artist.</li>
  <li>Artists have many genres through songs.</li>
  <li>Songs belong to a genre.</li>
  <li>A genre has many songs.</li>
  <li>A genre has many artists through songs.</li>
  </ul>

  <p>We will build these associations through the use of Active Record migrations and
  macros.</p>

  <h2>Building our Migrations</h2>

  <h3>The Song model</h3>

  <p>A song will belong to an artist <em>and</em> belong to a genre. Before we worry about
  the migration that will implement this in our songs table, let's think about
  what that table will look like:</p>

  <table>
  <thead>
  <tr>
  <th>id</th>
  <th>name</th>
  <th>artist_id</th>
  <th>genre_id</th>
  </tr>
  </thead>
  <tbody>
  <tr>
  <td>2</td>
  <td>Shake It Off</td>
  <td>1</td>
  <td>1</td>
  </tr>
  </tbody>
  </table>

  <p>We can see that the songs table will have an <code>artist_id</code> column and a <code>genre_id</code>
  column. We will give a given song an <code>artist_id</code> value of the artist it belongs
  to. The same goes for genre. These foreign keys, in conjunction with the
  Active Record association macros will allow our query to get an artist's songs or
  genres, a song's artist or genre, and a genre's songs and artists entirely
  through Active Record provided methods on our classes.</p>

  <p>Let's write the migration that will make this happen.</p>

  <ul>
  <li>Open a file, <code>db/migrate/03_create_songs.rb</code>
  </li>
  <li>Write the following migration:</li>
  </ul>

  <pre><code class="ruby">class CreateSongs &lt; ActiveRecord::Migration[4.2]
    def change
      create_table :songs do |t|
        t.string :name
        t.integer :artist_id
        t.integer :genre_id
      end
    end
  end
  </code></pre>

  <h3>The Artist Model</h3>

  <p>An artist will have many songs and it will have many genres <em>through</em> songs.
  These associations will be taken care of entirely through AR macros, which we'll
  get to in a bit. What do we mean by <em>through</em> songs? The table songs is the
  <code>JOIN</code> table! Remember that from previous labs? That means that songs has both
  an <code>artist_id</code> and a <code>genre_id</code> to combine those two tables together in a
  many-to-many relationship.</p>

  <p>Let's take a look at what our <code>artists</code> table will need to look like:</p>

  <table>
  <thead>
  <tr>
  <th>id</th>
  <th>name</th>
  </tr>
  </thead>
  <tbody>
  <tr>
  <td>1</td>
  <td>Taylor Swift</td>
  </tr>
  </tbody>
  </table>

  <p>Our artists table just needs a <code>name</code> column. Let's write the migration. In
  <code>db/migrate/01_create_artists.rb</code>:</p>

  <pre><code class="ruby">class CreateArtists &lt; ActiveRecord::Migration[4.2]
    def change
      create_table :artists do |t|
        t.string :name
      end
    end
  end
  </code></pre>

  <h3>The Genre Model</h3>

  <p>A genre will have many songs and it will have many artists through songs. These
  associations will be taken care of entirely through AR macros, which we'll get
  to in a bit.</p>

  <p>Let's take a look at what our genres table will need to look like:</p>

  <table>
  <thead>
  <tr>
  <th>id</th>
  <th>name</th>
  </tr>
  </thead>
  <tbody>
  <tr>
  <td>1</td>
  <td>pop</td>
  </tr>
  </tbody>
  </table>

  <p>Let's write our migration. In <code>db/migrate/02_create_genres.rb</code>:</p>

  <pre><code class="ruby">class CreateGenres &lt; ActiveRecord::Migration[4.2]
    def change
      create_table :genres do |t|
        t.string :name
      end
    end
  end
  </code></pre>

  <p>Great! Now go ahead and run <code>rake db:migrate</code> in your terminal to execute our
  table creations.</p>

  <h2>Building our Associations using AR Macros</h2>

  <h3>What is a macro?</h3>

  <p>A macro is a method that writes code for us (think metaprogramming). By invoking
  a few methods that come with Active Record, we can implement all of the
  associations we've been discussing.</p>

  <p>We'll be using the following AR macros (or methods):</p>

  <ul>
  <li><a href="http://guides.rubyonrails.org/association_basics.html#the-has-many-association"><code>has_many</code></a></li>
  <li><a href="http://guides.rubyonrails.org/association_basics.html#the-has-many-through-association"><code>has_many through</code></a></li>
  <li><a href="http://guides.rubyonrails.org/association_basics.html#the-belongs-to-association"><code>belongs_to</code></a></li>
  </ul>

  <p>Let's get started.</p>

  <h3>A Song Belongs to an Artist and A Genre</h3>

  <p>Create a file, <code>app/models/song.rb</code>. Define your <code>Song</code> class to inherit from
  <code>ActiveRecord::Base</code>. This is very important! If we don't inherit from Active
  Record Base, we won't get our fancy macro methods.</p>

  <pre><code class="ruby">class Song &lt; ActiveRecord::Base

  end
  </code></pre>

  <p>We need to tell the <code>Song</code> class that it will produce objects that can belong to
  an artist. We will do it with the <code>belongs_to</code> macro:</p>

  <pre><code class="ruby">class Song &lt; ActiveRecord::Base
    belongs_to :artist
  end
  </code></pre>

  <p>Songs also belong to a genre, so we'll use the same macro to implement that
  relationship:</p>

  <pre><code class="ruby">class Song &lt; ActiveRecord::Base
    belongs_to :artist
    belongs_to :genre
  end
  </code></pre>

  <h3>An Artist Has Many Songs</h3>

  <p>Create a file, <code>app/models/artist.rb</code>. Define your <code>Artist</code> class to inherit
  from <code>ActiveRecord::Base</code>:</p>

  <pre><code class="ruby">class Artist &lt; ActiveRecord::Base

  end
  </code></pre>

  <p>We need to tell the <code>Artist</code> class that each artist object can have many songs.
  We will use the <code>has_many</code> macro to do it.</p>

  <pre><code class="ruby">class Artist &lt; ActiveRecord::Base
    has_many :songs

  end
  </code></pre>

  <p>And that's it! Now, because our songs table has an <code>artist_id</code> column and
  because our <code>Artist</code> class uses the <code>has_many</code> macro, an artist has many songs!</p>

  <p>It is also true that an artist has many genres through songs. We will use the
  <code>has_many through</code> macro to implement this:</p>

  <pre><code class="ruby">class Artist &lt; ActiveRecord::Base
    has_many :songs
    has_many :genres, through: :songs
  end
  </code></pre>

  <h3>Genres Have Many Songs and Have Many Artists</h3>

  <p>Create a file <code>app/models/genre.rb</code>. In it, define a class, <code>Genre</code>, to inherit
  from <code>ActiveRecord::Base</code>.</p>

  <pre><code class="ruby">class Genre &lt; ActiveRecord::Base

  end
  </code></pre>

  <p>A genre can have many songs. Let's implement that with the <code>has_many</code> macro:</p>

  <pre><code class="ruby">class Genre &lt; ActiveRecord::Base
    has_many :songs
  end
  </code></pre>

  <p>A genre also has many artists through its songs. Let's implement this
  relationship with the <code>has_many through</code> macro:</p>

  <pre><code class="ruby">class Genre &lt; ActiveRecord::Base
    has_many :songs
    has_many :artists, through: :songs
  end
  </code></pre>

  <p>And that's it! The tests in this lesson are in place to ensure you've properly
  set up these associations. You can go ahead and run <code>learn</code> now to see if you
  pass them all before continuing.</p>

  <h2>Our Code in Action: Working with Associations</h2>

  <p>Go ahead and run the test suite and you'll see that we are passing all of our
  tests! Amazing! Our associations are all working, just because of our migrations
  and use of macros.</p>

  <p>Let's play around with our code.</p>

  <p>In your console, run <code>rake console</code>. Now we are in a Pry console that accesses
  our models.</p>

  <p>Let's make a few new songs:</p>

  <pre><code class="bash">[1]pry(main)&gt; hello = Song.new(name: "Hello")
  =&gt; #&lt;Song:0x007fc75a8de3d8 id: nil, name: "Hello", artist_id: nil, genre_id: nil&gt;
  [2]pry(main)&gt; hotline_bling = Song.new(name: "Hotline Bling")
  =&gt; #&lt;Song:0x007fc75b9f3a38 id: nil, name: "Hotline Bling", artist_id: nil, genre_id: nil&gt;
  </code></pre>

  <p>Okay, here we have two songs. Let's make some artists to associate them to. In
  the <em>same PRY sessions as above</em>:</p>

  <pre><code class="bash">[3] pry(main)&gt; adele = Artist.new(name: "Adele")
  =&gt; #&lt;Artist:0x007fc75b8d9490 id: nil, name: "Adele"&gt;
  [4] pry(main)&gt; drake = Artist.new(name: "Drake")
  =&gt; #&lt;Artist:0x007fc75b163c60 id: nil, name: "Drake"&gt;
  </code></pre>

  <p>So, we know that an individual song has an <code>artist_id</code> attribute. We <em>could</em>
  associate <code>hello</code> to <code>adele</code> by setting <code>hello.artist_id=</code> equal to the <code>id</code> of
  the <code>adele</code> object. BUT! Active Record makes it so easy for us. The macros we
  implemented in our classes allow us to associate a song object directly to an
  artist object:</p>

  <pre><code class="bash">[5] pry(main)&gt; hello.artist = adele
  =&gt; #&lt;Artist:0x007fc75b8d9490 id: nil, name: "Adele"&gt;
  </code></pre>

  <p>Now, we can ask <code>hello</code> who its artist is:</p>

  <pre><code class="bash">[6] pry(main)&gt; hello.artist
  =&gt; #&lt;Artist:0x007fc75b8d9490 id: nil, name: "Adele"&gt;
  </code></pre>

  <p>We can even chain methods to ask <code>hello</code> for the <em>name</em> of its artist:</p>

  <pre><code class="bash">[7] pry(main)&gt; hello.artist.name
  =&gt; "Adele"
  </code></pre>

  <p>Wow! This is great, but we're not quite where we want to be. Right now, we've
  been able to assign an artist to a song, but is the reverse true?</p>

  <pre><code class="bash">[7] pry(main)&gt; adele.songs
  =&gt; []
  </code></pre>

  <p>In this case, we still need to tell the <code>adele</code> Artist instance which songs
  it has. We can do this by pushing the song instance into <code>adele.songs</code>:</p>

  <pre><code class="bash">[7] pry(main)&gt; adele.songs.push(hello)
  =&gt; [#&lt;Song:0x007fc75a8de3d8 id: nil, name: "Hello", artist_id: nil, genre_id: nil&gt;]
  </code></pre>

  <p>Okay, now both sides of the relationships are updated, but so far all the work
  we've done has been with temporary instances of Artist and Song. To persist
  these relationships, we can use Active Record's <code>save</code> functionality:</p>

  <pre><code class="bash">[8] pry(main)&gt; adele.save
  =&gt; true
  [9] pry(main)&gt; adele
  =&gt; #&lt;Artist:0x007fc75b8d9490 id: 1, name: "Adele"&gt;
  </code></pre>

  <p>Notice that <code>adele</code> now has an <code>id</code>. What about <code>hello</code>?</p>

  <pre><code class="bash">[10] pry(main)&gt; hello
  =&gt; #&lt;Song:0x007fc75a8de3d8 id: 1, name: "Hello", artist_id: nil, genre_id: nil&gt;
  </code></pre>

  <p>Whoa! We didn't mention <code>hello</code> when we saved. However, we established an
  association by assigning <code>hello</code> as a song <code>adele</code> <em>has</em>. In order for <code>adele</code>
  to save, <code>hello</code> must also be saved. Thus, <code>hello</code> has also been given an <code>id</code>.</p>

  <p>Go ahead and do the same for <code>hotline_bling</code> and <code>drake</code> to try it out on your
  own.</p>

  <h2>Adding Additional Associations</h2>

  <p>Now, let's make a second song for adele:</p>

  <pre><code class="bash">[8] pry(main)&gt; someone_like_you = Song.new(name: "Someone Like You")
  =&gt; #&lt;Song:0x007fc75b5cabc8 id: nil, name: "Someone Like You", artist_id: nil, genre_id: nil&gt;
  [8] pry(main)&gt; someone_like_you.artist = adele
  =&gt; #&lt;Artist:0x007fc75b8d9490 id: 1, name: "Adele"&gt;
  </code></pre>

  <p>We've only updated the song, so we should expect that <code>adele</code> is not
  aware of this song:</p>

  <pre><code class="bash">[8] pry(main)&gt; someone_like_you.artist
  =&gt; #&lt;Artist:0x007fc75b8d9490 id: 1, name: "Adele"&gt;
  [9] pry(main)&gt; adele.songs
  =&gt; [#&lt;Song:0x007fc75b9f3a38 id: 1, name: "Hello", artist_id: 1, genre_id: nil&gt;]
  </code></pre>

  <p>Even if we save the song, <code>adele</code> will not be updated.</p>

  <pre><code class="bash">[8] pry(main)&gt; someone_like_you.save
  =&gt; true
  [8] pry(main)&gt; someone_like_you
  =&gt; #&lt;Song:0x007fc75b5cabc8 id: 2, name: "Someone Like You", artist_id: 1, genre_id: nil&gt;
  [9] pry(main)&gt; adele.songs
  =&gt; [#&lt;Song:0x007fc75b9f3a38 id: 1, name: "Hello", artist_id: 1, genre_id: nil&gt;]
  </code></pre>

  <p>But lets see what happens when we switch some things around. Creating one more
  song:</p>

  <pre><code class="bash">[8] pry(main)&gt; set_fire_to_the_rain = Song.new(name: "Set Fire to the Rain")
  =&gt; #&lt;Song:0x007fc75b5cabc8 id: nil, name: "Set Fire to the Rain", artist_id: nil, genre_id: nil&gt;
  </code></pre>

  <p>Then add the song to <code>adele</code>:</p>

  <pre><code class="bash">[9] pry(main)&gt; adele.songs.push(set_fire_to_the_rain)
  =&gt; [#&lt;Song:0x007fc75b9f3a38 id: 1, name: "Hello", artist_id: 1, genre_id: nil&gt;, #&lt;Song:0x00007feac2be4f38 id: 3, name: "Set Fire to the Rain", artist_id: 1, genre_id: nil&gt;]
  </code></pre>

  <p>Whoa! Check it out - we did not explicitly save <code>set_fire_to_the_rain</code>, but just
  by pushing the instance into <code>adele.songs</code>, Active Record has gone ahead and
  saved the instance. Not only that, notice that the song instance <em>also has an
  aritst</em>id!_</p>

  <pre><code class="bash">[8] pry(main)&gt; set_fire_to_the_rain.artist
  =&gt; #&lt;Artist:0x007fc75b8d9490 id: 1, name: "Adele"&gt;
  </code></pre>

  <p>So what is happening? Active Record is doing things for us behind the scenes,
  but when dealing with associations, it will behave differently depending on
  which side of a relationship between two models you are updating.</p>

  <p><strong>Remember</strong>: In a <code>has_many</code>/<code>belongs_to</code> relationship, we can think of the
  model that <code>has_many</code> as the parent in the relationship. The model that
  <code>belongs_to</code>, then, is the child. If you tell the child that it belongs to
  the parent, <em>the parent won't know about that relationship</em>. If you tell the
  parent that a certain child object has been added to its collection, <em>both the
  parent and the child will know about the association</em>.</p>

  <p>Let's see this in action again. Let's create another new song and add it to <code>adele</code>'s
  songs collection:</p>

  <pre><code class="bash">[10] pry(main)&gt; rolling_in_the_deep = Song.new(name: "Rolling in the Deep")
  =&gt; #&lt;Song:0x007fc75bb4d1e0 id: nil, name: "Rolling in the Deep", artist_id: nil, genre_id: nil&gt;
  </code></pre>

  <pre><code class="bash">[11] pry(main)&gt; adele.songs &lt;&lt; rolling_in_the_deep
  =&gt; [ #&lt;Song:0x007fc75bb4d1e0 id: 4, name: "Rolling in the Deep", artist_id: 1, genre_id: nil&gt;]
  [12] pry(main)&gt; rolling_in_the_deep.artist
  =&gt; #&lt;Artist:0x007fc75b8d9490 id: 4, name: "Adele"&gt;
  </code></pre>

  <p>We added <code>rolling_in_the_deep</code> to <code>adele</code>'s collection of songs and we can see
  the <code>adele</code> knows it has that song in the collection <em>and</em> <code>rolling_in_the_deep</code>
  knows about its artist. Not only that, <code>rolling_in_the_deep</code> is now persisted to
  the database.</p>

  <p>Notice that <code>adele.songs</code> returns an array of songs. When a model <code>has_many</code> of
  something, it will store those objects in an array. To add to that collection,
  we use the shovel operator, <code>&lt;&lt;</code>, to operate on that collection, and treat
  <code>adele.songs</code> like any other array.</p>

  <p>Let's play around with some genres and our has many through association.</p>

  <pre><code class="bash">[13] pry(main)&gt; pop = Genre.create(name: "pop")
  =&gt; #&lt;Genre:0x007fa34338d270 id: 1, name: "pop"&gt;
  </code></pre>

  <p>This time, we'll just use <code>create</code> directly, which would be the same as running
  <code>Genre.new</code>, then <code>Genre.save</code>.</p>

  <pre><code class="bash">[14] pry(main)&gt; pop.songs &lt;&lt; rolling_in_the_deep
  =&gt; [#&lt;Song:0x007fc75bb4d1e0 id: 4, name: "Rolling in the Deep", artist_id: 1, genre_id: 1&gt;]
  [15] pry(main)&gt; pop.songs
  =&gt; [#&lt;Song:0x007fc75bb4d1e0 id: 4, name: "Rolling in the Deep", artist_id: 1, genre_id: 1&gt;]
  [16] pry(main)&gt; rolling_in_the_deep.genre
  =&gt; #&lt;Genre:0x007fa34338d270 id: 1, name: "pop"&gt;
  </code></pre>

  <p>It's working! But even cooler is that we've established has many <em>through</em>
  relationships. By creating a genre, then pushing a song into that genre's list
  of songs, <em>the genre will now be able to produce its associated artists!</em></p>

  <pre><code class="bash">[16] pry(main)&gt; rolling_in_the_deep.artist
  =&gt; #&lt;Genre:0x007fa34338d270 id: 1, name: "pop"&gt;
  [17] pry(main)&gt; pop.artists
  =&gt; [#&lt;Artist:0x007fc75b8d9490 id: 1, name: "Adele"&gt;]
  [28] pry(main)&gt; adele.genres
  =&gt; [#&lt;Genre:0x007fa34338d270 id: 1, name: "pop"&gt;]
  </code></pre>

  <h2>Conclusion</h2>

  <p>So, order and direction of operations does matter when establishing associations
  between models - it is typically better to update the <code>has_many</code> side of a
  relationship to get the full benefit of Active Record's power. Still, as we can
  see, with just migrations and Active Record macros, we can start to build and
  persist associations between things!</p>

  <h2>Video Reviews</h2>

  <ul>
  <li><p><a href="https://www.youtube.com/watch?v=5dqPYRsQd10">Active Record Associations</a></p></li>
  <li><p><a href="https://www.youtube.com/watch?v=l9JCzNN2Z2U">Active Record Associations II</a></p></li>
  <li><p><a href="https://www.youtube.com/watch?v=WVBWlnUghOI">Aliasing Active Record Associations</a></p></li>
  <li><p><a href="https://www.youtube.com/watch?v=ZfJ1rqFcNFU">Blog CLI with Active Record and Associations</a></p></li>
  </ul>
