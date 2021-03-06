<template>
    <div>
        <v-toolbar flat color="white">
            <v-toolbar-title>Users</v-toolbar-title>
            <v-spacer></v-spacer>
            <v-dialog v-model="dialog" max-width="500px">
                <template v-slot:activator="{ on }">
                    <v-btn color="primary" dark class="mb-2" v-on="on">New Item</v-btn>
                </template>
                <v-card>
                    <v-card-title>
                        <span class="headline">{{ formTitle }}</span>
                    </v-card-title>

                    <v-card-text>
                        <v-container grid-list-md>
                            <v-layout wrap>
                                <v-flex xs12>
                                    <v-text-field v-model="editedItem.name" label="Name"></v-text-field>
                                </v-flex>
                                <v-flex xs12>
                                    <v-text-field v-model="editedItem.email" label="Email"></v-text-field>
                                </v-flex>
                                <v-flex xs12>
                                    <v-text-field v-model="editedItem.phonenumber" label="Phone Number"></v-text-field>
                                </v-flex>
                                <v-flex xs12 class="text-xs-center text-sm-center text-md-center text-lg-center">
                                    <img :src="imageUrl" height="150" v-if="imageUrl"/>
                                </v-flex>
                                <v-flex xs12 class="text-xs-center text-sm-center text-md-center text-lg-center">
                                    <v-btn color="primary" dark class="mb-2" @click='pickFile'>Upload File</v-btn>
                                    <input
                                            type="file"
                                            style="display: none"
                                            ref="image"
                                            accept="image/*"
                                            @change="onFilePicked"
                                    >
                                </v-flex>
                            </v-layout>
                        </v-container>
                        <span v-if="errorMessage" class="red--text">{{ errorMessage }}</span>
                    </v-card-text>

                    <v-card-actions>
                        <v-spacer></v-spacer>
                        <v-btn color="blue darken-1" flat @click="close">Cancel</v-btn>
                        <v-btn color="blue darken-1" flat @click="save">Save</v-btn>
                    </v-card-actions>
                </v-card>
            </v-dialog>
        </v-toolbar>
        <v-card>
            <v-card-title>
                Nutrition
                <v-spacer></v-spacer>
                <v-text-field
                    v-model="search"
                    append-icon="search"
                    label="Search"
                    single-line
                    hide-details
                ></v-text-field>
            </v-card-title>
            <v-data-table
                :headers="headers"
                :items="users"
                class="elevation-1"
                :search="search"
            >
                <template v-slot:items="props">
                    <td>{{ props.item.name }}</td>
                    <td>{{ props.item.email }}</td>
                    <td>{{ props.item.phonenumber }}</td>
                    <td>
                        <v-img
                                :src="props.item.image"
                                aspect-ratio="1"
                                class="grey lighten-2 ma-2"
                                height="100"
                        >
                            <template v-slot:placeholder>
                                <v-layout
                                    fill-height
                                    align-center
                                    justify-center
                                    ma-0
                                >
                                    <v-progress-circular indeterminate color="grey lighten-5"></v-progress-circular>
                                </v-layout>
                            </template>
                        </v-img>
                    </td>
                    <td class="justify-center layout px-0">
                        <v-icon
                                small
                                class="mr-2"
                                @click="editItem(props.item)"
                        >
                            edit
                        </v-icon>
                        <v-icon
                                small
                                @click="deleteItem(props.item)"
                        >
                            delete
                        </v-icon>
                    </td>
                </template>
                <v-alert v-slot:no-results :value="true" color="error" icon="warning">
                    Your search for "{{ search }}" found no results.
                </v-alert>
            </v-data-table>
        </v-card>
    </div>
</template>
<script>
  export default {
    data: () => ({
      dialog: false,
      search: '',
      headers: [
        { text: 'Name', value: 'name' },
        { text: 'Email', value: 'email' },
        { text: 'Phone Number', value: 'phonenumber' },
        { text: 'Image', value: 'image' },
        { text: 'Actions', value: 'name', sortable: false }
      ],
      editedIndex: -1,
      editedItem: {
        name: '',
        email: '',
        phonenumber: '',
        image: ''
      },
      defaultItem: {
        name: '',
        email: '',
        phonenumber: '',
        image: ''
      },
      imageUrl: '',
      errorMessage: ''
    }),

    computed: {
      users () {
        return this.$store.getters.loadedUsers
      },
      formTitle () {
        return this.editedIndex === -1 ? 'New Item' : 'Edit Item'
      }
    },

    watch: {
      dialog (val) {
        val || this.close()
      }
    },

    methods: {
      pickFile () {
        this.$refs.image.click ()
      },

      onFilePicked (e) {
        const files = e.target.files
        if(files[0] !== undefined) {
          const fr = new FileReader ()
          fr.readAsDataURL(files[0])
          fr.addEventListener('load', () => {
            this.imageUrl = fr.result
          })
        } else {
          this.imageUrl = ''
        }
      },

      editItem (item) {
        this.editedIndex = this.users.indexOf(item);
        this.editedItem = Object.assign({}, item);
        this.imageUrl = this.editedItem.image;
        this.dialog = true
      },

      deleteItem (item) {
        this.$store.dispatch('deleteUser', {id: item.id});
      },

      close () {
        this.dialog = false
        this.errorMessage = ''
        setTimeout(() => {
          this.editedItem = Object.assign({}, this.defaultItem)
          this.editedIndex = -1
        }, 300)
      },

      save () {
        if (!this.editedItem.name || !this.editedItem.email || !this.editedItem.phonenumber) {
          this.errorMessage = 'Name, email and phone numbers are required.';
          return;
        }
        if (this.editedIndex > -1) {
          const updateObj = {
            id: this.editedItem.id,
            name: this.editedItem.name,
            email: this.editedItem.email,
            phonenumber: this.editedItem.phonenumber
          };
          if (this.imageUrl !== this.editedItem.image)
            updateObj.image = this.imageUrl;
          this.$store.dispatch('updateUser', updateObj);
        } else {
          if (!this.imageUrl) {
            this.errorMessage = 'Image is required.';
            return;
          }
          this.$store.dispatch('createUser', {
            name: this.editedItem.name,
            email: this.editedItem.email,
            phonenumber: this.editedItem.phonenumber,
            image: this.imageUrl
          });
        }
        this.close()
      }
    },
    created () {
      this.$store.dispatch('loadUsers')
    }
  }
</script>
