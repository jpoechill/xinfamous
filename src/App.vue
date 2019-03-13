<template>
<!-- align-items-center -->
  <div id="app" class="d-flex pt-5 h-100">
    <div class="w-100">
      <div class="container">
        <div class="row">
          <div class="col-lg-6">
              <a href="#">xInfamous
              (<transition name="slide-fade" mode="out-in">
                <span :key="currInfamyLevel">{{ currInfamyLevel }}</span>
              </transition>)</a><br>
              <input v-on:keyup.enter="sayHello" ref="el" v-model="handle" type="text" placeholder="@PSNusername" autofous>
          </div>
          <div class="col-md-6 pl-5 text-right">
            <div class="d-inline-block text-left">
              Current Infamy <br>
              Points to Next <br>
              Points to Reset
            </div>
            <div class="d-inline-block w-25 pl-2">
              <transition name="slide-fade" mode="out-in">
                <div :key="currInfamy" class="text-right">
                  {{ currInfamy }}<br>
                  {{ pointsToNext }}<br>
                  {{ pointsToReset }}
                </div>
              </transition>
            </div>
          </div>
        </div>
      </div>
      <section>
        <div class="container pt-4">
          <div class="row pt-3">
            <div v-for="(lastGames, index) in allGamesSorted" :key="index" class="col-md-4 pb-3">
              <div v-if="index < 12">
                <div v-if="allGamesSorted[0] && !allGamesSorted[index].values.standing.basic.value" class="ripple-lgt tab p-2 mt-2 mb-2 text-light ls-1 font-weight-bold text-center bg-grn rounded">
                  WIN
                </div>
                <div v-else class="ripple-drk tab p-2 mt-2 mb-2 text-light ls-1 font-weight-bold text-center bg-red rounded">
                  LOSS
                </div>
                <!-- <button data-toggle="tooltip" data-placement="top" title="Tooltip on top">123</button> -->
                <!-- + {{ index }} -->
                <div class="w-100">
                  <div v-if="allGamesSorted[0] && !allGamesSorted[index].values.standing.basic.value" class="d-inline-block w-50 float-left">
                    + {{ calcWin() }}
                  </div>
                  <div v-else class="d-inline-block w-50 float-left">
                    + {{ calcLoss() }}
                  </div>
                  <div class="d-inline-block text-right w-50">
                    <!-- G{{ index }} -->
                  </div>
                </div>
              </div>
            </div>
            <div v-if="allGamesSorted.length === 0" class="col-md-4 pb-4">
              Loading...
            </div>
          </div>
        </div>
      </section>
      <div class="container">
        <div class="row pt-4 pb-2">
            <div class="col-lg-12">
                Discretion: <br>
                Points gained and lost are  projections based solely on data from your W/L record and does not account for those gained by bounties and other means.
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'App',
  data: function () {
    return {
      APIconfig: {
        headers: { 'X-API-Key': 'ae20c30b1c1543ec9a45ba4852b32c92' }
      },
      handle: null,
      memID: null,
      memIDs: null,
      currInfamy: 0,
      pointsToNext: 0,
      pointsToReset: 0,
      show: false,
      chars: [],
      allGames: new Array(12),
      infamySteps: [0, 250, 600, 1000, 1550, 2500, 3100, 3750, 4500, 5350, 6350, 7500, 8800, 10300, 12000, 15000],
      info: null
    }
  },
  computed: {
    allGamesSorted: function () {
      let sorted = this.allGames.sort(function (a, b) {
        if (a.period < b.period) return -1
        else if (a.period > b.period) return 1
        else return 0
      }).reverse().slice(0, 12)

      // console.log(this.allGames)
      console.log(sorted)

      sorted.forEach(function(item) {
        console.log(item.period)
        console.log(item.activityDetails.mode)
        console.log(item.values.standing.basic.displayValue)
      })

      // this.allGames.forEach((i) => {
      //   console.log(i.values.standing.basic.displayValue)
      // })

      return sorted
    },
    currInfamyLevel: function () {
      // this.currInfamy
      let currInfamy = this.currInfamy

      // 1 (0-999)
      // 2 (1000-2499)
      // 3 (2500-4499)
      // 4 (4500-7499)
      // 5 (7500-11999)
      // 6 (1200-15000)

      switch (true) {
        case (currInfamy > 0 && currInfamy <= 999):
          return 0
        case (currInfamy > 1000 && currInfamy <= 2499):
          return 1
        case (currInfamy > 2500 && currInfamy <= 4499):
          return 2
        case (currInfamy > 4500 && currInfamy <= 7499):
          return 3
        case (currInfamy > 7500 && currInfamy <= 11999):
          return 4
        case (currInfamy > 12000 && currInfamy <= 15000):
          return 5
        default:
          return 0
      }
    }
  },
  mounted: function () {
    // this.getIDByUsername()
    let self = this
    setTimeout(() => {
      self.$refs.el.focus()
    }, 1200)
  },
  methods: {
    sayHello () {
      if (this.handle !== '') {
        this.$refs.el.blur()
        this.allGames = []
        this.getIDByUsername()
      }
    },
    calcWin() {
      return '100-250'
    },
    calcLoss() {
      return '0-80'
    },
    calcPoints() {
      return 30
    },
    getIDByUsername: function () {
      let query = this.handle
      let url = 'http://www.bungie.net/platform//Destiny2/SearchDestinyPlayer/-1/' + query

      axios(url, this.APIconfig).then(response => {
        this.memID = response.data.Response[0].membershipId
        this.memIDs = response.data.Response.map(memAccount => [memAccount.membershipId, memAccount.membershipType])

        // console.log('TRACE')
        // console.log(this.memIDs)

        this.getInfamy()
      })
    },
    getInfamy: function () {
      let memID = this.memID
      let url = 'https://www.bungie.net/Platform/Destiny2/2/Profile/' + memID + '/?components=202'

      axios(url, this.APIconfig).then(response => {
        let self = this
        // update infamy points
        // setTimeout(function () {
          self.currInfamy = Object.values(response.data.Response.characterProgressions.data)[0].progressions['2772425241'].currentProgress
          self.pointsToNext = Object.values(response.data.Response.characterProgressions.data)[0].progressions['2772425241'].nextLevelAt -
            Object.values(response.data.Response.characterProgressions.data)[0].progressions['2772425241'].progressToNextLevel
          self.pointsToReset = (15000 - self.currInfamy)

          self.show = true
        // }, 1000)

        // get account char IDs
        this.chars[0] = Object.keys(response.data.Response.characterProgressions.data)[0] || null
        this.chars[1] = Object.keys(response.data.Response.characterProgressions.data)[1] || null
        this.chars[2] = Object.keys(response.data.Response.characterProgressions.data)[2] || null

        this.getLastGambit()
        this.getLastGP()
      })
    },
    getLastGambit: function () {
      // Gambit games
      let urls = [
        'https://www.bungie.net/Platform/Destiny2/2/Account/' + this.memID + '/Character/' + this.chars[0] + '/Stats/Activities/?mode=63',
        'https://www.bungie.net/Platform/Destiny2/2/Account/' + this.memID + '/Character/' + this.chars[1] + '/Stats/Activities/?mode=63',
        'https://www.bungie.net/Platform/Destiny2/2/Account/' + this.memID + '/Character/' + this.chars[2] + '/Stats/Activities/?mode=63'
      ]

      axios(urls[0], this.APIconfig).then(response => {
        if (response.data.Response.activities) {
          this.allGames.push(...response.data.Response.activities.slice(0, 12))
        }
      })

      axios(urls[1], this.APIconfig).then(response => {
        if (response.data.Response.activities) {
          this.allGames.push(...response.data.Response.activities.slice(0, 12))
        }
      })

      axios(urls[2], this.APIconfig).then(response => {
        if (response.data.Response.activities) {
          this.allGames.push(...response.data.Response.activities.slice(0, 12))
        }
      })
    },
    getLastGP: function () {
      // Gambit Prime games
      let urls = [
        'https://www.bungie.net/Platform/Destiny2/2/Account/' + this.memID + '/Character/' + this.chars[0] + '/Stats/Activities/?mode=75',
        'https://www.bungie.net/Platform/Destiny2/2/Account/' + this.memID + '/Character/' + this.chars[1] + '/Stats/Activities/?mode=75',
        'https://www.bungie.net/Platform/Destiny2/2/Account/' + this.memID + '/Character/' + this.chars[2] + '/Stats/Activities/?mode=75'
      ]
      axios(urls[0], this.APIconfig).then(response => {
        if (response.data.Response.activities) {
          this.allGames.push(...response.data.Response.activities.slice(0, 12))
        }
      })

      axios(urls[1], this.APIconfig).then(response => {
        if (response.data.Response.activities) {
          this.allGames.push(...response.data.Response.activities.slice(0, 12))
        }
      })

      axios(urls[2], this.APIconfig).then(response => {
        if (response.data.Response.activities) {
          this.allGames.push(...response.data.Response.activities.slice(0, 12))
        }
      })
    }
  }
}
</script>

<style>
#app, body, html {
  height: 100%;
}

body {
  background: url('/static/infamous_bg.png') no-repeat center center fixed!important;
  background-color: #183743!important;
  background-size: cover!important;
  color: #FFF!important;
}

a {
  color: #FFF!important;
}

.font-md {
  font-size: 18px;
}

a:hover {
  color: #DDD!important;
  text-decoration: none!important;
}

input[type=text] {
  background: none;
  color: #FFF;
  border: none;
  outline: none;
}

::placeholder {
  color: #FFF;
}

.tab {
  box-shadow: 1px 1px 1px 1px rgba(16, 16, 16, .5);
  -webkit-transition: all .3s ease-out;
  -moz-transition: all .3s ease-out;
  -o-transition: all .3s ease-out;
  -ms-transition: all .3s ease-out;
  transition: all .3s ease-out;
}

.tab:hover {
  box-shadow: 3px 3px 5px 3px rgba(16, 16, 16, .5);
  /* transition: .2s ease-in-out; */
  cursor: pointer;
}

.bg-red {
  background-color: #B1352A !important;
}

.bg-grn {
  background-color: #01B08F !important;
}

.ls-1 {
  letter-spacing: .06rem;
}

.p-2 {
  padding: .6rem!important;
}

/* Ripple effect */
.ripple-lgt, .ripple-drk {
  background-position: center;
  /* transition: background 0.8s; */
}

.ripple-lgt:hover {
  background: #5EAD9E radial-gradient(circle, transparent 1%, #5EAD9E 1%) center/15000%;
}

.ripple-lgt:active {
  background-size: 100%;
  transition: background 0s;
}

.ripple-drk:hover {
  background: #A9352B radial-gradient(circle, transparent 1%, #A9352B 1%) center/15000%;
}

.ripple-drk:active {
  background-size: 100%;
  transition: background 0s;
}

/* Vue Transitions */

.fade-enter-active, .fade-leave-active {
  transition: opacity 1.8s;
}

.fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
  opacity: 0;
}

/* Slide-fade */

.slide-fade-enter-active {
  transition: all .6s ease;
}
.slide-fade-leave-active {
  transition: all .6s cubic-bezier(1.0, 0.5, 0.8, 1.0);
}
.slide-fade-enter, .slide-fade-leave-to
/* .slide-fade-leave-active for <2.1.8 */ {
  /* transform: translateX(10px); */
  opacity: 0;
}
</style>
