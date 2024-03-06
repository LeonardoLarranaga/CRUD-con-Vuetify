<template>
  <v-data-table 
    :headers="headers" 
    :items="posts" 
    :items-per-page="-1" 
    :sort-by="[{ key: 'author', order: 'asc' }]"
    >

    <template v-slot:top>
      <v-toolbar color="indigo-darken-2">
        <v-toolbar-title color="indigo-darken-2">CRUD with Vuetify</v-toolbar-title>
        <v-divider class="mx-4" inset vertical></v-divider>

        <v-dialog v-model="dialog" max-width="500px">
          <template v-slot:activator="{ props }">
            <v-btn prepend-icon="mdi-plus-circle-multiple" variant="tonal" elevation="10" color="white" v-bind="props">
              New Post
            </v-btn>
          </template>
          <v-card>
            <v-card-title>
              <v-icon class="text-h5 margin" style="margin-top: -5px; margin-right: 8px;">{{ this.editedIndex === -1 ? "mdi-plus-circle-multiple" : "mdi-pencil-circle"}} </v-icon>
              <span class="text-h5 font-weight-bold">{{this.editedIndex === -1 ? "New Post" : "Edit Post"}}</span>
            </v-card-title>

            <v-card-text>
              <v-container>
                <v-row>
                  <v-col cols="12"  sm="6">
                    <v-text-field v-model="editedItem.author" label="Author"/>
                  </v-col>
                  <v-col cols="12" sm="6">
                    <v-text-field v-model="editedItem.title" label="Title"/>
                  </v-col>
                  <v-col cols="12">
                    <v-textarea v-model="editedItem.post" label="Post Content" clearable/>
                  </v-col>
                </v-row>
              </v-container>
            </v-card-text>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="indigo-darken-2" variant="text" @click="close">
                Cancel
              </v-btn>
              <v-btn color="indigo-darken-2" variant="tonal" @click="save">
                POST
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
        <v-dialog v-model="dialogDelete" max-width="350px">
          <v-card>
            <v-card-title>
              <p class="text-h5 font-weight-bold" style="white-space: pre-wrap; text-align: center;">Are you sure you want to delete this post?</p>
            </v-card-title>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue-darken-1" variant="text" @click="closeDelete">Cancel</v-btn>
              <v-btn color="blue-darken-1" variant="tonal" @click="deleteItemConfirm">OK</v-btn>
              <v-spacer></v-spacer>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-toolbar>
    </template>

    <template v-slot:item.actions="{ item }">
      <v-icon class="me-2" size="small" @click="editItem(item)">
        mdi-pencil
      </v-icon>
      <v-icon size="small" @click="deleteItem(item)"> mdi-delete </v-icon>
    </template>
  </v-data-table>
</template>

<script>
export default {
    data: () => ({
      dialog: false,
      dialogDelete: false,
      headers: [
        {
          title: "Author",
          sortable: false,
          key: 'author',
        },
        { title: 'Title', 
          key: 'title', 
          sortable: false 
        },
        { 
          title: 'Post', 
          key: 'post', 
          sortable: false 
        },
        { 
          title: 'Actions', 
          key: 'actions', 
          sortable: false 
        },
      ],
      posts: [],
      editedIndex: -1,
      editedItem: {
        author: "",
        title: "",
        post: ""
      },
      defaultItem: {
        author: "",
        title: "",
        post: ""
      },

    }),

    watch: {
      dialog(val) {
        val || this.close()
      },
      dialogDelete(val) {
        val || this.closeDelete()
      },
    },

    mounted() {
      this.fetchPosts()
    },

    methods: {
      async fetchPosts() {
        try {
          const postsResponse = await fetch("https://jsonplaceholder.typicode.com/posts")
          let posts = await postsResponse.json()
          posts = posts.slice(0, 10)

          const authorsResponse = await fetch("https://jsonplaceholder.typicode.com/users")
          const authors = await authorsResponse.json()
          
          this.posts = authors.map((author, i) => {
            return {
              author: author.name,
              title: posts[i].title,
              post: posts[i].body,
            };
          }) 
        } catch (error) {
          console.log(error)
        }
      },

      async createPost() {
        try {
          const response = await fetch("https://jsonplaceholder.typicode.com/posts", {
            method: "POST",
            body: JSON.stringify(this.editedItem),
            headers: { "Content-type": "application/json; charset=UTF-8" }
          })

          const newPost = await response.json()
          this.posts.push(newPost)
        } catch (error) {
          console.log(error)
        }
      },

      async editPost() {
        try {
          const response = await fetch("https://jsonplaceholder.typicode.com/posts/1", {
            method: "PUT",
            body: JSON.stringify(this.editedItem),
            headers: { "Content-type": "application/json; charset=UTF-8" }
          })

          const editedPost = await response.json()
          delete editedPost.id
          Object.assign(this.posts[this.editedIndex], editedPost)
          this.close()
        } catch (error) {
          console.log(error)
        }
      },

      editItem(item) {
        this.editedIndex = this.posts.indexOf(item)
        this.editedItem = Object.assign({}, item)
        this.dialog = true
      },

      deleteItem(item) {
        this.editedIndex = this.posts.indexOf(item)
        this.editedItem = Object.assign({}, item)
        this.dialogDelete = true
      },

      async deleteItemConfirm() {
        try {
          const response = await fetch("https://jsonplaceholder.typicode.com/posts/1", {
            method: "DELETE",
          })

          if (response.ok) {
            this.posts.splice(this.editedIndex, 1)
          }

        } catch (error) {
          console.log(error)
        }

        this.closeDelete()
      },

      close() {
        this.dialog = false
        this.$nextTick(() => {
          this.editedItem = Object.assign({}, this.defaultItem)
          this.editedIndex = -1
        })
      },

      closeDelete() {
        this.dialogDelete = false
        this.$nextTick(() => {
          this.editedItem = Object.assign({}, this.defaultItem)
          this.editedIndex = -1
        })
      },

      save() {
        if (this.editedIndex > -1) {
          this.editPost()
        } else {
          this.createPost()
          this.close()
        }
      },
    },
  }
</script>
