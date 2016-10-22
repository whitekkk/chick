 <template>
  <div @mousemove="mousePosition()" tabindex="0"
        @keydown.z="upSpeed"
        @keyup.z="normalSpeed"
        @keydown.x="eat"
        @keyup.x="shutup"
  >
    <div class="screen" @mouseover="move(time)" @mouseout="outOfArea">
      <img v-if="myAvatar.id === avatar.id" src="../static/img/gird-bg.png" :style="{'position':'absolute', 'top': -myAvatar.y+25+(winHeight/2)-50 + 'px', 'left': -myAvatar.x+25+(winWidth/2)-50 + 'px'}" v-for="avatar in avatars"/>

      <div class="avatar" :style="{'top': avatar.y-myAvatar.y+(winHeight/2)-50 + 'px', 'left': avatar.x-myAvatar.x+(winWidth/2)-50 + 'px'}" v-for="avatar in avatars">
        <div>
          <!-- <img src="../static/img/king.png" width="65" height="65" style="position: absolute; z-index: 999; top:5px; left:-1px;"/>
          -->
          <div v-if="myAvatar.id !== avatar.id" >
            <a style="margin: 10px;">
              {{avatar.name}}
            </a>
            <img :src="'../static/img/' + avatar.body + '.png'" width="60" height="60" style="position: absolute; z-index: 99;"/>
            <img :src="'../static/img/LEG' + avatar.leg1 + '.png'" width="10" height="10" style="position: absolute; z-index: 99; top:75px; left:20px;"/>
            <img :src="'../static/img/LEG' + avatar.leg2 + '.png'" height="10" style="position: absolute; z-index: 99; top:75px; left:30px;"/>
          </div>
        </div>
      </div>

      <div class="avatar" :style="{'position':'absolute', 'top': winHeight/2-50 + 'px', 'left': winWidth/2-50 + 'px'}">
        <a style="margin: 10px;">
          {{myAvatar.name}}
        </a>
        <img :src="'../static/img/' + myAvatar.body + '.png'" width="60" height="60" style="position: absolute; z-index: 99;"/>
        <img :src="'../static/img/LEG' + myAvatar.leg1 + '.png'" width="10" height="10" style="position: absolute; z-index: 99; top:75px; left:20px;"/>
        <img :src="'../static/img/LEG' + myAvatar.leg2 + '.png'" height="10" style="position: absolute; z-index: 99; top:75px; left:30px;"/>
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
      avatar.speed = snapshot.val().speed
      avatar.eat = snapshot.val().eat
      // change
      if (vm.myAvatar.id === id) {
        vm.myAvatar.x = snapshot.val().x
        vm.myAvatar.y = snapshot.val().y
        vm.myAvatar.body = snapshot.val().body
        vm.myAvatar.leg1 = snapshot.val().leg1
        vm.myAvatar.leg2 = snapshot.val().leg2
        vm.myAvatar.speed = snapshot.val().speed
        vm.myAvatar.eat = snapshot.val().eat
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
    // camera = 1024*768 or 800*600
    // bg = 3000 × 2988
    // coner = 25 × 0
    // 25
    // coner = 2975 × 2913
    // 25 75
    // 2950
    // 2838
    let x = Math.floor(Math.random() * 2950) + 25
    let y = Math.floor(Math.random() * 2838)
    let body = Math.floor(Math.random() * 3) + 1
    let winHeight = window.screen.availHeight
    let winWidth = window.screen.availWidth

    return {
      winHeight,
      winWidth,
      mouseX: 0,
      mouseY: 0,
      avatars: [],
      time: 10,
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
        speed: false,
        eat: false,
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
    move (time) {
      // console.log(this.winHeight)
      let vm = this
      let xCenter = vm.winWidth / 2
      let yCenter = vm.winHeight / 2
      var xOrigin = vm.myAvatar.x
      var yOrigin = vm.myAvatar.y
      var x1 = vm.mouseX
      var y1 = vm.mouseY
      var dx = 0
      var dy = 0
      var checkX = 1
      var checkY = 1
      var err = 0
      // var checkArea = (((xOrigin < 0 || yOrigin < 0) && (x1 < xCenter || y1 < yCenter)) || ((xOrigin > 3000 || yOrigin > 2988) && (x1 > xCenter || y1 > yCenter)))
      // var checkCenter = ((xCenter + 25 > x1) && (xCenter - 25 < x1)) && ((yCenter - 25 < y1) && (yCenter + 25 > y1))
      vm.myAvatar.active = setInterval(function () {
        x1 = vm.mouseX
        y1 = vm.mouseY
        dx = Math.abs(x1 - xCenter)
        dy = Math.abs(y1 - yCenter)
        checkX = (xCenter < x1) ? 1 : -1
        checkY = (yCenter < y1) ? 1 : -1
        err = dx - dy
        if (vm.myAvatar.active) {
          firebase.database().ref('avatars/' + vm.myAvatar.id).update({
            y: yOrigin,
            x: xOrigin
          })
        }
        var e2 = 2 * err

        if (!(((xCenter + 25 > x1) && (xCenter - 25 < x1)) && ((yCenter - 25 < y1) && (yCenter + 25 > y1)))) {
          // clearInterval(vm.myAvatar.active)
          if (e2 > -dy) {
            err -= dy
            xOrigin += checkX
          }
          if (e2 < dx) {
            err += dx
            yOrigin += checkY
          }
        }
        // check out of area
        if (yOrigin < 0 && y1 < yCenter) {
          yOrigin = 0
        }
        if (xOrigin < 25 && x1 < xCenter) {
          xOrigin = 25
        }
        if (yOrigin > 2913 && y1 > yCenter) {
          yOrigin = 2913
        }
        if (xOrigin > 2975 && x1 > xCenter) {
          xOrigin = 2975
        }
        vm.actionLeg()
      }, time)
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
      let slope = (vm.winHeight / 2 - vm.mouseY) / (vm.winWidth / 2 - vm.mouseX)
      // console.log(slope)
      if (vm.mouseX > vm.winWidth / 2 && vm.mouseY > vm.winHeight / 2) { // q 4
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
      } else if (vm.mouseX > vm.winWidth / 2 && vm.mouseY < vm.winHeight / 2) { // q1
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
      } else if (vm.mouseX < vm.winWidth / 2 && vm.mouseY < vm.winHeight / 2) { // q2
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
      } else if (vm.mouseX < vm.winWidth / 2 && vm.mouseY > vm.winHeight / 2) { // q3
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
      if (vm.myAvatar.eat) {
        firebase.database().ref('avatars/' + vm.myAvatar.id).update({
          body: this.myAvatar.body + '-0'
        })
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
    },
    upSpeed () {
      if (!this.myAvatar.speed) {
        this.time = -100
        clearInterval(this.myAvatar.active)
        this.move(this.time)
        firebase.database().ref('avatars/' + this.myAvatar.id).update({
          speed: true
        })
      }
    },
    normalSpeed () {
      this.time = 10
      clearInterval(this.myAvatar.active)
      this.move(this.time)
      firebase.database().ref('avatars/' + this.myAvatar.id).update({
        speed: false
      })
    },
    eat () {
      if (this.myAvatar.body.search('-0') === -1) {
        firebase.database().ref('avatars/' + this.myAvatar.id).update({
          body: this.myAvatar.body + '-0',
          eat: true
        })
      }
    },
    shutup () {
      firebase.database().ref('avatars/' + this.myAvatar.id).update({
        body: this.myAvatar.body.replace('-0', ''),
        eat: false
      })
    }
  }
}
</script>

<style>
html {
  overflow:hidden;
}
div.screen {
  position: relative;
  width: 100vw;
  height: 100vh;
  top: 0;
  left: 0;
  overflow:hidden;
  border-color: #FFFFFF
  ;
}
#outOfArea {
  width: 100vw;
  height: 100vh;
}
div.avatar {
  position: absolute;
  width: 50px;
  height: 50px;
}
</style>
