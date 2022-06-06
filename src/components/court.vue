<template>
  <div
    id="court_container"
    tabindex="-1"
    @keydown.right="controlPad('moveTank', 'right', 1)"
    @keydown.left="controlPad('moveTank', 'left', 1)"
    @keydown.up="controlPad('moveTank', 'up', 1)"
    @keydown.down="controlPad('moveTank', 'down', 1)"
    @keyup.right="controlPad('stopTank', 'right', 1)"
    @keyup.left="controlPad('stopTank', 'left', 1)"
    @keyup.up="controlPad('stopTank', 'up', 1)"
    @keyup.down="controlPad('stopTank', 'down', tankNum)"
    @keydown.68="controlPad('moveTank', 'right', tankNum)"
    @keydown.65="controlPad('moveTank', 'left', tankNum)"
    @keydown.87="controlPad('moveTank', 'up', tankNum)"
    @keydown.83="controlPad('moveTank', 'down', tankNum)"
    @keyup.68="controlPad('stopTank', 'right', tankNum)"
    @keyup.65="controlPad('stopTank', 'left', tankNum)"
    @keyup.87="controlPad('stopTank', 'up', tankNum)"
    @keyup.83="controlPad('stopTank', 'down', tankNum)"
    @keydown.102="controlPad('rotateCannon', 'clockWise', 1)"
    @keyup.102="controlPad('stopCannon', 'clockWise', 1)"
    @keydown.100="controlPad('rotateCannon', 'counterClockWise', 1)"
    @keyup.100="controlPad('stopCannon', 'counterClockWise', 1)"
    @keydown.74="controlPad('rotateCannon', 'clockWise', tankNum)"
    @keyup.74="controlPad('stopCannon', 'clockWise', tankNum)"
    @keydown.71="controlPad('rotateCannon', 'counterClockWise', tankNum)"
    @keyup.71="controlPad('stopCannon', 'counterClockWise', tankNum)"
    @keydown.enter="fire(1)"
    @keydown.107="switchWeapon(tankNum)"
    @keydown.16="switchWeapon(tankNum)"
    @keydown.space="fire(tankNum)"
  >
    <p>{{ tankNum }}</p>
    <informsBand v-if="mapNumber >= 1">
      <inform1></inform1>
      <clock></clock>
      <inform2></inform2>
    </informsBand>

    <startPage v-if="mapNumber == -1"></startPage>
    <endPage v-if="mapNumber == -2"></endPage>
    <chooseMapPage v-if="mapNumber == 0"></chooseMapPage>
    <map1 v-if="mapNumber >= 1">
      <bullet1></bullet1>
      <bullet2></bullet2>
      <tank1></tank1>
      <tank2></tank2>
    </map1>
    <p id="copy">copyright ©2020 tank-vue.js jimmystereo</p>
  </div>
</template>

<script>
import $ from 'jquery'
import { io } from 'socket.io-client'

import { bus } from '../main'
import tank1 from './tank1'
import tank2 from './tank2'
import bullet1 from './bullet1'
import bullet2 from './bullet2'
import map1 from './maps/map'
import informsBand from './informsBand'
import inform2 from './inform2'
import inform1 from './inform1'
import clock from './clock'
import startPage from './maps/startPage'
import endPage from './maps/endPage'
import chooseMapPage from './maps/chooseMapPage'
import resetState from '../mixins/reset'
import checkWinner from '../mixins/checkWinner'
const main = {}
const event = {
  server: {
    connected: 'connected',
    player2Connected: 'player2Connected',
    remotePlayerChanged: 'remotePlayerChanged',
  },
  client: {
    playerChanged: 'playerChanged',
  },
}

const params = new Proxy(new URLSearchParams(window.location.search), {
  get: (searchParams, prop) => searchParams.get(prop),
})

const roomId = window.location.pathname.substring(1)
const tankNum = parseInt(params['tank-num'])
const remoteTankNum = tankNum === 1 ? 2 : 1

let syncRemotePlayerInterval
export default {
  components: {
    tank1,
    tank2,
    bullet1,
    bullet2,
    informsBand,
    inform1,
    inform2,
    clock,
    map1,
    startPage,
    chooseMapPage,
    endPage,
  },
  data() {
    return {}
  },
  methods: {
    controlPad: function (function_name, direction, tank_number) {
      if (this.$store.state.mapNumber >= 1) {
        this.$store.commit(function_name, [direction, tank_number])
        this.text_direction = direction
      }
    },
    syncRemotePlayer: function () {
      const remoteTankNum = tankNum === 1 ? 2 : 1
      main.socket.emit(event.client.playerChanged, {
        winner: this.$store.state.winner,
        champ: this.$store.state.champ,
        count_down: this.$store.state.count_down,
        tankNum: tankNum,
        ['tank' + remoteTankNum]: this.$store.state['tank' + remoteTankNum],
        ['tank' + tankNum]: this.$store.state['tank' + tankNum],
        ['bullet' + tankNum]: this.$store.state['bullet' + tankNum],
      })
    },
    switchWeapon: function (tankNum) {
      if (
        ++this.$store.state['bullet' + tankNum].tmpType >
        this.$store.state.maxType
      ) {
        this.$store.state['bullet' + tankNum].tmpType = 1
      }
    },
    switchWeapon1: function () {
      if (++this.$store.state.bullet1.tmpType > this.$store.state.maxType) {
        this.$store.state.bullet1.tmpType = 1
      }
    },
    switchWeapon2: function () {
      if (++this.$store.state.bullet2.tmpType > this.$store.state.maxType) {
        this.$store.state.bullet2.tmpType = 1
      }
    },
    fire: function (tankNum) {
      bus.$emit('fire' + tankNum, 'nothing')
    },
    fire1: function () {
      bus.$emit('fire1', 'nothing')
    },
    fire2: function () {
      bus.$emit('fire2', 'nothing')
    },
    switchMap: function () {
      this.$store.state.mapNumber--
      console.log(this.$store.state.mapNumber)
    },
    // setTank: function (tank, tank_number) {
    //   this.$store.commit('setTank', [tank, tank_number])
    // },
  },
  computed: {
    currentMap: function () {
      return this.$store.getters.currentMap
    },
    bullet1: function () {
      return this.$store.state.bullet1
    },
    bullet2: function () {
      return this.$store.state.bullet2
    },
    tank1: function () {
      return this.$store.state.tank1
    },
    tank2: function () {
      return this.$store.state.tank2
    },
    mapNumber: function () {
      return this.$store.state.mapNumber
    },
    tankNum: function () {
      return tankNum
    },
  },
  mixins: [resetState],
  created: function () {
    const compo = this

    main.socket = io(':8081', {
      query: {
        roomId: roomId,
        newGameId: 'tank',
      },
    })

    main.socket.on('connected', function (players) {
      main.players = players
      const myId = main.socket.id

      Object.keys(players).forEach(function (id) {
        if (players[id].playerId === myId) {
          console.log('myId: ' + myId)
          // addPlayer(main, players[id], myNameEle)
        } else {
          // addOtherPlayers(main, players[id])
        }
      })

      // onConnected(main, myId)

      console.log('connected')
      // Game
      // addGameEvent(main)
      syncRemotePlayerInterval = setInterval(() => {
        compo.syncRemotePlayer()
      }, 1000 / 30)
    })

    main.socket.on(event.server.remotePlayerChanged, function (remoteState) {
      compo.$store.commit('setRemoteState', [remoteState, remoteTankNum])
      // checkWinner()
    })
  },
  mounted() {
    $('#copy').hide()
    $('#copy').fadeIn(1000)
  },
}
</script>
<style>
#court_container {
  border-color: transparent;
  background-color: black;
  margin-top: 0%;
  width: 100%;
  height: 1200px;
}
p {
  display: inline;
}
h2 {
  display: inline;
}
.tanks {
  z-index: 47;
  border-width: 5px;
  border-style: outset;
  text-align: center;
  border-radius: 50%;
  position: absolute;
  margin: 0 0;
  margin-right: 1px;
}
#copy {
  position: absolute;
  color: white;
  top: 1110px;
  left: 45%;
}
</style>
