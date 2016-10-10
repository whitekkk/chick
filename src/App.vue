<template>
  <div @mousemove="mousePosition()">
    <div id="screen"  @mouseover="move" @mouseout="outOfArea">
      <div class="avatar" :style="{'top': avatar.y-25 + 'px', 'left': avatar.x-25 + 'px'}" v-for="avatar in avatars">
        <a style="margin: 10px;">
          {{avatar.name}}
        </a>
        <div>
          <!-- <img src="../static/img/king.png" width="65" height="65" style="position: absolute; z-index: 999; top:5px; left:-1px;"/>
          -->
          <img :src="'../static/img/' + avatar.body + '.png'" width="60" height="60" style="position: absolute; z-index: 99;"/>
          <img :src="'../static/img/LEG' + avatar.leg1 + '.png'" width="10" height="10" style="position: absolute; z-index: 99; top:75px; left:20px;"/>
          <img :src="'../static/img/LEG' + avatar.leg2 + '.png'" height="10" style="position: absolute; z-index: 99; top:75px; left:30px;"/>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import firebase from 'firebase'

var config = {
  apiKey: 'AIzaSyDI7TmN6g-aS8rIvzdWPu0KTtFe0_HW0A4',
  authDomain: 'todos-4ded8.firebaseapp.com',
  databaseURL: 'https://todos-4ded8.firebaseio.com',
  storageBucket: 'todos-4ded8.appspot.com',
  messagingSenderId: '245537528887'
}
firebase.initializeApp(config)

var Avatars = firebase.database().ref('avatars')
var myId = ''
window.onbeforeunload = function () {
  firebase.database().ref('avatars/' + myId).remove()
}

export default {
  ready () {
    let vm = this
    vm.addAvatar(vm.myAvatar)
    Avatars.on('child_added', function (snapshot) {
      var item = snapshot.val()
      item.id = snapshot.key
      vm.avatars.push(item)
    })
    Avatars.on('child_changed', function (snapshot) {
      var id = snapshot.key
      var avatar = vm.avatars.find(avatar => avatar.id === id)
      avatar.x = snapshot.val().x
      avatar.y = snapshot.val().y
      avatar.body = snapshot.val().body
      avatar.leg1 = snapshot.val().leg1
      avatar.leg2 = snapshot.val().leg2
      // change
      if (vm.myAvatar.id === id) {
        vm.myAvatar.x = snapshot.val().x
        vm.myAvatar.y = snapshot.val().y
        vm.myAvatar.body = snapshot.val().body
        vm.myAvatar.leg1 = snapshot.val().leg1
        vm.myAvatar.leg2 = snapshot.val().leg2
      }
    })
    Avatars.on('child_removed', function (snapshot) {
      var id = snapshot.key
      vm.avatars.$remove(vm.avatars.find(avatar => avatar.id === id))
    })
  },
  data () {
    // let r = Math.floor(Math.random() * 255)
    // let g = Math.floor(Math.random() * 255)
    // let b = Math.floor(Math.random() * 255)

    let x = Math.floor(Math.random() * 500)
    let y = Math.floor(Math.random() * 500)
    let body = Math.floor(Math.random() * 3) + 1

    return {
      mouseX: 0,
      mouseY: 0,
      avatars: [],
      myAvatar: {
        name: 'Ponzi',
        // color: `${chickColor}`, // color: `rgb(${r}, ${g}, ${b})`,
        x,
        y,
        active: false,
        body: body + '-7',
        color: body,
        leg1: '1',
        leg2: '1',
        king: false
      }
    }
  },

  // computed property for form validation state
  computed: {
  },

  // methods
  methods: {
    addAvatar: function (newAvatar) {
      let result = Avatars.push(newAvatar)
      this.myAvatar.id = result.key
      myId = this.myAvatar.id
    },
    move () {
      let vm = this
      var xOrigin = vm.myAvatar.x
      var yOrigin = vm.myAvatar.y
      vm.myAvatar.active = setInterval(function () {
        var x1 = vm.mouseX
        var y1 = vm.mouseY
        var dx = Math.abs(x1 - xOrigin)
        var dy = Math.abs(y1 - yOrigin)
        var checkX = (xOrigin < x1) ? 1 : -1
        var checkY = (yOrigin < y1) ? 1 : -1
        var err = dx - dy
        if (vm.myAvatar.active) {
          firebase.database().ref('avatars/' + vm.myAvatar.id).update({
            y: yOrigin,
            x: xOrigin
          })
        }
        if ((xOrigin === x1) && (yOrigin === y1)) clearInterval(vm.myAvatar.active)
        var e2 = 2 * err
        if (e2 > -dy) {
          err -= dy
          xOrigin += checkX
        }
        if (e2 < dx) {
          err += dx
          yOrigin += checkY
        }
        vm.actionLeg()
      }, 10)
    },
    outOfArea () {
      console.log('out of area')
      let vm = this
      clearInterval(vm.myAvatar.active)
      firebase.database().ref('avatars/' + vm.myAvatar.id).update({
        active: vm.myAvatar.active = false
      })
    },
    mousePosition () {
      let vm = this
      let position = window.event
      vm.mouseX = position.clientX
      vm.mouseY = position.clientY
      vm.actionChick()
    },
    actionChick () {
      let vm = this
      let body = vm.myAvatar.color.toString()
      let slope = (vm.myAvatar.y - vm.mouseY) / (vm.myAvatar.x - vm.mouseX)
      console.log(slope)
      if (vm.mouseX > vm.myAvatar.x && vm.mouseY > vm.myAvatar.y) { // q 4
        if (slope < 0.25) {
          firebase.database().ref('avatars/' + vm.myAvatar.id).update({
            body: body + '-1'
          })
        } else if (slope < 0.75) {
          firebase.database().ref('avatars/' + vm.myAvatar.id).update({
            body: body + '-8'
          })
        } else {
          firebase.database().ref('avatars/' + vm.myAvatar.id).update({
            body: body + '-7'
          })
        }
      } else if (vm.mouseX > vm.myAvatar.x && vm.mouseY < vm.myAvatar.y) { // q1
        if (slope < -0.75) {
          firebase.database().ref('avatars/' + vm.myAvatar.id).update({
            body: body + '-3'
          })
        } else if (slope < -0.25) {
          firebase.database().ref('avatars/' + vm.myAvatar.id).update({
            body: body + '-2'
          })
        } else {
          firebase.database().ref('avatars/' + vm.myAvatar.id).update({
            body: body + '-1'
          })
        }
      } else if (vm.mouseX < vm.myAvatar.x && vm.mouseY < vm.myAvatar.y) { // q2
        if (slope < 0.25) {
          firebase.database().ref('avatars/' + vm.myAvatar.id).update({
            body: body + '-5'
          })
        } else if (slope < 0.75) {
          firebase.database().ref('avatars/' + vm.myAvatar.id).update({
            body: body + '-4'
          })
        } else {
          firebase.database().ref('avatars/' + vm.myAvatar.id).update({
            body: body + '-3'
          })
        }
      } else if (vm.mouseX < vm.myAvatar.x && vm.mouseY > vm.myAvatar.y) { // q3
        if (slope > -0.25) {
          firebase.database().ref('avatars/' + vm.myAvatar.id).update({
            body: body + '-5'
          })
        } else if (slope > -0.75) {
          firebase.database().ref('avatars/' + vm.myAvatar.id).update({
            body: body + '-6'
          })
        } else {
          firebase.database().ref('avatars/' + vm.myAvatar.id).update({
            body: body + '-7'
          })
        }
      }
    },
    actionLeg () {
      let vm = this
      if (vm.myAvatar.x % 20 === 0 || vm.myAvatar.y % 20 === 0) {
        firebase.database().ref('avatars/' + vm.myAvatar.id).update({
          leg1: '1',
          leg2: '3'
        })
      } else if (vm.myAvatar.x % 10 === 0 || vm.myAvatar.y % 10 === 0) {
        firebase.database().ref('avatars/' + vm.myAvatar.id).update({
          leg1: '2',
          leg2: '1'
        })
      }
    }
  }
}
</script>

<style>
html {
  background-image: url("../static/img/gird-bg.png");
}
#screen {
  width: 100vw;
  height: 100vh;
  top: 0;
  left: 0;
}
#outOfArea {
  width: 100vw;
  height: 100vh;
}
.avatar {
  position: absolute;
  width: 50px;
  height: 50px;
}
</style>
