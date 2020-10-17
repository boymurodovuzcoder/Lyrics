<template>
  <q-page class="flex flex-center">
    <div class="container container-fluid">
      <transition name="bounce">
        <div id="detail" v-if="detail">
            <div class="form-group area">
            <label for="exampleFormControlTextarea1">Enter lyrics:</label>
            <button type="button" class="btn btn-secondary" @click="lyrics=''">Clear</button>
            <textarea class="form-control" id="exampleFormControlTextarea1" rows="3" v-model="lyrics"></textarea>
        </div>
        <div class="form-group row area">
          <div class="col">
              <label for="upload">Upload Music</label>
              <input type="file" class="form-control-file" id="upload" @change="handleFiles">
          </div>
          <div class="col">
            <label for="Number">Enter Number of Omitting Words: </label>
            <input type="number" class="form-control" id="Number" v-model="omitNumber" :placeholder="recommendation">
          </div>
        </div>
        <button type="button" @click="startAction()"  class="btn btn-primary btn-lg btn-block">Start!</button>
      </div>
      </transition>
      <button v-if="!detail" type="button" class="btn btn-info" style="margin-top: 1rem;" @click="backAction()">Back!</button>
      <div style="margin-top: 1rem; margin-bottom: 1rem;">
        <audio id="audio" controls style="width:100%;">
          <source src="" id="src" />
        </audio>
      </div>
        <transition name="bounce">
          <div v-if="!detail" style="margin-top: 1rem; margin-bottom: 1rem;">
          <div>
            <xmp>
              <span v-html="formatted"></span>
            </xmp>
          </div>
          <button type="button" @click="endAction()" class="btn btn-primary">Finish!</button>
          <button type="button" @click="showAnswers()" class="btn btn-warning" style="margin-left: 1rem;">Show Answers</button>
        </div>
        </transition>
    </div>
  </q-page>
</template>

<script>
import $ from 'jquery'

export default {
  name: 'PageIndex',
  data () {
    return {
      detail: true,
      lyrics: null,
      omitNumber: null,
      formatted: null,
      result: null,
      mp3Url: null
    }
  },
  computed: {
    recommendation: function () {
      var copyArray = []
      if (this.lyrics) {
        var r = /(\w|\s)*\w(?=")|\w+('\w+)?/gm
        var array = this.lyrics.match(r)
        array.forEach(word => {
          if ((word.length > 3) && (word.indexOf("'") === -1)) {
            copyArray.push(word)
          }
        })
      }
      var result = ''
      copyArray.length > 1 ? result = copyArray.length + ' words are available' : result = copyArray.length + ' word is available'
      return result
    }
  },
  methods: {
    handleFiles (event) {
      var files = event.target.files
      this.mp3Url = URL.createObjectURL(files[0])
    },
    backAction () {
      this.detail = !this.detail
      if (this.mp3Url) {
        document.getElementById('audio').pause()
      }
    },
    endAction () {
      for (var i = 0; i < this.result.length; i++) {
        if (this.result[i].toLocaleLowerCase() === document.getElementById(`${i}`).value.toLocaleLowerCase()) {
          document.getElementById(`${i}`).style.borderColor = 'green'
        } else {
          document.getElementById(`${i}`).style.borderColor = 'red'
        }
      }
    },
    showAnswers () {
      for (var i = 0; i < this.result.length; i++) {
        document.getElementById(`${i}`).value = this.result[i]
        document.getElementById(`${i}`).style.borderColor = 'green'
      }
    },
    startAction () {
      if (!this.lyrics) {
        alert('Textarea is empty!')
      } else if (!this.omitNumber) {
        alert('Number of omitting words is empty!')
      } else {
        var r = /(\w|\s)*\w(?=")|\w+('\w+)?/gm
        var array = this.lyrics.match(r)
        var copyArray = []
        array.forEach(word => {
          if ((word.length > 3) && (word.indexOf("'") === -1)) {
            copyArray.push(word)
          }
        })
        if (this.omitNumber > copyArray.length) {
          alert(`${this.omitNumber} words can not be omitted!`)
        } else {
          this.detail = !this.detail
          if (this.mp3Url) {
            $('#src').attr('src', this.mp3Url)
            document.getElementById('audio').load()
            document.getElementById('audio').play()
          }
          var result = this.getRandom(copyArray, this.omitNumber)
          var lyrics = this.lyrics
          var index = 0
          result.forEach(word => {
            lyrics = lyrics.replace(word, `<input type="text" class="rounded" style="width: ${word.length * 10}px; border-color: black; margin:0.1rem;" id="${index}">`)
            index += 1
          })
          this.formatted = lyrics
          console.log(result)
          this.result = result
          console.log(lyrics)
        }
      }
    },
    getRandom (arr, n) {
      var result = new Array(n),
        len = arr.length,
        taken = new Array(len)
      if (n > len) {
        throw new RangeError('getRandom: more elements taken than available')
      }
      while (n--) {
        var x = Math.floor(Math.random() * len)
        result[n] = arr[x in taken ? taken[x] : x]
        taken[x] = --len in taken ? taken[len] : len
      }
      return result
    }
  }
}
</script>

<style scoped>
  .area {
    font-size: 1rem;
    font-weight: 500;
  }
  .bounce-enter-active {
    animation: bounce-in .5s;
  }
  .bounce-leave-active {
    animation: bounce-in .5s reverse;
  }
  @keyframes bounce-in {
    0% {
      transform: scale(0);
    }
    50% {
      transform: scale(1.5);
    }
    100% {
      transform: scale(1);
    }
  }
  .btn-secondary {
    float: right;
  }
</style>
