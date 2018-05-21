##### 1. Create new project:

Go to phoenix folder, go to  `phoenix/installer`

```
$ cd phoneix/installer

$ mix phx.new --no-ecto --no-brunch myproject  or $ mix phx.new myproject
$ cd myproject
$ mix deps.get
$ mix ecto.create && mix ecto.migrate
```

##### 2. 

1. How to use console: 

   `$ iex -S mix`

2.  Debug in Elixir: 

   `require IEx`

   in below we set breakpoint: `IEx.pry`

   Next: `respawn()`

   Run: `iex -S mix phx.server`







Database: 

virtual relationship to get data faster: 



 User model: 

    has_many :project_access,      Projects.Access  # have database with name "project_access"
    has_many :accessible_projects, through: [:project_access, :project]

project_access database: 



```
 schema "project_access" do
    belongs_to :user,       Inilab.Accounts.User
    belongs_to :project,    Inilab.Projects.Project
    belongs_to :invitation, Inilab.Coherence.Invitation

    field :name,    :string
    field :email,   :string

    timestamps()
  end
```





How to get data in controller: 

```
def user(conn, _params) do
    current_user =
      conn.assigns.current_user

    owned_projects =
      conn.assigns.projects
      |> Projects.owned_by_user(current_user)
      |> Ecto.Query.order_by(desc: :inserted_at)
      |> Repo.all
      |> Repo.preload([:city, :canton, :categories, :user, :votes, :favorites])

    accessible_projects =
      current_user
      |> Ecto.assoc(:accessible_projects)
      |> Ecto.Query.order_by(desc: :inserted_at)
      |> Repo.all
      |> Repo.preload([:city, :canton, :categories, :user, :votes, :favorites])


    render(conn, "user.html",
           owned_projects:      owned_projects,
           accessible_projects: accessible_projects,
           current_user:        current_user,
    )
  end
```



Create new migration: 

`$ mix ecto.gen.migration create_users`





Other: 

```
➜ mix help | grep -i phx
mix local.phx          # Updates the Phoenix project generator locally
mix phx.digest         # Digests and compresses static files
mix phx.digest.clean   # Removes old versions of static assets.
mix phx.gen.channel    # Generates a Phoenix channel
mix phx.gen.context    # Generates a context with functions around an Ecto schema
mix phx.gen.embedded   # Generates an embedded Ecto schema file
mix phx.gen.html       # Generates controller, views, and context for an HTML resource
mix phx.gen.json       # Generates controller, views, and context for a JSON resource
mix phx.gen.presence   # Generates a Presence tracker
mix phx.gen.schema     # Generates an Ecto schema and migration file
mix phx.gen.secret     # Generates a secret
mix phx.new            # Creates a new Phoenix v1.3.0 application
mix phx.new.ecto       # Creates a new Ecto project within an umbrella project
mix phx.new.web        # Creates a new Phoenix web project within an umbrella project
mix phx.routes         # Prints all routes
mix phx.server         # Starts applications and their servers
```



##### Create new module for api: 



* ```
  $ mix phx.gen.json Blog Post posts title:string content:string --no-context

  - creating lib/hello_web/controllers/post_controller.ex
  - creating lib/hello_web/views/post_view.ex
  - creating test/hello_web/controllers/post_controller_test.exs
  - creating lib/hello_web/views/changeset_view.ex
  - creating lib/hello_web/controllers/fallback_controller.ex
  ```


 

  ```

  
 mix phx.gen.json Projects Building buildings

- creating lib/real_estate_web/controllers/building_controller.ex
- creating lib/real_estate_web/views/building_view.ex
- creating test/real_estate_web/controllers/building_controller_test.exs
- creating lib/real_estate/projects/building.ex
- creating priv/repo/migrations/20180315095855_create_buildings.exs
- injecting lib/real_estate/projects/projects.ex
- injecting test/real_estate/projects/projects_test.exs










  ```





3. ##### Query: 

   ```
   def query(query \\ User, params, kind)
   def query(q, params, :user) when is_map(params) do
       Ecto.Query.where(q, [u], similarity(u.name, ^params["name"]) or similarity(u.name, ^params["given_name"]))
     end
   def query(q, term, :user) when is_bitstring(term) do
       Ecto.Query.where(q, [u], similarity(u.name, ^term) or similarity(u.given_name, ^term))
     end
   ```

   ​



2. Role, permission using Canary: 

   Note: we have to use `halt` in view to handle authentication 