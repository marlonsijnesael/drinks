<template>
  <v-card id="allbeers">
    <v-container>
      <v-layout row wrap class="pa-4">
        <v-flex class="xs12">
          <v-card-actions>
            <h1>Bestaande bieren</h1>
            <v-spacer></v-spacer>
            <v-btn color="primary" @click="changeModal('portal')"
              >Nieuw bier toevoegen</v-btn
            >
          </v-card-actions>
          <div class="d-sm-none">
            <p>
              Om bestaande data aan te passen, dient u gebruik te maken van een computer,
              laptop of tablet.
            </p>
          </div>

          <v-data-table
            class="d-none d-sm-block transparent"
            :key="update"
            :item-class="rowClass"
            :headers="headers"
            :items="drinks"
            :items-per-page="10"
            :search="search"
            :loading="loading"
            loading-text="Loading... Please"
            :footer-props="{
              firstIcon: 'mdi-arrow-collapse-left',
              lastIcon: 'mdi-arrow-collapse-right',
              prevIcon: 'mdi-chevron-left',
              nextIcon: 'mdi-chevron-right',
              'items-per-page-text': 'Dranken per pagina',
            }"
          >
            <template v-slot:top>
              <v-toolbar class="transparent" flat>
                <v-text-field
                  v-model="search"
                  append-icon="mdi-magnify"
                  label="Search"
                  single-line
                  hide-details
                ></v-text-field>
              </v-toolbar>

              <v-dialog v-model="dialog" max-width="500px">
                <v-card>
                  <v-form ref="form" autocomplete="off" v-model="valid" lazy-validation>
                    <v-container grid-list-md text-xs-center>
                      <v-layout row wrap>
                        <v-flex md12>
                          <v-text-field
                            hint="Hoe heet het nieuwe drankje?"
                            filled
                            required
                            outlined
                            autocomplete="off"
                            v-model="editedItem.name"
                            :counter="20"
                            :rules="nameRules"
                            label="Naam"
                            required
                            clearable
                          >
                          </v-text-field>
                        </v-flex>
                        <v-flex md6>
                          <v-text-field
                            hint="Hoeveel alcohol bevat de drank?"
                            filled
                            required
                            width="20%"
                            :type="'number'"
                            outlined
                            suffix="%"
                            v-model="editedItem.alcohol"
                            :rules="alcoholRules"
                            label="Alcohol percentage"
                            required
                            clearable
                          ></v-text-field>
                        </v-flex>
                        <v-flex md6>
                          <v-text-field
                            hint="Hoeveel kost de drank?"
                            suffix="€"
                            filled
                            required
                            outlined
                            v-model="editedItem.price"
                            :rules="priceRules"
                            label="Prijs"
                            required
                            clearable
                            @change="replaceComma()"
                          ></v-text-field>
                        </v-flex>
                        <v-flex md6>
                          <v-text-field
                            hint="Vul dit alleen in voor bier in fles."
                            filled
                            outlined
                            :type="'number'"
                            v-model="editedItem.stock"
                            label="Voorraad"
                            required
                            clearable
                          ></v-text-field>
                        </v-flex>
                        <v-flex md6>
                          <v-select
                            filled
                            required
                            :items="types"
                            label="Soort drank"
                            outlined
                            v-model="editedItem.drinkType"
                          ></v-select>
                        </v-flex>
                        <v-textarea
                          filled
                          required
                          no-resize
                          outlined
                          v-model="editedItem.description"
                          label="Beschrijving"
                          required
                          clearable
                          :counter="150"
                        ></v-textarea>
                      </v-layout>
                    </v-container>
                  </v-form>

                  <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="blue darken-1" text @click="close"> Cancel </v-btn>
                    <v-btn color="blue darken-1" text @click="save"> Save </v-btn>
                  </v-card-actions>
                </v-card>
              </v-dialog>
              <v-dialog v-model="dialogDelete" max-width="500px">
                <v-card color="#EDF6F9">
                  <v-card-title class="subtitle-1"
                    >Weet je zeker dat je dit item wil verwijderen?</v-card-title
                  >
                  <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="error" text @click="closeDelete">Cancel</v-btn>
                    <v-btn color="primary" text @click="deleteItemConfirm">OK</v-btn>
                    <v-spacer></v-spacer>
                  </v-card-actions>
                </v-card>
              </v-dialog>
            </template>

            <template v-slot:item.price="{ item }">
              {{ item.price.toFixed(2) }}
            </template>

            <template v-slot:item.stock="{ item }">
              <v-card-actions v-if="item.stock !== null">
                {{ item.stock }}
                <v-spacer></v-spacer>
                <v-icon small class="mr-2" @click="changeStock(item, 1)">
                  mdi-plus-box
                </v-icon>
                <v-icon
                  :disabled="item.stock === 0"
                  small
                  class="mr-2"
                  @click="changeStock(item, -1)"
                >
                  mdi-minus-box
                </v-icon>
              </v-card-actions>
              <v-card-actions v-else> Tap </v-card-actions>
            </template>

            <template v-slot:item.actions="{ item }">
              <v-icon small class="mr-2" @click="editItem(item)"> mdi-pencil </v-icon>
              <v-icon small @click="deleteItem(item)"> mdi-delete </v-icon>
            </template>
          </v-data-table>
        </v-flex>
      </v-layout>
    </v-container>
  </v-card>
</template>

<script>
import axios from "axios";
import qs from "qs";
export default {
  name: "DatabasePortal",
  created() {
    this.getPost();
  },
  components: {},
  data: () => ({
    update: 0,
    search: "",
    expanded: [],
    drinks: [],
    loading: false,
    singleExpand: true,
    dialog: false,
    dialogDelete: false,
    editedIndex: -1,
    valid: true,
    types: [
      "Speciaalbier",
      "Pils",
      "Sterke dranken",
      "0.0%",
      "Warme dranken",
      "Frisdranken",
      "Kleine kaart",
    ],
    nameRules: [
      (v) => !!v || "Vul een naam is alsjeblieft",
      (v) => (v && v.length <= 20) || "Vul een kortere naam in.",
    ],
    priceRules: [
      (v) => !!v || "Vul een prijs in alsjeblieft.",
      (v) => (v && v.length <= 6) || "Vul een geldige prijs in.",
    ],
    alcoholRules: [
      (v) => !!v || "vull een alcoholpercentage in.",
      (v) => (v < 100 && v >= 0) || "Vul een geldig alcohol percentage in",
    ],
    editedItem: {
      name: "",
      price: 0,
      alcohol: 0,
      type: 0,
      stock: 0,
    },
    defaultItem: {
      name: "",
      calories: 0,
      fat: 0,
      carbs: 0,
      protein: 0,
    },
    headers: [
      {
        text: "Drank",
        align: "start",
        sortable: false,
        value: "name",
        class: "text subtitle-1 font-weight-bold",
      },
      {
        text: "Alcohol %",
        value: "alcohol",
        class: "text subtitle-1 font-weight-bold",
      },
      {
        text: "Prijs",
        value: "price",
        class: "text subtitle-2 font-weight-bold",
      },
      {
        text: "Voorraad",
        value: "stock",
        class: "text subtitle-2 font-weight-bold",
      },
      {
        text: "ID",
        value: "_id",
        sortable: false,
        class: "text subtitle-2 font-weight-bold",
      },
      {
        text: "Actions",
        value: "actions",
        sortable: false,
      },
    ],
  }),
  computed: {},
  methods: {
    changeStock(drink, n) {
      if (drink.stock + n >= 0) {
        drink.stock += n;
        this.put(drink._id, drink);
      }
    },
    async put(id, data) {
      var optionAxios = {
        headers: {
          "Content-Type": "application/x-www-form-urlencoded",
        },
      };

      data = qs.stringify({
        name: data.name,
        description: data.description,
        alcohol: data.alcohol,
        price: data.price,
        drinkType: data.drinkType,
        stock: data.stock,
      });

      axios
        .put("https://zuidapi.herokuapp.com/posts/" + id, data, optionAxios)
        .then((res) => {
          this.getPost();
        });
    },

    async getPost() {
      let get = await axios.get("https://zuidapi.herokuapp.com/posts/", {
        headers: {},
      });

      this.drinks = [];
      for (let i in get.data) {
        this.drinks.push(get.data[i]);
      }
    },
    changeModal(modal) {
      let event = new CustomEvent("changeModal", {
        detail: modal,
      });
      window.dispatchEvent(event);
    },
    rowClass(item) {
      ////console.log("item",item);
      return "style-2";
    },
    editItem(item) {
      console.log("item", item);
      this.editedIndex = this.drinks.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialog = true;
    },
    replaceComma() {
      const numericalChar = new Set([
        ",",
        ".",
        "0",
        "1",
        "2",
        "3",
        "4",
        "5",
        "6",
        "7",
        "8",
        "9",
      ]);
      if (this.editedItem.price !== "") {
        this.editedItem.price = this.editedItem.price
          .split("")
          .filter((char) => numericalChar.has(char))
          .join("");
        this.editedItem.price = this.editedItem.price.replace(",", ".");
      }
    },

    deleteItem(item) {
      this.editedIndex = this.drinks.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialogDelete = true;
    },

    deleteItemConfirm() {
      var optionAxios = {
        headers: {
          "Content-Type": "application/x-www-form-urlencoded",
        },
      };

      let apiString =
        "https://zuidapi.herokuapp.com/posts/" + this.drinks[this.editedIndex]._id;
      axios.delete(apiString).then((response) => {
        let event = new Event("refreshDb");
        this.getPost();
      });
      this.closeDelete();
    },

    close() {
      this.dialog = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      });
    },

    closeDelete() {
      this.dialogDelete = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      });
    },

    save() {
      if (this.editedIndex > -1) {
        this.put(this.drinks[this.editedIndex]._id, this.editedItem);
      } else {
        this.drinks.push(this.editedItem);
      }
      this.close();
    },
  },
};
</script>

<style>
tbody tr:nth-of-type(odd) {
  background-color: rgba(0, 0, 0, 0.05);
}
</style>
