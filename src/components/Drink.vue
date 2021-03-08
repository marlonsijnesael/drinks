<template>
    <v-card  :color="color(hex)" outlined class="drink">
        <v-layout wrap class="">
            <v-flex xs8 >
                <v-card-title id="beerName" style=" padding-bottom: 0;font-size: 20px ;color:white " class="subtitle-1 font-weight-bold">
                    <v-icon class="drinkIcon" color="#D19C1D">mdi-bottle-wine</v-icon>
                    <svg xmlns="http://www.w3.org/2000/svg" width="100%" height="100%" viewBox='0 0 400 64'>
                        <text fill="#202020"   class="svgtext" font-size="36"   x='5' y="75%" >
                        {{currentDrink.name}}
                        </text>
                    </svg>
                </v-card-title>
            </v-flex>
            <v-flex no-gutters xs4>
                <v-card-title style="padding-bottom:0;" class="subtitle-1 font-weight-bold">
                    <svg xmlns="http://www.w3.org/2000/svg" width="100%" height="100%" viewBox='0 0 200 64'>
                        <text fill="#202020"   class="svgtext" font-size="42"   x='48' y="75%" >
                        {{currentDrink.price.toFixed(2)}}
                        </text>
                        <text fill="#D19C1D"  class="svgtext" font-size="48"   x='0' y="75%" >
                        €</text>
                    </svg>
                </v-card-title>
            </v-flex>
            <v-flex xs12 no-gutters style="padding-bottom: 0;">
                <v-card-title style=" padding-bottom: 0;font-size: 20px ;color:white " class="subtitle-1 font-weight-bold">
                    <v-icon class="drinkIcon" color="#D19C1D">mdi-glass-wine</v-icon>

                    <svg xmlns="http://www.w3.org/2000/svg" width="100%" height="100%" viewBox='0 0 400 64'>
                        <text fill="#202020"  class="svgtext" font-size="22"   x='5' y="64%" >
                        {{currentDrink.alcohol}}%
                        </text>
                    </svg>
                </v-card-title>
            </v-flex>
        </v-layout>

        <v-expand-transition>
            <v-layout wrap class="">
                <div id="description" class="text-justify" v-show="open">
                    <v-flex xs12>
                        <v-divider></v-divider>
                        <v-card-text style="color: #202020; font-weight: 500">
                            {{currentDrink.description}}
                        </v-card-text>
                    </v-flex>
                </div>
            </v-layout>
        </v-expand-transition>

        <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn icon>
                <v-icon>{{ open ? 'mdi-chevron-up' : 'mdi-chevron-down' }}</v-icon>
            </v-btn>
            </v-layout>
            <v-spacer></v-spacer>
        </v-card-actions>
        <div class="strip_tri"></div>
    </v-card>
</template>

<script>
export default {
    name: 'Drink',
    props: ['currentDrink', 'open', 'hex','id'],
    created() {},
    data: () => ({
        show: false,
        cardClass: "drink",
    }),
    methods: {
        openPage() {},
        color(n) {
            if (n === 0) {
                return "#508484";
            }
            return "#EDF6F9";
        },
        darkMode(n) {
            return n === 0;
        }
    },
}
</script>

<style scoped>
.v-card__title {
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen-Sans, Ubuntu, Cantarell, "Helvetica Neue", sans-serif;
}
.svgtext {
    color:white ;;
}

svg {
  width: 75%;
}
.v-card__actions {
    padding-top: 4px;
    padding-bottom: 4px;
}

#beerName {
    text-align: center;
}

#borderDown {
    min-width: 100%;
    min-height: 4px;
    background-color: black;
}

#description {
    padding: 0% 2% 0% 2%;
    width: 100%;
    min-height: 10px;
}
.drinkIcon{
    opacity:80%;
}
div .v-card {
    border-radius: 4px 4px 4px 4px;
    overflow: hidden;
}
.title{
    font-size: 8px;
    font-size: 3.75vw;
}
div .v-card::after {
    content: '';
    position: absolute;
    display: block;
    width: 50px;
    height: 0;
    box-shadow: 0 10px 0 1px #D19C1D, 0 17px 0 1px #D19C1D, 0 24px 0 1px #D19C1D;
    transform: rotate(-45deg) scale(0.5);
    right: -13px;
    bottom: 10px;
}

.infoField {
    font-size: 90%;
}

.drink {
    margin-top: 10px;
    border-left: 10px;
}

text{
    color:white;
}

.even {
    background-color: #83C5BE;
}

.uneven {
    background-color: #83C5BE;
}
.v-card__actions{
    padding: 0;
    margin-top: -24px;
}
#type {
    background-color: black;
    height: 100%;
}
</style>


