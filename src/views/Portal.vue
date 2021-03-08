<template>
  <div id="portal">
    <v-card v-if="currentModal === 'portal'" id="beerForm">
      <v-container>
        <v-layout row wrap class="pa-4">
          <v-flex class="xs12">
            <v-card-actions>
              <h1>Nieuwe drank toevoegen</h1>
            </v-card-actions>

            <v-form
              ref="form"
              autocomplete="off"
              v-model="valid"
              lazy-validation
            >
              <v-container grid-list-md text-xs-center>
                <v-layout row wrap>
                  <v-flex md12>
                    <v-text-field
                      hint="Hoe heet het nieuwe drankje?"
                      filled
                      required
                      outlined
                      autocomplete="off"
                      v-model="beer.name"
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
                      v-model="beer.alcohol"
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
                      v-model="beer.price"
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
                      v-model="beer.stock"
                      label="Voorraad"
                      required
                      clearable
                      @change="replaceComma()"
                    ></v-text-field>
                  </v-flex>
                  <v-flex md6>
                    <v-select
                      filled
                      required
                      :items="types"
                      label="Soort drank"
                      outlined
                      v-model="beer.type"
                    ></v-select>
                  </v-flex>
                  <v-textarea
                    filled
                    required
                    no-resize
                    outlined
                    v-model="beer.description"
                    label="Beschrijving"
                    required
                    clearable
                    :counter="150"
                  ></v-textarea>
                </v-layout>
              </v-container>
            </v-form>
            <v-card-actions>
              <div></div>
              <v-spacer></v-spacer>
              <v-btn width="30%" @click="cancel()" dark color="#E29578"
                >Annuleer</v-btn
              >

              <v-btn
                dark
                width="20%"
                color="#83C5BE"
                :disabled="!valid"
                @click="postPost()"
                >Voeg toe</v-btn
              >
            </v-card-actions>
          </v-flex>
        </v-layout>
      </v-container>
    </v-card>

    <databasePortal v-if="currentModal === 'details'"></databasePortal>
    <v-alert
      :value="alert.show"
      dark
      border="top"
      icon="mdi-home"
      transition="scale-transition"
      :class="alert.class"
      id="alert"
    >
      {{ alert.text }}
    </v-alert>
  </div>
</template>

<script>
import axios from 'axios'
import DatabasePortal from '@/components/DatabasePortal'
import qs from 'qs'

axios.defaults.headers.common = {
  'Content-Type': 'application/json'
}

export default {
  name: 'Portal',
  components: {
    DatabasePortal
  },
  mounted() {
    if (alert) {
      this.hide_alert()
    }
  },

  data: () => ({
    alert: {
      show: false,
      class: '',
      text: ''
    },
    valid: true,
    showFormModal: false,
    alertText: 'Drank succesvol toegevoed',
    currentModal: 'details',
    types: [
      'Speciaalbier',
      'Pils',
      'Sterke dranken',
      '0.0%',
      'Warme dranken',
      'Frisdranken',
      'Kleine kaart'
    ],
    beer: {
      name: '',
      description: '',
      alcohol: '',
      price: '',
      type: '',
      stock: ''
    },
    nameRules: [
      (v) => !!v || 'Vul een naam is alsjeblieft',
      (v) => (v && v.length <= 20) || 'Vul een kortere naam in.'
    ],
    priceRules: [
      (v) => !!v || 'Vul een prijs in alsjeblieft.',
      (v) => (v && v.length <= 6) || 'Vul een geldige prijs in.'
    ],
    alcoholRules: [
      (v) => !!v || 'vull een alcoholpercentage in.',
      (v) => (v < 100 && v >= 0) || 'Vul een geldig alcohol percentage in'
    ],
    select: null,
    items: ['Item 1', 'Item 2', 'Item 3', 'Item 4'],
    checkbox: false,
    drinks: [],
    pageNumber: 0
  }),
  created() {
    window.addEventListener('changeModal', (evt) => {
      this.currentModal = evt.detail
    })
  },
  computed: {
    paginatedData() {
      const start = this.pageNumber * this.size - this.size,
        end = start + this.size
      return this.drinks.slice(start, end)
    },
    pageCount() {
      let l = this.listData.length,
        s = this.size
      return Math.ceil(l / s) - 1
    }
  },

  methods: {
    cancel() {
      this.currentModal = 'details'
      this.reset()
    },
    nextPage(page) {
      this.pageNumber = page
    },

    postPost() {
      var optionAxios = {
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded'
        }
      }

      var data = qs.stringify({
        name: this.beer.name,
        description: this.beer.description,
        alcohol: this.beer.alcohol,
        price: this.beer.price,
        drinkType: this.beer.type,
        stock: this.beer.stock
      })
      // data = JSON.stringify(data);

      var bodyFormData = new FormData()
      bodyFormData.append('name', this.beer.name)
      bodyFormData.append('price', this.beer.price)
      bodyFormData.append('description', this.beer.description)
      bodyFormData.append('alcohol', this.beer.alcohol)

      this.validate()
      axios
        .post('https://zuidapi.herokuapp.com/posts', data, optionAxios)
        .then((response) => {
          //console.log("response", response)
          this.reset()

          this.alert.show = true
          if (response.data.message === undefined) {
            this.alert.text = 'Nieuwe drank toegevoegd!'
            this.alert.class = 'alertSucces'
            this.currentModal = 'details'
          } else {
            this.alert.text = 'Er is iets fout gegaan.'
            this.alert.class = 'alertError'
          }
        })
        .catch((e) => {
          //console.log(e);
        })
    },

    replaceComma() {
      //console.log(typeof this.beer.price);
      const numericalChar = new Set([
        ',',
        '.',
        '0',
        '1',
        '2',
        '3',
        '4',
        '5',
        '6',
        '7',
        '8',
        '9'
      ])
      if (this.beer.price !== '') {
        this.beer.price = this.beer.price
          .split('')
          .filter((char) => numericalChar.has(char))
          .join('')
        this.beer.price = this.beer.price.replace(',', '.')
      }
    },
    validate() {
      this.$refs.form.validate()
    },
    reset() {
      this.$refs.form.reset()
    },
    resetValidation() {
      this.$refs.form.resetValidation()
    },
    hide_alert: function(event) {
      window.setInterval(() => {
        this.alert.show = false
      }, 4000)
    }
  }
}
</script>

<style scoped>
.alertError {
  background-color: red;
}

.alertSucces {
  background-color: green;
}

#alert {
  width: 300px;
  display: block;
  margin-left: auto;
  margin-right: auto;
}

h1 {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto,
    Oxygen-Sans, Ubuntu, Cantarell, 'Helvetica Neue', sans-serif;
}

#title {
  display: flex;
  justify-content: center;
  align-items: center;
}

.circle {
  border-radius: 50%;
  width: 200px;
  height: 200px;
  background-color: #001314;
  display: block;
  margin-left: auto;
  margin-right: auto;
  /* width and height can be anything, as long as they're equal */
}

.circle_content {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 200px;
  color: white;
}

#portal {
  width: 100vw;
  min-height: 100%;
  background-color: #092327;
}

#beerForm {
  width: 95vw;
  max-width: 1000px;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  background-color: #edf6f9;
}

#allbeers {
  width: 95vw;
  min-height: 75vh;
  max-width: 1000px;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  background-color: #edf6f9;
}
</style>
