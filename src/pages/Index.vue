<template>
  <q-page class="flex flex-center back">
    <vue-topprogress ref="topProgress" color="#F18900" :height="4" :speed="350" ></vue-topprogress>
    <div class="container container-fluid">
      <transition name="bounce">
        <div id="detail" v-if="detail">
            <div class="form-group area">
            <label for="exampleFormControlTextarea1">Enter lyrics:</label>
            <button type="button" class="btn btn-secondary" @click="lyrics=''">Clear</button>
            <button type="button" class="btn btn-success" @click="pasteAction()">Paste</button>
            <textarea class="form-control" id="exampleFormControlTextarea1" rows="3" v-model="lyrics"></textarea>
        </div>
        <div class="form-group row area block-space">
          <div class="col">
              <label for="upload">Upload Music</label>
              <input type="file" class="form-control-file" id="upload" @change="handleFiles">
          </div>
          <div class="col omitNumb">
            <label for="Number">Enter Number of Omitting Words: </label>
            <input type="number" class="form-control" id="Number" v-model="omitNumber" :placeholder="recommendation">
          </div>
        </div>
        <button type="button" @click="startAction()"  class="btn btn-primary btn-lg btn-block">Start!</button>
      </div>
      </transition>
      <button v-if="!detail" type="button" class="btn btn-info" style="margin-top: 1rem;" @click="backAction()">Back</button>
      <div style="margin-top: 1rem; margin-bottom: 1rem;">
        <audio id="audio" controls style="width:100%;">
          <source src="" id="src" />
        </audio>
      </div>
        <transition name="bounce">
          <div v-if="!detail" style="margin-top: 1rem; margin-bottom: 1rem;">
          <div class="pre">
            <xmp>
              <span v-html="formatted"></span>
            </xmp>
          </div>
          <button type="button" @click="endAction()" class="btn btn-primary">Finish!</button>
          <button type="button" @click="showAnswers()" class="btn btn-warning" style="margin-left: 1rem;">Show Answers</button>
          <button v-if="!detail" type="button" class="btn btn-info" style="margin-left: 1rem;" @click="backAction()">Back</button>
        </div>
        </transition>
    </div>
  </q-page>
</template>

<script>
import $ from 'jquery'
import { vueTopprogress } from 'vue-top-progress'

export default {
  name: 'PageIndex',
  mounted () {
    this.$refs.topProgress.start()

    // Use setTimeout for demo
    setTimeout(() => {
      this.$refs.topProgress.done()
    }, 2000)
  },
  components: {
    vueTopprogress
  },
  data () {
    return {
      detail: true,
      lyrics: '',
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
    pasteAction () {
      navigator.clipboard.readText().then(text => {
        this.lyrics += text
      })
    },
    backAction () {
      this.detail = !this.detail
      if (this.mp3Url) {
        document.getElementById('audio').pause()
      }
    },
    endAction () {
      for (var i = 0; i < this.result.length; i++) {
        console.log(this.result[i], ' ', i)
        if (this.result[i].toLowerCase() === $(`#${i}`).val().toLowerCase()) {
          $(`#${i}`).css('border-color', 'green')
          $(`#${i}`).css('border-width', '0.2rem')
        } else {
          $(`#${i}`).css('border-color', 'red')
          $(`#${i}`).css('border-width', '0.2rem')
        }
      }
    },
    showAnswers () {
      for (var i = 0; i < this.result.length; i++) {
        $(`#${i}`).val(this.result[i])
        $(`#${i}`).css('border-color', 'green')
        $(`#${i}`).css('border-width', '0.2rem')
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
            lyrics = lyrics.replace(word, `<input type="text" class="rounded" style="width: ${word.length * 10}px; border-color: black; margin:0.1rem;" id="${index}" oninput="$('#${index}').val().length === ${word.length} ? $('#${index}').next().focus() : console.log('')">`)
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
  xmp {
    white-space: pre-wrap;
    word-break: keep-all
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
    margin-left: 0.5rem;
  }
  .btn-success {
    float: right;
  }
  .back  {
    background: url('../assets/back.jpg');
    background-repeat: no-repeat;
    background-size: cover;
  }
  .container {
    background: rgb(186,209,219);
    background: linear-gradient(10deg, rgba(186,209,219,0.9) 25%, rgba(184,226,245,0.9) 58%, rgba(184,226,245,0.9) 100%);
    border-radius: 10px;
  }
  #detail {
    margin-top: 1rem;
  }
  textarea::-webkit-scrollbar {
    width: 12px;
    background-color: #F5F5F5; }

  textarea::-webkit-scrollbar-thumb {
    border-radius: 10px;
    -webkit-box-shadow: inset 0 0 6px rgba(0, 0, 0, 0.1);
    background-color: #4285F4; }
  @media only screen and (max-width: 535px) {
    #detail {
      font-size: 0.8rem;
    }
    .block-space {
      display: block;
    }
    .omitNumb {
      margin-top: 1rem;
    }
    #Number {
      width: 50%;
    }
    .pre {
      font-size: 0.8rem;
    }
  }
</style>
