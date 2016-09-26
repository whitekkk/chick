<template>
  <div @mousemove="mousePosition()">
    <div id="app" @mouseover="move" @mouseout="outOfArea">
    </div>
  </div>

    <div class="avatar" :style="{'background': avatar.color, 'top': avatar.y + 'px', 'left': avatar.x + 'px'}" v-for="avatar in avatars">
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
    let r = Math.floor(Math.random() * 255)
    let g = Math.floor(Math.random() * 255)
    let b = Math.floor(Math.random() * 255)

    let x = Math.floor(Math.random() * 500)
    let y = 0

    return {
      avatars: [],
      myAvatar: {
        color: `rgb(${r}, ${g}, ${b})`,
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
      console.log('move')
      let vm = this
      vm.myAvatar.active = setInterval(function () {
        if (vm.myAvatar.y < 500 && vm.myAvatar.active) {
          firebase.database().ref('avatars/' + vm.myAvatar.id).update({
            y: vm.myAvatar.y + 1
          })
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
      let position = window.event
      console.log('Position X : ' + position.clientX + ' Position Y : ' + position.clientY)
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
  width: 10px;
  height: 10px;
  background: #F00;
}
</style>
