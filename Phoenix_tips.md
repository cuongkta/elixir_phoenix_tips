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





##### 3. Query: 

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

##### 4 . Role, permission using Canary: 

Note: we have to use `halt` in view to handle authentication 

##### 5. Custom validation in Changeset: 

```elixir
 @doc false
  def changeset(%Company{} = company, attrs) do
    company
    |> cast(attrs, [:name, :description, :avatar_path, :type, :number_phone, :email, :address])
    |> validate_required([:name, :email, :number_phone])
    |> validate_format(:email, ~r/@/)
    |> validate_url_format(:avatar_path)
    |> unique_constraint(:name, message: "Company is existing")
    |> unique_constraint(:email, message: "Company's email is existing")
  end



@url "http"
  defp validate_url_format(changeset, field) do
    case changeset.valid? do
      true ->
        url = get_field(changeset, field)
        if is_nil(url) do
          changeset
        else
          case String.starts_with?(url, @url) or File.exists?(url) do
            true -> changeset
            _ -> add_error(changeset, :avatar_path, "It's not correct path of avatar")
          end
        end
      _ ->
        changeset
    end
  end
```



#### II.

How to get value of key in a list. 

```
 params = [
      keyword:      params["keyword"], 
      district_id:  params["district_id"], 
      ward_id:      params["ward_id"],
      city_id:      params["city_id"],
      developer:    params["developer"],
      page:         params["page"]
    ] 
# Get
search_term =  params[:keyword]
```





#### III. Note for mess: 

How to work with relationship in database: https://medium.com/@Stephanbv/elixir-phoenix-a-todo-and-user-relationship-todo-application-part-4-7c2d80d22dea





Nested associate: 

1. Repo: https://hexdocs.pm/ecto/Ecto.Repo.html

   ```elixir
   # Use a single atom to preload an association
   posts = Repo.preload posts, :comments

   # Use a list of atoms to preload multiple associations
   posts = Repo.preload posts, [:comments, :authors]

   # Use a keyword list to preload nested associations as well
   posts = Repo.preload posts, [comments: [:replies, :likes], authors: []]

   # Use a keyword list to customize how associations are queried
   posts = Repo.preload posts, [comments: from(c in Comment, order_by: c.published_at)]

   # Use a two-element tuple for a custom query and nested association definition
   query = from c in Comment, order_by: c.published_at
   posts = Repo.preload posts, [comments: {query, [:replies, :likes]}]
   ```

   ​

2. Ecto: https://robots.thoughtbot.com/preloading-nested-associations-with-ecto

   ```
   user = Blog.User
   |> where([user], user.id == ^user_id)
   |> join(:left, [u], _ in assoc(u, :posts))
   |> join(:left, [_, posts], _ in assoc(posts, :comments))
   |> preload([_, p, c], [posts: {p, comments: c}])
   |> Blog.Repo.one
   ```

   ​



Database with constraints: 

 Eg: create unique in migration and check in changset

```python

```



3. Create new field and reference to other models: 

   Eg:

   ```elixir
   schema "notifications" do
       field :type, :integer
       embeds_many :message, Message, on_replace: :delete
       field :frequency, RealEstate.Type.Frequency, default:      :daily
       field :sent, :boolean, default: false, null: false
       field :anttention_id, :integer

       embeds_many :data, Data, on_replace: :delete
       belongs_to :receiver,     User
       belongs_to :sender,       User
       field :message_objects, :any, virtual: true
       field :data_objects  , :any, virtual: true
       timestamps()
     end

     @doc false
     def changeset(%Notification{} = notification, attrs) do
       notification
       |> cast(attrs, [
         :receiver_id, :sender_id, :frequency, :message_objects,
         :data_objects, :anttention_id
       ])
       |> validate_required([])
     end
   end
   ```

   in migration: 

   ```elixir
   def change do
       create table(:notifications) do
         add :type, :integer
         add :receiver_id, references(:users, on_delete: :delete_all)
         add :sender_id, references(:users, on_delete: :delete_all)
         add :sent, :boolean, default: false, null: false
         add :message, :map
         add :frequency, :string

         timestamps()
       end
       create index(:notifications, [:receiver])
       create index(:notifications, [:frequency, :sender])
       create index(:notifications, [:frequency])

     end
   end

   ```

   ​

4. Create and update with many to many relationship:

5.  Compare and get exclude items between two list: 

   ```elixir
    # check and update condo
         Enum.filter([1,2,4], &(&1 in [2, 4, 7, 9, 10]))   
         # Result: [2,4]
         Enum.filter([1,2,4], &(&1 not in [2, 4, 7, 9, 10]))
         # Result: [9, 10]
   ```

   ​