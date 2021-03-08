<template>
  <div id="content">
    <v-container>
      <v-select
        id="select"
        solo
        hide-selected
        :menu-props="{ maxHeight: 720, top: true, overflowY: false }"
        :items="drinkTypes"
        label="Ons assortiment"
        v-model="filterName"
      ></v-select>

      <div v-if="dataLoaded">
        <div>
          <div
            v-for="(p, index) in paginatedData"
            :key="p.name"
            @click="openDrink(p.name)"
          >
            <Drink
              :currentDrink="p"
              :open="openName === p.name"
              :hex="cardColor(index)"
              :id="index"
            ></Drink>
          </div>
          <div class="pageBtns">
            <v-btn
              :disabled="pageNumber === 1"
              class="pagebtn"
              @click="decrementPagent"
              ><v-icon>mdi-chevron-left</v-icon></v-btn
            >
            <v-btn
              :disabled="pageNumber === paginationLength"
              class="pagebtn"
              @click="incrementPage"
              ><v-icon>mdi-chevron-right</v-icon></v-btn
            >
          </div>
        </div>
      </div>
      <div v-else>
        <v-skeleton-loader
          class="pa skel"
          type="list-item-two-line"
        ></v-skeleton-loader>
        <v-skeleton-loader
          class="pa skel"
          type="list-item-two-line"
        ></v-skeleton-loader>
        <v-skeleton-loader
          class="pa skel"
          type="list-item-two-line"
        ></v-skeleton-loader>
      </div>
    </v-container>
  </div>
</template>

<script>
import Drink from '@/components/Drink.vue'
import axios from 'axios'
export default {
  name: 'Bierlist',
  components: {
    Drink
  },
  async created() {
    let data = await this.getBeers()

    let result = []
    for (let i in data) {
      result.push(data[i])
    }

    this.listData = result
  },

  data: () => ({
    fab: false,
    pageNumber: 1,
    checked: true,
    show: false,
    openName: '',
    size: 5,
    filter: true,
    filterName: 'Ons assortiment',
    currentList: '',
    drinkTypes: [
      'Ons assortiment',
      'Speciaalbier',
      'Pils',
      'Sterke dranken',
      '0.0%',
      'Warme dranken',
      'Frisdranken',
      'Kleine kaart'
    ],
    listData: [],
    attrs: {}
  }),
  methods: {
    async getBeers() {
      let get = await axios.get('https://zuidapi.herokuapp.com/posts/', {
        headers: {}
      })
      return get.data
    },
    openDrink(drink) {
      if (this.openName === drink) {
        this.openName = ''
      } else {
        this.openName = drink
      }
    },
    setFilter(type) {
      this.filterName = type
      this.pageNumber = 1
    },
    nextPage(page) {
      this.pageNumber = page
    },
    incrementPage() {
      if (this.pageNumber < this.paginationLength) {
        this.pageNumber++
        this.$vuetify.goTo('#select')
      }
    },
    decrementPagent() {
      if (this.pageNumber > 1) {
        this.pageNumber--
        this.$vuetify.goTo('#select')
      }
    },
    filterType(dataset, type) {
      return dataset.filter(function(el) {
        return el.drinkType.toLowerCase() === type.toLowerCase() && el.stock > 0
      })
    },
    cardColor(index) {
      if (index % 2 === 0) {
        console.log('even')
        return 0
      }
      console.log(1)
      return 1
    },
    onScroll(e) {
      if (typeof window === 'undefined') return
      const top = window.pageYOffset || e.target.scrollTop || 0
      this.fab = top > 20
    },
    toTop() {
      this.$vuetify.goTo(0)
    }
  },

  computed: {
    dataLoaded() {
      return this.listData.length > 0
    },
    filteredData() {
      this.currentList = this.listData.filter((el) => {
        if (el.stock === 0) return false
        if (this.filterName === 'Ons assortiment') return true
        return el.drinkType.toLowerCase() === this.filterName.toLowerCase()
      })
      return this.currentList
    },

    paginationLength() {
      const length = this.filteredData.length
      let pages = Math.ceil(length / this.size)
      console.log('pages', pages)
      return pages
    },
    paginatedData() {
      const start = this.pageNumber * this.size - this.size,
        end = start + this.size
      console.log('wow', this.filteredData.slice(start, end))
      return this.currentList.slice(start, end)
    },
    pageCount() {
      let l = this.listData.length,
        s = this.size
      console.log(this.listData.length)
      return Math.ceil(l / s) - 1
    }
  }
}
</script>

<style scoped>
.skel {
  padding: 10px 0px 0px 0px;
}
.pageBtns {
  padding: 20px;
}
.pagebtn:nth-child(1) {
  position: relative;
  width: 32%;
  margin-left: 16%;
  float: left;
}
.pagebtn:nth-child(2) {
  position: relative;
  width: 32%;
  margin-right: 16%;
  float: right;
}
#content {
  width: 100vw;
  max-width: 900px;
  overflow-x: hidden;
  width: 100vw;
  background-color: #092327;
  backdrop-filter: blur(8px);
  margin: 0 auto;
  position: relative;
}

#pagination > .v-pagination__item {
  background-color: black;
}
</style>
