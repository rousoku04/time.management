<template>
  <div class="container">
    <div id="taskRegister" v-show="showRegister" v-on:click="closeRegister">
      <div id="trContent" ref="trContent"  v-on:click="stopEvent">
        <p>タスクを登録</p>
        <label>カテゴリー名</label>

        <input type="text" v-model="category"></input><br>

        <!-- <select v-model="selected">
          <option v-for="task in tasks" v-bind:value="task.name" v-model="category">{{ task.name }}</option>
        </select> -->

        <label>開始時間：</label>
        <input type="datetime-local" v-model="start_time"></input><br>
        <label>終了時間：</label>
        <input type="datetime-local" v-model="finish_time"></input><br>
        <button @click="onSubmit()">送信する</button>
        <!--  <text class="button" type="text" @click="onSubmit()">送信する</text> -->
      </div>
    </div>

    <div class="gantt" id="gantt" @mousedown="locationDown" @mousemove="locationMove" @mouseup="locationUp">
      <div class="task" ref="newTask"></div>

      <div class="task" v-for="task in tasks" :key="task.id" :ref="task.id" @click="openModel(task)"  @mousedown.stop @mousemove.stop @mouseup.stop>
        <div class="taskDetail">
        {{ task.name }}<br>
        {{ task.start_time | moment }} - {{ task.finish_time | moment }}
        </div>
      </div>
      <div ref="borderLine" id="borderLine"></div>
      <div v-for="n in 23" :class="'timesBorder'+ n"></div>
      <div v-for="n in 25" :class="'times'+ n">{{ n-1 }}:00</div>
    </div>

    <div id="overlay" v-show="showContent" v-on:click="closeModel">
      <div id="content" v-on:click="stopEvent">
        <button v-on:click="closeModel" class="close">×</button>
        <p class="taskName">{{ chosenTask.name}}</p>
        <p class="taskst">{{ chosenTask.start_time | momentDays}} / {{ chosenTask.start_time | moment}}-{{ chosenTask.finish_time | moment}}</p>
        <p class="taskSentence">このタスクを変更/削除しますか？</p>
        <div class="buttonSet">
          <button v-on:click="edit" class="edit">編集</button>
          <button v-on:click="delete_1" class="delete">削除</button>
        </div>
      </div>
    </div>
  </div>
</template>
<script>

// vueからcontrollerを呼び出している！！！

export default {
  // mounted：instanceを読み込まれちょっと後に呼び出される。
  mounted() {
    //   ここで、まず必要なデータを引っ張ってくる.
    this.fetchTasks()
    // this.setGantt()
    setInterval(this.borderPlace(), 1000);
  },
  //サーバから引っ張ってきたデータや、自分で打ち込んだデータを入れていく
  data() {
    
    return {
      showContent:false,
      showRegister:false,
      taskId: 0,
      yDownPage: 0,
      yDown: 0,
      yMove: 0,
      yUp: 0,
      beforeDis: 0,
      taskBoolean: false,
      startPointX: 0,
      startPointY: 0,
      category:'',
      finish_time:'',
      start_time:'',
      chosenTask: [
        {

        },
      ],
      tasks: [
        {
        
        },
      ]
    }
  },

  //methods：処理を埋め込んで、後で呼び出す。
  methods: {
    // awaitを使うときはasyncを書く
    async fetchTasks() {
      // web.phpにfetch_task_dataのURLでリクエストしている。
        const url = "fetch_task_data_2home1day"
        // awaitによって非同期処理を待っている。
        await axios.get(url).then(res => {
            this.tasks = res.data
            // console.log(this.tasks)
        })
        this.setGantt()
    },
    setGantt() {
      // const length = this.tasks.length
      const length = Object.keys(this.tasks).length
      for(let i = 0; i < length; i++) {
        this.setCoordinate(this.tasks[i])
        // console.log(this.tasks[i])
      }
    },
    setCoordinate(task) {
      const id = task.id
      this.$refs[id][0].style.top = String(this.time2coordinate(task.start_time)) + 'px'
      this.$refs[id][0].style.height = String(this.time2coordinate(task.finish_time) - this.time2coordinate(task.start_time)) + 'px'
      this.$refs[id][0].style.color = 'white'
    },
    time2coordinate(str) {
      // split：文字列を分割して配列にする。
      const his = str.split(' ')[1]
      const hisArr = his.split(':')
      const h = Number(hisArr[0])
      const i = Number(hisArr[1])
      // const s = Number(hisArr[2])
      const totalMinutes = h * 60 + i
      const dayMinutes = 24 * 60
      const leftCoordinate = 1440 * totalMinutes / dayMinutes
      // console.log("a"+leftCoordinate)
      return leftCoordinate
    },
    // alert(task){
    //   alert(task.id)
    // },
    openModel: function(task){
      this.showContent = true
      this.chosenTask = task
      this.taskId = task.id
      console.log(this.chosenTask)
    },
    closeModel: function(){
      this.showContent = false
    },
    stopEvent: function(event){
      event.stopPropagation()
    },
    edit: function(){
      const task_id = this.taskId
      const editUrl = "/task_edit/" + String(task_id)
      window.location.href = editUrl
    },
    delete_1: function(){
      const task_id_2 = this.taskId
      axios.post('/task_delete_2',{
        delete_task_id: task_id_2
      })
      window.location.href = "/home"
    },
    borderPlace: function(){
      let current_date = new Date()
      let nowTime = 60 * current_date.getHours() + current_date.getMinutes()
      this.$refs.borderLine.style.top = String(nowTime) + 'px'
      // this.$refs.borderLine.style.color = 'red'
      // console.log("place"+nowTime)
    },
    todayDate: function(){
      let date = new Date()
      let year = date.getFullYear()
      let month = ("0" + (date.getMonth() + 1)).slice(-2)
      let date1 = ( "0" + date.getDate()).slice(-2)

      return year + "-" + month + "-" + date1 + "T"
    },

    locationDown: function(locDown){
      this.$refs.newTask.style.top = '0px'
      this.$refs.newTask.style.height = '0px'
      // this.startPointX = 15 * Math.floor(locDown.clientX/15)
      this.startPointY = 15 * Math.floor(locDown.clientY/15)

      let yCoordinate1 = 15 * Math.floor(locDown.offsetY/15)
      this.yDown = yCoordinate1
      this.yDownPage = locDown.pageY
      this.$refs.newTask.style.top = String(yCoordinate1) + 'px'

      let downHour = ( "0" + Math.floor(yCoordinate1/60)).slice(-2)
      let downMinute = ("0" + yCoordinate1 % 60).slice(-2)
      let downToday = this.todayDate()

      this.start_time = downToday + downHour + ":" + downMinute
      this.taskBoolean = true
    },
    
    locationMove: function(e){
      if(this.taskBoolean){
        this.yMove = 15 * Math.floor(e.offsetY/15)
        if(e.pageY - this.yDownPage <= 0){
          this.$refs.newTask.style.height = '0px'
        }
        else{
          if(this.yMove - this.beforeDis >= 0){
            this.beforeDis = this.yMove
            this.$refs.newTask.style.height = String(this.yMove - this.yDown) + 'px'
          }else{
            this.beforeDis = this.yMove
            this.$refs.newTask.style.height = String(this.yMove) + 'px'
          }
        }
      }
    },
    locationUp: function(locUp){
      let yCoordinate3 = 0
      yCoordinate3 = 15 * Math.floor(locUp.offsetY/15)
      this.yUp = yCoordinate3
      this.$refs.newTask.style.height = String(yCoordinate3 - this.yDown) + 'px'

      let upHour = ( "0" + Math.floor(yCoordinate3/60)).slice(-2)
      let upMinute = ("0" + yCoordinate3 % 60).slice(-2)
      let upToday = this.todayDate()

      this.finish_time = upToday + upHour + ":" + upMinute
      this.taskBoolean = false

      this.displayModal()
    },
    displayModal: function(){
      this.showRegister = true
      this.$refs.trContent.style.top = String(this.startPointY) + 'px'
      // this.$refs.trContent.style.left = String(this.startPointX) + 'px'
    },
    closeRegister: function(){
      this.showRegister = false
      this.$refs.newTask.style.top = '0px'
      this.$refs.newTask.style.height = '0px'
    },
    onSubmit: function() {

        const params = {
            category_name2: this.category,
            start_time: this.start_time,
            finish_time: this.finish_time,
        };
        console.log(category_name2)
        axios.post('/task_send', params)
              window.location.href = "/home"
            // .then(function(response){
            // })
            // .catch(function(error){
            //     // 失敗したとき
            // });
    }
  },

  filters:{
    moment: function(date){
      return moment(date).format('HH:mm');
    },
    momentDays: function(date){
      return moment(date).format('MM月DD日');
    }
  }
  
}


</script>
<style scoped lang="scss">
.cantainer {
  height: 100%;
}
$ganttWidth: 700px;
$ganttHeight: 1440px;
.gantt {
  margin: 50px auto 100px auto;
  width: $ganttWidth;
  height: $ganttHeight;
  border: solid 4px rgba(0,0,0,.1);
  position: relative;
  > .task {
    position: absolute;
    z-index: 1;
    left: 0;
    top: 0;
    width: 100%;
    background-color: rgba(60,180,255,1);
    font-size: 15px;
    border-radius: 3px;
  }

  > #borderLine {
    position: absolute;
    z-index:2;
    width:100%;
    border-top: #ea4335 solid 2px;
    background-color: red;
  }

  @for $i from 1 through 23 {
    >.timesBorder#{$i}{
      position: absolute;
      z-index:0;
      width: 100%;
      border-top: rgba(0,0,0,0.1) solid 1px;
      background-color: rgba(0,0,0,0.1);
      top: $i * 60px;
    }
  }

  @for $i from 1 through 25 {
    >.times#{$i}{
      position: absolute;
      font-size: 16px;
      @if $i == 25 {
      top: $i * 60 - 77px;
      }
      @else{
      top: $i * 60 - 72px;
      }

      @if $i < 11{
      left: -40px;
      }
      @else {
      left: -50px;
      }
    }
  }
  
}
</style>
<style>
#overlay{
  /*　要素を重ねた時の順番　*/
  z-index:10;

  /*　画面全体を覆う設定　*/
  position:fixed;
  top:0;
  left:0;
  width:100%;
  height:100%;
  background-color:rgba(0,0,0,0.5);

  /*　画面の中央に要素を表示させる設定　*/
  display: flex;
  align-items: center;
  justify-content: center;

}
#content{
  z-index:12;
  width:30%;
  padding: 0.2em 2em 1em 2em;
  background:#fff;
  border-radius: 3px;
}
.taskName{
  font-size: 30px;
  padding-top: 1em;
}
.taskst{
  font-size: 20px;
}

.close{
  font-size: 30px;
  color: black;
  float: right;
  clear: both;
}
.taskSentence{
  font-size: 20px;
  padding-top: 40px;
  padding-bottom: 30px;
}
.buttonSet{
  text-align: right;
}
.delete{
  font-size: 17px;
  color: white;
  width: 60px;
  height: 30px;
  background-color: rgb(255,70,70);
  border-radius: 3px;
}
.delete:hover{
  background-color: rgba(255,70,70,0.9);
}
.edit{
  font-size: 17px;
  color: white;
  width: 60px;
  height: 30px;
  background-color: #1a73e8;
  border-radius: 3px;
  margin-right: 7px;
}
.edit:hover{
  background-color: rgba(26,115,232,0.9);
}

#taskRegister{
  /*　要素を重ねた時の順番　*/
  z-index:10;

  /*　画面全体を覆う設定　*/
  position:fixed;
  top:0;
  left:0;
  width:100%;
  height:100%;
}
#trContent{
  z-index:12;
  width:20%;
  height: 30%;
  padding: 0.2em 2em 1em 2em;
  background:#fff;
  border-radius: 3px;
  position: absolute;  
  left: 10%;
  background-color: rgb(240,240,240);
  border: solid 3px rgb(0,0,0);
}

</style>