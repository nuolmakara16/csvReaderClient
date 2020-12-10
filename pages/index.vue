<template>
  <v-container>
    <v-row justify="center">
      <v-col cols="12" sm="6" md="8">
        <v-file-input
          v-model="chosenFile"
          label="Click here to select a csv file"
          outlined
          accept=".csv"
          color="deep-purple accent-4"
          show-size
          prepend-icon="mdi-paperclip"
          :error="errors"
          :error-messages="error_message"
        >
          <template v-slot:selection="{ index, text }">
            <v-chip
              v-if="index < 2"
              color="deep-purple accent-4"
              dark
              label
              small
            >
              {{ text }}
            </v-chip>
          </template>
        </v-file-input>
      </v-col>
      <v-col cols="12" sm="6" md="4">
        <v-btn
          outlined
          color="primary"
          :loading="converting"
          large
          @click="readFile"
          >Convert</v-btn
        >
      </v-col>
    </v-row>
    <v-text-field
      v-model="search"
      counter="25"
      label="Search"
      outlined
      append-icon="mdi-magnify"
    ></v-text-field>
    <v-dialog v-model="dialogDelete" max-width="500px">
      <v-card>
        <v-card-title class="headline"
          >Are you sure you want to delete this item?</v-card-title
        >
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="blue darken-1" text @click="closeDelete">Cancel</v-btn>
          <v-btn color="red darken-1" text @click="deleteItemConfirm">OK</v-btn>
          <v-spacer></v-spacer>
        </v-card-actions>
      </v-card>
    </v-dialog>
    <v-data-table
      :headers="tableData.headers"
      :items="tableData.data"
      class="elevation-1"
      :search="search"
    >
      <template v-slot:item.actions="{ item }">
        <v-icon color="blue" small class="mr-2" @click="editItem(item)">
          mdi-pencil
        </v-icon>
        <v-icon color="red" small @click="deleteItem"> mdi-delete </v-icon>
      </template>
    </v-data-table>
  </v-container>
</template>

<script>
export default {
  data() {
    return {
      converting: false,
      error_message: null,
      errors: false,
      chosenFile: null, // <- initialize the v-model prop
      search: '',
      dialog: false,
      dialogDelete: false,
      editedIndex: -1,
      defaultItem: {},
      tableData: {
        data: [],
        headers: [
          { text: 'Hiragana', value: 'hiragana' },
          { text: 'katagana', value: 'katagana' },
          { text: 'Kanji', value: 'kanji' },
          { text: 'KH-Sample', value: 'kh_sample' },
          { text: 'JP-Sample', value: 'jp_sample' },
          { text: 'Action', value: 'actions' },
        ],
      },
    }
  },
  computed: {
    formTitle() {
      return this.editedIndex === -1 ? 'New Item' : 'Edit Item'
    },
  },
  watch: {
    dialog(val) {
      val || this.close()
    },
    dialogDelete(val) {
      val || this.closeDelete()
    },
  },
  methods: {
    editItem(item) {
      window.alert(item)
    },
    deleteItem(item) {
      this.editedIndex = this.tableData.data.indexOf(item)
      this.editedItem = Object.assign({}, item)
      this.dialogDelete = true
      console.log(`Delete Item`)
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
        Object.assign(this.tableData.data[this.editedIndex], this.editedItem)
      } else {
        this.tableData.data.push(this.editedItem)
      }
      this.close()
    },
    deleteItemConfirm() {
      this.tableData.data.splice(this.editedIndex, 1)
      this.closeDelete()
    },
    readFile() {
      this.converting = true
      if (!this.chosenFile) {
        this.data = 'No File Chosen'
      }
      const reader = new FileReader()
      // Use the javascript reader object to load the contents
      // of the file in the v-model prop
      reader.readAsText(this.chosenFile)
      reader.onload = () => {
        try {
          const result = []
          const lines = reader.result.split('\n')
          const headers = lines[0].split(',')
          for (let i = 1; i < lines.length - 1; i++) {
            const obj = {}
            const currentline = lines[i].split(',')
            for (let j = 0; j < currentline.length; j++) {
              obj[headers[j].replace('\r', '')] = currentline[j].replace(
                '\r',
                ''
              )
            }
            result.push(obj)
          }
          this.tableData.data = result
          this.chosenFile = null
          this.error_message = null
          this.errors = false
        } catch (err) {
          this.errors = true
          this.error_message = "CSV'S Format is not valid!"
        } finally {
          setTimeout(() => {
            this.converting = false
          }, 2000)
        }
      }
    },
  },
}
</script>
