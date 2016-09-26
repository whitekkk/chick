<template>
  <div @mousemove="mousePosition()">
    <div id="app"  @mouseover="move" @mouseout="outOfArea">
      <div class="avatar" :style="{'top': avatar.y-25 + 'px', 'left': avatar.x-25 + 'px'}" v-for="avatar in avatars">
        <a style="margin: 10px;">
          {{avatar.name}}
        </a>
        <div>
          <img src="../img/king.png" width="65" height="65" style="position: absolute; z-index: 999; top:5px; left:-1px;"/>
          <img src="../img/1.png" width="60" height="60" style="position: absolute; z-index: 99;"/>
          <img src="../img/LEG1.png" width="10" height="10" style="position: absolute; z-index: 99; top:75px; left:20px;"/>
          <img src="../img/LEG1.png" width="10" height="10" style="position: absolute; z-index: 99; top:75px; left:30px;"/>
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
      if (vm.myAvatar.id === id) {
        vm.myAvatar.x = snapshot.val().x
        vm.myAvatar.y = snapshot.val().y
      }
    })
    Avatars.on('child_removed', function (snapshot) {
      var id = snapshot.key
      vm.avatars.$remove(vm.avatars.find(avatar => avatar.id === id))
    })
  },
  data () {
    let chickColor = Math.floor(Math.random() * 3)
    // let r = Math.floor(Math.random() * 255)
    // let g = Math.floor(Math.random() * 255)
    // let b = Math.floor(Math.random() * 255)

    let x = Math.floor(Math.random() * 500)
    let y = Math.floor(Math.random() * 500)

    return {
      mouseX: 0,
      mouseY: 0,
      avatars: [],
      myAvatar: {
        name: 'Ponzi',
        color: `${chickColor}`, // color: `rgb(${r}, ${g}, ${b})`,
        x,
        y,
        active: false
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
        if (vm.myAvatar.y < 500 && vm.myAvatar.active) {
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
    }
  }
}
</script>

<style>
#app {
  width: 500px;
  height: 500px;
  background: #ccc;
}
#outOfArea {
  width: 1000px;
  height: 1000px;
  background: #0cc;
}
.avatar {
  position: absolute;
  width: 50px;
  height: 50px;
}
</style>
